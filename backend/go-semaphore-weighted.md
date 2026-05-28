---
title: golang.org/x/sync/semaphore 加权信号量
date: 2026-05-26 22:00
tags: [Go, semaphore, 并发控制, 限流]
category: backend
---

# golang.org/x/sync/semaphore 加权信号量

## 核心概念

`semaphore.Weighted` 是 Go 官方提供的加权信号量实现，本质上是一个**资源配额管理器**。它跟踪可用资源的数量，goroutine 必须 Acquire（获取）到足够资源才能执行，执行完毕后 Release（释放）归还资源。

底层实现基于 **mutex + condition variable + 链表等待队列**，比 channel 方案的性能更好。

## 使用场景

- 限制数据库连接池并发查询数
- 控制对外部 API 的最大并发调用数
- 限制文件 IO / 网络 IO 的并发量
- 实现带权重的任务调度（大任务占用更多资源）
- 优雅降级（TryAcquire 非阻塞尝试）

## 核心 API

```go
import "golang.org/x/sync/semaphore"

// 创建信号量，maxResources 是最大并发数
sem := semaphore.NewWeighted(maxResources)

// 阻塞式获取，支持 context 取消/超时
sem.Acquire(ctx, n)

// 非阻塞尝试，成功返回 true
sem.TryAcquire(n)

// 释放资源
sem.Release(n)
```

## 代码示例

### 基本使用：控制并发爬虫

```go
sem := semaphore.NewWeighted(3) // 最多 3 个并发

for _, url := range urls {
    sem.Acquire(ctx, 1) // 阻塞等待空位
    go func(u string) {
        defer sem.Release(1)
        fetch(u)
    }(url)
}
```

### 带超时的并发控制

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

if err := sem.Acquire(ctx, 1); err != nil {
    // 3 秒内拿不到 -> 降级
    return http.StatusServiceUnavailable
}
defer sem.Release(1)
```

### 非阻塞尝试 + 降级

```go
if !sem.TryAcquire(1) {
    http.Error(w, "服务器忙", http.StatusServiceUnavailable)
    return
}
defer sem.Release(1)
```

## 与同类方案对比

| 特性 | semaphore.Weighted | chan struct{} | sync.WaitGroup |
|------|:---:|:---:|:---:|
| 限制并发数 | ✅ | ✅ | ❌ |
| 加权获取 | ✅ | ❌ | ❌ |
| 上下文取消 | ✅ | ❌ | ❌ |
| 非阻塞尝试 | ✅ | select+default | ❌ |
| 动态调容量 | ✅ | ❌ | ❌ |

## 源码级原理剖析

### 数据结构

```go
type Weighted struct {
    size    int64       // 信号量总容量
    cur     int64       // 当前已分配资源数
    mu      sync.Mutex  // 保护所有字段
    waiters list.List   // 等待队列（双向链表，FIFO）
}

type waiter struct {
    n     int64          // 该 waiter 需要的资源数
    ready chan<- struct{} // close 此 channel 来唤醒 goroutine
}
```

### 调度算法核心：notifyWaiters

```go
func (s *Weighted) notifyWaiters() {
    for {
        next := s.waiters.Front()
        if next == nil { break }
        w := next.Value.(waiter)
        if s.size-s.cur < w.n { break } // [关键] 队首资源不够 -> 停止
        s.cur += w.n
        s.waiters.Remove(next)
        close(w.ready) // 通过 close channel 发信号
    }
}
```

**设计决策：严格 FIFO + 队首阻塞即停**。不允许跳过队首的小请求先执行，虽然会降低吞吐量，但**防止大请求无限饥饿**（例如 Writer 需要 N 个资源，Reader 们不断插队导致 Writer 永无机会）。

### Acquire 三路径

| 路径 | 条件 | 行为 |
|------|------|------|
| 快速失败 | ctx 已取消 | 直接返回 ctx.Err() |
| 快速成功 | 资源够 + 无人排队 | cur += n，零等待返回 |
| 阻塞等待 | 资源不够 / 有人排队 | 入队 → select(ctx, ready) 二路等待 |

### Release & 取消处理

- Release 释放资源后立即调用 notifyWaiters 唤醒队首
- ctx 取消时分为两种情况：已拿到资源则归还 + 通知后面的人；未拿到则出队 + 如果自己是队首则尝试通知

### 关键设计决策总结

| 决策 | 选择 | 原因 |
|------|------|------|
| 数据结构 | mutex + 双向链表 | O(1) 插入删除 + 严格 FIFO |
| 唤醒机制 | close(channel) | 一次性事件通知，比发值更安全 |
| 排队策略 | 严格 FIFO | 防止大请求饥饿 |
| 快速路径 | 无人排队才走 | 保证 fairness |
| 取消语义 | 完整支持 context | 超时 + 优雅取消 |

## 注意事项

- Release 数量不能超过 Acquire 总数，否则超出初始容量
- 必须配对 Acquire/Release，建议用 `defer` 保证释放
- 底层有锁，但锁持有时间极短
- 常与 errgroup 组合使用，形成黄金并发控制组合

---

*最后更新：2026-05-26*
