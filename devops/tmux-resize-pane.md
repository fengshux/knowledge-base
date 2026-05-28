---
title: tmux 调整窗口和 Pane 大小
date: 2026-04-28 15:45
tags: [tmux, 终端复用，pane 管理，窗口调整，Linux]
category: devops
---

# tmux 调整窗口和 Pane 大小

## 核心概念

tmux 中的窗口（window）可以分割成多个窗格（pane），每个 pane 都是独立的终端会话。调整 pane 大小是日常使用 tmux 的常见需求，可以通过快捷键或命令实现。

## 使用场景

- 需要给某个 pane 更多空间查看输出
- 调整 pane 布局以适应不同的工作内容
- 优化终端空间利用率
- 快速切换 pane 大小布局

## 快捷键调整方法

### 1. 标准快捷键（先按前缀键）

**前缀键**：默认是 `Ctrl + b`，按下后松开，再按以下组合：

| 操作 | 快捷键 | 说明 |
|------|--------|------|
| 向上调整 | `Ctrl + ↑` | 增大上方 pane 的高度 |
| 向下调整 | `Ctrl + ↓` | 增大下方 pane 的高度 |
| 向左调整 | `Ctrl + ←` | 增大左侧 pane 的宽度 |
| 向右调整 | `Ctrl + →` | 增大右侧 pane 的宽度 |

**完整操作**：
```bash
# 例如向下调整：
1. 按 Ctrl + b（前缀键）
2. 松开
3. 按 Ctrl + ↓
```

### 2. Mac 用户快捷键

Mac 上使用 `Option`（或 `Alt`）键代替 `Ctrl`：

| 操作 | 快捷键 |
|------|--------|
| 向上调整 | `Ctrl + b` → `Option + ↑` |
| 向下调整 | `Ctrl + b` → `Option + ↓` |
| 向左调整 | `Ctrl + b` → `Option + ←` |
| 向右调整 | `Ctrl + b` → `Option + →` |

### 3. 命令模式调整

```bash
# 进入命令模式
Ctrl + b，然后按 :

# 输入调整命令（单位是字符数）
resize-pane -U 10    # 向上调整 10 行
resize-pane -D 10    # 向下调整 10 行
resize-pane -L 10    # 向左调整 10 列
resize-pane -R 10    # 向右调整 10 列

# 调整特定 pane（需要 pane 的 ID）
resize-pane -t pane-id -U 5
```

**查看 pane ID：**
```bash
# 在命令模式下输入
list-panes
# 或按 Ctrl+b 然后按 q 显示 pane 编号
```

## 自定义快捷键配置

### 推荐配置（无需前缀键）

在 `~/.tmux.conf` 中添加：

```bash
# 使用 Ctrl + 方向键直接调整（无需先按 Ctrl+b）
bind-key -n C-Up resize-pane -U 5
bind-key -n C-Down resize-pane -D 5
bind-key -n C-Left resize-pane -L 5
bind-key -n C-Right resize-pane -R 5

# 或者使用 Alt/Option + 方向键
bind-key -n M-Up resize-pane -U 5
bind-key -n M-Down resize-pane -D 5
bind-key -n M-Left resize-pane -L 5
bind-key -n M-Right resize-pane -R 5
```

**配置说明：**
- `-n` 表示无需前缀键，直接按组合键生效
- `C-` 表示 Ctrl 键
- `M-` 表示 Alt/Meta/Option 键
- 数字 `5` 表示每次调整 5 个字符单位

### 启用鼠标支持

在 `~/.tmux.conf` 中添加：
```bash
# 启用鼠标，可以直接拖拽 pane 边框调整大小
set -g mouse on
```

### 完整配置示例

```bash
# ~/.tmux.conf

# 设置前缀键为 Ctrl+a（可选）
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 启用鼠标支持
set -g mouse on

# 自定义 pane 调整快捷键（无需前缀键）
bind-key -n C-Up resize-pane -U 5
bind-key -n C-Down resize-pane -D 5
bind-key -n C-Left resize-pane -L 5
bind-key -n C-Right resize-pane -R 5

# 调整整个 window 的大小
bind-key -n C-M-Up resize-window -y +5
bind-key -n C-M-Down resize-window -y -5
bind-key -n C-M-Left resize-window -x -5
bind-key -n C-M-Right resize-window -x +5

# 重新加载配置的快捷键
bind r source-file ~/.tmux.conf \; display "配置已重载！"
```

## 调整整个 Window 大小

```bash
# 进入命令模式
Ctrl + b，然后按 :

# 设置 window 的宽度和高度（字符数）
resize-window -x 120    # 设置宽度为 120 列
resize-window -y 40     # 设置高度为 40 行

# 相对调整
resize-window -x +10    # 宽度增加 10 列
resize-window -y -5     # 高度减少 5 行
```

## 均衡 Pane 布局

```bash
# 进入命令模式或使用快捷键
Ctrl + b，然后按 :

# 水平均分所有 pane
select-layout even-horizontal

# 垂直均分所有 pane
select-layout even-vertical

# 平铺布局（自动排列）
select-layout tiled

# 恢复上一个布局
Ctrl + b，然后按 l（小写 L）
```

## 实用场景示例

### 场景 1：快速扩大当前 pane

```bash
# 配置好自定义快捷键后，直接按 Ctrl+↓ 扩大当前 pane
# 无需先按 Ctrl+b
```

### 场景 2：精确调整到指定大小

```bash
# 进入命令模式，精确调整 20 行
Ctrl+b → : → resize-pane -D 20
```

### 场景 3：使用鼠标拖拽

```bash
# 启用鼠标后（set -g mouse on）
# 直接用鼠标拖拽 pane 的边框调整大小
```

### 场景 4：查看 pane 编号后调整特定 pane

```bash
# 1. 按 Ctrl+b 然后按 q 显示 pane 编号
# 2. 记住要调整的 pane 编号（如 1）
# 3. 进入命令模式：Ctrl+b → :
# 4. 输入：resize-pane -t 1 -R 10
```

### 场景 5：快速均衡布局

```bash
# 当 pane 大小混乱时，快速恢复均衡
Ctrl+b → : → select-layout even-horizontal
```

## 注意事项

- ⚠️ **终端兼容性**：某些终端模拟器可能不支持方向键组合，需要配置 tmux 或使用命令模式
- ⚠️ **Mac 终端设置**：Mac 用户可能需要在终端偏好设置中启用"使用 Option 键作为 Meta 键"
- ⚠️ **调整单位**：调整的单位是字符数（行/列），不是像素
- ⚠️ **最小尺寸**：pane 有最小尺寸限制，不能无限缩小
- ✅ **推荐配置**：在 `~/.tmux.conf` 中配置自定义快捷键，使用更方便
- ✅ **鼠标支持**：启用鼠标后可以直接拖拽调整，更直观
- ✅ **配置生效**：修改配置后需要执行 `source-file ~/.tmux.conf` 重载

## 故障排查

### 问题 1：方向键不生效

**解决方案：**
```bash
# 1. 尝试使用命令模式
Ctrl+b → : → resize-pane -D 5

# 2. 检查终端是否支持方向键
# 在终端输入：cat -v 然后按方向键，看是否有输出

# 3. 在 ~/.tmux.conf 中添加终端兼容性配置
set -g terminal-overrides 'xterm*:kUP5=\e[1;5A:kDOWN5=\e[1;5B:kLEFT5=\e[1;5D:kRIGHT5=\e[1;5C'
```

### 问题 2：Mac 上 Option 键不生效

**解决方案：**
```bash
# 1. 在终端偏好设置中：
# 终端 → 偏好设置 → 键盘 → 勾选"使用 Option 键作为 Meta 键"

# 2. 或者使用 Ctrl 键代替
bind-key -n C-Up resize-pane -U 5
```

### 问题 3：配置不生效

**解决方案：**
```bash
# 1. 在 tmux 内重载配置
Ctrl+b → : → source-file ~/.tmux.conf

# 2. 检查配置文件语法
tmux show-options -g

# 3. 重启 tmux server
tmux kill-server
tmux
```

## 最佳实践

1. **配置自定义快捷键**：在 `~/.tmux.conf` 中配置无需前缀键的组合键
2. **启用鼠标支持**：方便快速拖拽调整
3. **保存常用布局**：使用 `choose-tree` 或插件保存常用布局
4. **合理分割 pane**：根据工作内容合理规划 pane 布局
5. **定期清理**：关闭不用的 pane，保持工作区整洁

## 相关插件推荐

### tmux-resurrect
保存和恢复 tmux 会话（包括 pane 布局）：
```bash
# 在 ~/.tmux.conf 中
set -g @plugin 'tmux-plugins/tmux-resurrect'
```

### tmux-pane-sequences-prefix
使用数字快捷键快速调整 pane：
```bash
# 在 ~/.tmux.conf 中
set -g @plugin 'tmux-plugins/tmux-pane-sequences-prefix'
```

## 参考资料

- [tmux 官方文档](https://github.com/tmux/tmux/wiki)
- [tmux 调整 pane 大小](https://unix.stackexchange.com/questions/24688/how-to-resize-panes-in-tmux)
- [tmux 配置指南](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
- [tmux 快捷键速查](https://tmuxcheatsheet.com/)

---

*最后更新：2026-04-28*
