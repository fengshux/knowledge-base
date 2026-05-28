---
title: tmux 常用快捷键
date: 2026-04-02
tags: [tmux, 终端，分屏，快捷键，命令行，开发工具]
category: devops
---

# tmux 常用快捷键

> 终端复用器 tmux (Terminal Multiplexer) 的常用操作快捷键汇总

## 目录
- [分屏操作](#分屏操作)
- [窗口操作](#窗口操作)
- [Pane 调整](#pane-调整)
- [快速技巧](#快速技巧)

---

## 分屏操作

### 垂直分屏
将屏幕左右分成两个 pane

```
Ctrl + b  然后按  Shift + %
```

### 水平分屏
将屏幕上下分成两个 pane

```
Ctrl + b  然后按  "
```

### 切换 Pane
在各个 pane 之间切换

```
Ctrl + b  然后按 方向键
```

### 关闭当前 Pane
关闭当前所在的 pane

```
Ctrl + b  然后按  x
```

### 最大化/还原当前 Pane
将当前 pane 最大化全屏显示，再次按可恢复

```
Ctrl + b  然后按  z
```

---

## 窗口操作

| 操作 | 快捷键 | 说明 |
|------|--------|------|
| 新建窗口 | `Ctrl + b` + `c` | Create new window |
| 列出窗口 | `Ctrl + b` + `w` | Window list |
| 切换窗口 | `Ctrl + b` + 数字 | 快速跳转到指定编号的窗口 |
| 重命名窗口 | `Ctrl + b` + `,` | Rename current window |

---

## Pane 调整

### 调整 Pane 大小
```
Ctrl + b  然后按  :
```
然后输入：
```
resize-pane -D  # 向下调整
resize-pane -U  # 向上调整
resize-pane -L  # 向左调整
resize-pane -R  # 向右调整
```

### 显示 Pane 编号并快速跳转
```
Ctrl + b  +  q
```
显示编号后，按数字键可直接跳转到对应 pane

---

## 快速技巧

### 快捷键使用方式
1. 首先按下 `Ctrl + b`（这是 tmux 的默认前缀键）
2. 放开后，再按对应的功能键

### 常用组合快捷键

| 操作             | 快捷键                           |
| -------------- | ----------------------------- |
| 列出所有快捷键        | `Ctrl + b` + `?`              |
| 水平分屏（替代方案）     | `Ctrl + b` + `Alt + 2`（需终端支持） |
| 切换到最近使用的 pane  | `Ctrl + b` + `;`              |
| 交换当前 pane 与上一个 | `Ctrl + b` + `{`              |
| 交换当前 pane 与下一个 | `Ctrl + b` + `}`              |

---

## 自定义配置

如果需要调整前缀键或进行更详细的自定义，可以编辑 `~/.tmux.conf` 配置文件。

### 常见配置示例
```bash
# 将前缀键从 Ctrl+b 改为 Ctrl+a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 启用鼠标支持
set -g mouse on

# 启用窗口编号从 1 开始
set -g base-index 1

# 启用面板编号从 1 开始
set -g pane-base-index 1
```

---

## 标签
- #tmux #终端 #分屏 #快捷键 #命令行 #开发工具

---

*最后更新：2026-04-02*