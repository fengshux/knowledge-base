---
title: Go 异常捕获（panic/recover）
date: 2026-05-14 15:36
tags: [Go, Golang, panic, recover, error handling]
category: backend
---

# Go 异常捕获（panic/recover）

## 核心概念

Go 没有 try-catch 异常机制。它采用两条不同路线处理错误：

1. **普通错误（error）**：可预期的错误（文件不存在、网络超时等），通过 `error` 接口 + `if err != nil` 处理
2. **异常（panic/recover）**：程序 bug、数组越界、空指针等不可恢复情况，通过 `panic` 触发，`recover` 捕获

> `panic` 不应替代错误处理。`recover` 的存在是为了让程序在意外 panic 时有"最后的体面"。

## 使用场景

- main() 最外层兜底：防止程序未经任何日志就崩溃
- goroutine 中的 panic 捕获：每个 goroutine 必须单独 recover
- HTTP 服务中间件：单个请求 panic 不搞崩整个进程
- 第三方库内部 panic：用 recover 将 panic 转为 error 返回

## 基本语法

```go
defer func() {
    if r := recover(); r != nil {
        // 捕获到了 panic，r 是 panic 传入的值
        log.Printf("recovered from panic: %v", r)
    }
}()
```

**注意**：
- `recover` 只有在 `defer` 函数中调用才有效
- `defer` 必须定义在可能发生 `panic` 的代码**之前**
- 一个 goroutine 的 recover 不能捕获另一个 goroutine 的 panic

## 代码示例

### main() 最外层捕获

```go
package main

import (
    "fmt"
    "log"
)

func main() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("[FATAL] %v", r)
        }
    }()
    panic("致命错误")
}
```

### 安全的 goroutine 启动器

```go
func goSafe(fn func()) {
    go func() {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("[CRASH] goroutine panic: %v\n%s", r, debug.Stack())
            }
        }()
        fn()
    }()
}
```

### HTTP 恢复中间件

```go
func recoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if rec := recover(); rec != nil {
                log.Printf("[PANIC] %v\n%s", rec, debug.Stack())
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()
        next.ServeHTTP(w, r)
    })
}
```

## 注意事项

- ❌ `main()` 的 defer recover 捕不到子 goroutine 的 panic
- ❌ 不要用 panic/recover 代替 if err != nil
- ❌ 不要在 recover 后什么都不做（至少打日志）
- ✅ recover 配合 `debug.Stack()` 打印完整调用栈
- ✅ 对于 goroutine，建议封装 goSafe/GoSafely 等工具函数统一处理

## 最佳实践

1. **每个 goroutine 必须兜底**：用工具函数包装，一行调用
2. **HTTP 服务必须加 recover 中间件**：避免单请求拖垮整个服务
3. **recover 后至少打印调用栈**：方便定位问题
4. **panic 转 error**：在库/中间件里，把内部 panic 转为 error 返回给调用方
5. **真正致命的不 recover**：比如启动时配置错误，直接 panic 停掉反而更安全

## 参考资料

- [Go Blog: Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)
- [Go 官方文档: Built-in functions](https://go.dev/ref/spec#Handling_panics)

---

*最后更新：2026-05-14*