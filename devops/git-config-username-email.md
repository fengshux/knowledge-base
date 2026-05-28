---
title: Git 配置用户名和邮箱
date: 2026-05-08 10:30
tags: [Git, 配置，版本控制]
category: devops
---

# Git 配置用户名和邮箱

## 核心概念

Git 在每次提交（commit）时都需要记录作者信息，包括用户名（user.name）和邮箱（user.email）。Git 配置分为三个层级，优先级从低到高依次为：系统级、全局级、项目级。

## 配置层级

| 层级 | 参数 | 配置文件位置 | 生效范围 |
|------|------|-------------|---------|
| 系统级 | `--system` | `/etc/gitconfig` | 系统所有用户 |
| 全局级 | `--global` | `~/.gitconfig` | 当前用户的所有项目 |
| 项目级 | `--local` 或不写参数 | `.git/config` | 仅当前项目 |

## 项目级配置（针对单个项目）

当需要为特定项目使用不同的身份时（如公司项目用工作邮箱，个人项目用个人邮箱），使用项目级配置：

```bash
# 进入项目目录
cd /path/to/your/project

# 设置当前项目的用户名
git config user.name "Your Name"

# 设置当前项目的邮箱
git config user.email "your.email@example.com"
```

## 全局配置（默认使用）

设置一次后，所有项目都会使用这个配置：

```bash
# 设置全局用户名
git config --global user.name "Your Name"

# 设置全局邮箱
git config --global user.email "your.email@example.com"
```

## 验证配置

```bash
# 查看当前项目的用户名和邮箱
git config user.name
git config user.email

# 查看所有配置（包含所有层级）
git config --list

# 查看配置来源（显示来自哪个配置文件）
git config --show-origin --get user.name
git config --show-origin --get user.email

# 仅查看项目级配置
git config --local --list
```

## 代码示例

### 场景：为公司项目配置工作邮箱

```bash
# 进入公司项目目录
cd /Users/ext.xuxiaoyu12/Documents/Loomy\ Workspace/my-company-project

# 配置工作身份
git config user.name "Xu Xiaoyu"
git config user.email "xiaoyu.xu@company.com"

# 验证配置
git config user.name
# 输出：Xu Xiaoyu

git config user.email
# 输出：xiaoyu.xu@company.com

# 查看配置来源
git config --show-origin --get user.name
# 输出：file:.git/config    Xu Xiaoyu
```

### 场景：切换回个人项目使用个人邮箱

```bash
# 进入个人项目目录
cd ~/projects/personal-blog

# 配置个人身份（项目级，优先级高于全局）
git config user.name "Xiaoyu Xu"
git config user.email "xiaoyu.personal@gmail.com"
```

## 注意事项

- [ ] **项目级配置优先级最高**：如果同时设置了全局和项目级配置，Git 会优先使用项目级的配置
- [ ] **配置文件位置**：项目级配置保存在项目根目录的 `.git/config` 文件中
- [ ] **需要先进入项目目录**：必须在 Git 仓库根目录下执行项目级配置，否则会配置到全局
- [ ] **配置后立即生效**：之后的 commit 会使用新的用户名和邮箱
- [ ] **不影响已有提交**：修改配置不会影响之前的 commit 记录

## 删除配置

```bash
# 删除项目级配置（会回退到全局配置）
git config --unset user.name
git config --unset user.email

# 删除全局配置
git config --global --unset user.name
git config --global --unset user.email
```

## 常见问题排查

### 问题：commit 时用的不是预期的邮箱

**排查步骤：**

```bash
# 1. 查看当前生效的配置
git config user.email

# 2. 查看配置来源
git config --show-origin --get user.email

# 3. 查看所有层级的配置
git config --list --show-origin

# 4. 如果配置错误，重新设置
git config --global user.email "correct.email@example.com"
```

### 问题：如何修改已提交的历史记录中的邮箱

如果已经用错误的邮箱提交了，可以使用以下命令修改最近的 commit：

```bash
# 修改最近一次 commit 的作者信息
git commit --amend --author="New Name <new.email@example.com>"

# 批量修改历史提交（谨慎使用）
git filter-branch --env-filter '
export GIT_AUTHOR_NAME="New Name"
export GIT_AUTHOR_EMAIL="new.email@example.com"
export GIT_COMMITTER_NAME="New Name"
export GIT_COMMITTER_EMAIL="new.email@example.com"
' HEAD
```

## 最佳实践

1. **全局配置个人邮箱**：在全局配置中设置个人邮箱作为默认值
2. **项目级配置工作邮箱**：进入公司项目后，单独配置工作邮箱
3. **使用 Git Hooks 自动切换**：可以编写脚本在项目目录变化时自动切换配置
4. **定期检查配置**：重要项目提交前，用 `git config --list` 确认邮箱正确

## 实际应用场景

- **多身份开发**：同时参与多个公司项目或个人项目，需要不同的邮箱标识
- **开源贡献**：为公司贡献开源项目时使用公司邮箱，个人项目使用个人邮箱
- **隐私保护**：在公开仓库中使用别名或不暴露个人邮箱

## 参考资料

- [Git 官方文档 - Git 配置](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%B0%B1%E9%85%8D%E7%BD%AE)
- [Git Config 文档](https://git-scm.com/docs/git-config)

---

*最后更新：2026-05-08*
