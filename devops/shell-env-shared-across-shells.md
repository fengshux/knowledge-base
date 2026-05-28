---
title: Shell 环境变量跨 shell 共享（zsh vs bash）
date: 2026-05-26 15:25
tags: [shell, zsh, bash, 环境变量, macOS]
category: devops
---

# Shell 环境变量跨 shell 共享

## 核心概念

不同的 shell（zsh、bash 等）有各自独立的配置文件，在一个 shell 中设置的环境变量不会自动被另一个 shell 继承。例如 zsh 读取 `~/.zshrc`，bash 读取 `~/.bash_profile` / `~/.bashrc`，两者互不感知。

## 使用场景

- 在 zsh（终端默认 shell）中设置了环境变量，但使用 bash 运行的进程（如某些 CLI 工具、CI 脚本、Loomy 等）读取不到
- 需要在不同 shell 之间保持一致的环境变量配置

## 解决方案

### 方案一：抽取公共环境变量文件（推荐）

创建独立的环境变量文件，让 zsh 和 bash 都 source 它：

```bash
# 1. 创建公共文件
cat > ~/.env_shared << 'EOF'
# ===== 公共环境变量 =====
export MY_API_KEY="abc123"
export WORK_DIR="/path/to/project"
export PATH="$PATH:/usr/local/custom/bin"
# 继续添加其他变量...
EOF

# 2. ~/.zshrc 中添加
echo '[ -f "$HOME/.env_shared" ] && source "$HOME/.env_shared"' >> ~/.zshrc

# 3. ~/.bash_profile 中添加
echo '[ -f "$HOME/.env_shared" ] && source "$HOME/.env_shared"' >> ~/.bash_profile
```

### 方案二：分别维护两份配置

把环境变量分别写入 `~/.zshrc` 和 `~/.bash_profile`。缺点：需要维护两份。

### 方案三：macOS launchctl 系统级

```bash
# 影响所有进程（含 GUI 应用）
launchctl setenv MY_API_KEY abc123
# 重启后失效，可放在 ~/.zshrc 中自动执行
```

## 验证方法

```bash
bash -l -c 'echo $YOUR_VARIABLE'
```

## 各 shell 配置文件读取顺序

| Shell | Login shell | Non-login interactive |
|-------|-------------|----------------------|
| bash  | `~/.bash_profile` → `~/.bashrc` | `~/.bashrc` |
| zsh   | `~/.zprofile` → `~/.zshrc` | `~/.zshrc` |

## 注意事项
- `source` 或 `.` 都可以用来加载外部文件
- `~/.bash_profile` 只在 login shell 启动时读取，Loomy 默认以 non-login 方式启动 bash 的话，可能需要确认加载机制
- macOS 从 Catalina 开始默认 shell 是 zsh
- 环境变量改动后，需要重新加载配置文件或新启动进程才能生效

---

*最后更新：2026-05-26*