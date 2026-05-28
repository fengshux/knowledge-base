---
title: tmux Session 创建 - 新建和管理会话
date: 2026-05-22 17:50
tags: [tmux, 终端复用，session 管理, session 创建, Linux, 开发工具]
category: devops
---

# tmux Session 创建 - 新建和管理会话

> tmux session（会话）是 tmux 的顶层容器，一个 session 可以包含多个 window（窗口），每个 window 又可以分割成多个 pane（窗格）。掌握 session 的创建和管理是使用 tmux 的基础。

## 核心概念

tmux 的层级结构：**Session（会话）→ Window（窗口）→ Pane（窗格）**

- **Session**：独立的工作区，可以包含多个窗口。每个 session 有独立的名称和编号
- **Window**：类似浏览器标签页，默认一个 session 只有一个 window，可以创建多个
- **Pane**：window 内的分割区域，可以水平/垂直分割

## 创建新 Session

### 1. 从命令行创建（常用）

```bash
# 创建并命名新 session
tmux new -s my-session

# 或使用长选项
tmux new-session -s my-session

# 不指定名称，自动分配数字名称
tmux new

# 创建 session 并在特定目录启动
tmux new -s project -c ~/project

# 创建 session 并执行一条命令（命令执行完后退出）
tmux new -s build-session bash build.sh
```

### 2. 在 tmux 内部创建新 session

**方法一：命令行模式**
```
# 按前缀键进入命令模式
Ctrl + b  →  :

# 在底部提示符输入
new-session -s my-session
```

**方法二：通过 session 选择界面**
```
Ctrl + b  →  s
```
进入 session 选择界面后，按 `c`（create）可创建新 session，按方向键选择已有 session 后回车切换。

### 3. 创建并附加到新 session（从外部）

```bash
# 创建新 session 并立即附加（attach）
tmux new -s my-session

# 如果已有同名的 session，先杀掉再新建
tmux new -s my-session || tmux kill-session -t my-session && tmux new -s my-session

# 检测 session 是否存在，存在则 attach，不存在则新建
tmux new -A -s my-session
# -A 表示：如果 session 已存在，直接 attach 上去
```

## 常用快捷键速查

### Session 管理快捷键

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `Ctrl + b` → `d` | **分离（detach）** | 从当前 session 断开，session 在后台继续运行 |
| `Ctrl + b` → `s` | **选择/切换 session** | 弹出交互式列表，可用方向键选择并回车切换 |
| `Ctrl + b` → `(` | **切换到上一个 session** | 循环切换 |
| `Ctrl + b` → `)` | **切换到下一个 session** | 循环切换 |
| `Ctrl + b` → `L` | **切换到最近使用的 session** | 快速跳回上一个 work 的 session |
| `Ctrl + b` → `$` | **重命名当前 session** | 底部会提示输入新名称 |
| `Ctrl + b` → `:` → `new-session` | **在内部新建 session** | 需要在命令模式输入 |

### Window 管理快捷键（session 内）

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + b` → `c` | 在当前 session 中新建 window |
| `Ctrl + b` → `数字键` | 跳转到指定编号的 window |
| `Ctrl + b` → `n` | 切换到下一个 window |
| `Ctrl + b` → `p` | 切换到上一个 window |
| `Ctrl + b` → `w` | 列出所有 window 供选择 |
| `Ctrl + b` → `,` | 重命名当前 window |
| `Ctrl + b` → `&` | 关闭当前 window（需确认） |

## 实用命令示例

### 场景 1：按项目创建独立 session

```bash
# 为不同项目创建独立 session
tmux new -s blog -c ~/projects/blog
tmux new -s api    -c ~/projects/api
tmux new -s app    -c ~/projects/app

# 分离后重新连接
tmux attach -t blog
```

### 场景 2：一键创建或重连（idempotent）

```bash
# 如果 session 存在就 attach，不存在则创建
tmux new -A -s my-session

# 可以结合 -c 指定目录
tmux new -A -s my-session -c ~/project

# 在 shell 别名中常用
alias tblog='tmux new -A -s blog -c ~/projects/blog'
```

### 场景 3：批量管理多个 session

```bash
# 列出所有 session
tmux ls

# 列出 session 及其详细信息
tmux list-sessions -v

# 切换到其他 session（从 tmux 内部）
tmux switch-client -t other-session

# 发送命令到指定 session 的窗口
tmux send-keys -t my-session:0 'git status' Enter
```

### 场景 4：启动时自动创建 session 布局

```bash
# 创建 session 并自动创建多个 window
tmux new -s dev -d          # 创建并 detach
tmux new-window -t dev -n editor  # 新建 editor 窗口
tmux new-window -t dev -n server  # 新建 server 窗口
tmux send-keys -t dev:editor 'vim' Enter  # 在 editor 窗口启动 vim
tmux send-keys -t dev:server 'npm run dev' Enter  # 在 server 窗口启动服务
tmux attach -t dev           # 附加到 dev session
```

tmux 3.2+ 还可以一行搞定：
```bash
# 使用 tmuxp（tmux 会话管理器）或 tmuxinator 管理复杂布局
```

### 场景 5：自定义快捷键一键新建 session

在 `~/.tmux.conf` 中配置：

```bash
# Ctrl + b 然后按 C（大写）直接新建 session
bind C new-session

# Ctrl + b 然后按 S（大写）列出 session 或在底部创建新 session
bind S command-prompt -p "新 session 名称:" "new-session -s '%%'"
```

## 注意事项

- ⚠️ session 名称**区分大小写**，`MySession` 和 `mysession` 是不同的
- ⚠️ session 名称中**不要包含冒号** `:`，它会被 tmux 解析为分隔符
- ⚠️ 从内部创建 session 时，新 session 会在后台创建，不会自动切换过去
- ⚠️ `Ctrl + b` → `s` 进入列表后按 `c` 可以**在当前会话下创建新 session**
- ✅ 创建 session 时**养成命名的习惯**，后续管理会更方便
- ✅ 使用 `tmux new -A -s name` 可以做到"不存在则创建，存在则连接"，是日常最安全的用法

## 最佳实践

### 1. 按任务命名 session

```bash
# ❌ 不命名或随便命名
tmux new

# ✅ 使用有意义的名称
tmux new -s blog-dev
tmux new -s api-server
tmux new -s db-admin
```

### 2. 配置启动脚本

将常用项目布局写成脚本或使用 tmuxp：

```yaml
# ~/.tmuxp/blog.yml（使用 tmuxp 插件）
session_name: blog
windows:
  - window_name: code
    panes:
      - vim
  - window_name: server
    panes:
      - npm run dev
  - window_name: git
    panes:
      - lazygit
```

启动：
```bash
tmuxp load blog
```

### 3. 使用 alias 快速启动

在 `~/.zshrc` 或 `~/.bashrc` 中添加：

```bash
alias t='tmux new -A -s $(basename $(pwd))'
alias tblog='tmux new -A -s blog -c ~/projects/blog'
alias tapi='tmux new -A -s api -c ~/projects/api'

# 自动在当前目录创建 session
tm() {
  local name="${1:-$(basename $PWD)}"
  tmux new -A -s "$name" -c "$PWD"
}
```

## 故障排查

### Q: 创建 session 时报 "sessions should be nested with care"

A: 在 tmux 内部再启动 tmux 会出现嵌套警告。可以用以下方式避免：
```bash
# 方式 1：在外部 tmux 中先 detach 当前 session
Ctrl+b → d
# 然后新建

# 方式 2：使用 -d 标志创建新 session 而不 attach
tmux new -s nested -d
tmux attach -t nested
```

### Q: 创建 session 后立刻自动退出了

A: 通常是启动命令执行完毕后 session 就退出了。可以用 `-d` 保持：
```bash
# 启动服务后保持 session 开启
tmux new -s server "npm run dev; exec bash"
# 或
tmux new -s server bash -c "npm run dev; exec bash"
```

### Q: 如何查看当前在哪个 session 中？

A: 在 tmux 中执行：
```bash
tmux display-message -p '#S'
# 输出：my-session
```

## 参考资料

- [tmux 官方 GitHub](https://github.com/tmux/tmux/wiki)
- [tmux 快速入门指南](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
- [tmuxp - Session Manager](https://github.com/tmux-python/tmuxp)
- [tmux 快捷键速查](https://tmuxcheatsheet.com/)

---

*最后更新：2026-05-22*