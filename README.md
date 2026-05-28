---
title: 技术知识库索引
date: 2026-05-27
tags: [索引, 知识库, README]
category: meta
---

# 📚 技术知识库索引

> 个人技术知识库，按技术类别分目录，每个知识点独立文件

**最后更新**：2026-05-27

---

## 📁 目录结构

```
knowledge-base/
├── README.md                        # 本索引文件
├── ai/                              # AI 与机器学习（2 个知识点）
│   ├── embodied-ai-overview.md      # 具身智能（Embodied AI）学习路线图
│   └── ros-introduction.md          # ROS（Robot Operating System）简介
├── database/                        # 数据库技术（6 个知识点）
│   ├── elasticsearch-architecture-principles.md  # Elasticsearch 架构原理与优化实战
│   ├── mysql-batch-bypass-gateway-timeout.md     # MySQL 分批执行绕过网关超时
│   ├── mysql-if-else.md             # MySQL IF ELSE 条件判断
│   ├── mysql-join-update.md         # MySQL 跨表关联更新
│   ├── mysql-table-backup.md        # MySQL 备份单表
│   └── mysql-union-syntax.md        # MySQL UNION 语法
├── frontend/                        # 前端技术（待添加）
├── backend/                         # 后端技术（2 个知识点）
│   ├── go-panic-recover.md          # Go 异常捕获（panic/recover）
│   └── go-semaphore-weighted.md     # golang.org/x/sync/semaphore 加权信号量
├── middleware/                      # 中间件技术（1 个知识点）
│   └── kafka-consumer-group.md      # Kafka 消费者组机制
├── devops/                          # 运维与 DevOps（15 个知识点）
│   ├── git-config-username-email.md # Git 配置用户名和邮箱
│   ├── linux-dns-config.md          # Linux DNS 服务器配置
│   ├── linux-network-traffic-monitoring.md  # Linux 网络流量查看方法
│   ├── oss-connection-reuse.md      # OSS 对象存储连接复用机制
│   ├── s3-copyobject-vs-uploadpartcopy.md   # S3 CopyObject vs UploadPartCopy
│   ├── shell-env-detection.md       # Shell 环境探测
│   ├── shell-env-shared-across-shells.md    # Shell 环境变量跨 shell 共享
│   ├── supervisor-process-management.md     # Supervisor 进程管理工具
│   ├── supervisor-scheduling.md     # Supervisor 定时任务调度
│   ├── tmux-resize-pane.md          # tmux 调整窗口和 Pane 大小
│   ├── tmux-session-create.md       # tmux Session 创建
│   ├── tmux-session-management.md   # tmux Session 管理
│   ├── tmux-shortcuts.md            # tmux 快捷键
│   ├── vim-delete.md                # Vim 删除操作
│   └── wc-l-line-count.md           # wc -l 统计行数少一行的原因
└── performance/                     # 性能优化（待添加）
```

---

## 🏷️ 标签索引

### #数据库 #SQL #MySQL
- [mysql-union-syntax](./database/mysql-union-syntax.md) — MySQL UNION 语法
- [mysql-join-update](./database/mysql-join-update.md) — MySQL 跨表关联更新
- [mysql-if-else](./database/mysql-if-else.md) — MySQL IF ELSE 条件判断
- [mysql-batch-bypass-gateway-timeout](./database/mysql-batch-bypass-gateway-timeout.md) — MySQL 分批执行绕过网关超时
- [mysql-table-backup](./database/mysql-table-backup.md) — MySQL 备份单表

### #Elasticsearch #搜索引擎 #分布式
- [elasticsearch-architecture-principles](./database/elasticsearch-architecture-principles.md) — Elasticsearch 架构原理与优化实战

### #命令行 #终端 #快捷键
- [tmux-shortcuts](./devops/tmux-shortcuts.md) — tmux 常用快捷键
- [tmux-session-create](./devops/tmux-session-create.md) — tmux Session 创建
- [tmux-session-management](./devops/tmux-session-management.md) — tmux Session 管理
- [tmux-resize-pane](./devops/tmux-resize-pane.md) — tmux 调整窗口和 Pane 大小
- [vim-delete](./devops/vim-delete.md) — Vim 内容删除操作
- [wc-l-line-count](./devops/wc-l-line-count.md) — wc -l 统计行数少一行的原因

### #Linux #网络
- [linux-dns-config](./devops/linux-dns-config.md) — Linux DNS 配置
- [linux-network-traffic-monitoring](./devops/linux-network-traffic-monitoring.md) — Linux 网络流量监控

### #Shell #Bash #Zsh
- [shell-env-detection](./devops/shell-env-detection.md) — Shell 环境探测
- [shell-env-shared-across-shells](./devops/shell-env-shared-across-shells.md) — 环境变量跨 shell 共享

### #Git #版本控制
- [git-config-username-email](./devops/git-config-username-email.md) — Git 配置用户名和邮箱

### #Supervisor #进程管理
- [supervisor-process-management](./devops/supervisor-process-management.md) — Supervisor 进程管理
- [supervisor-scheduling](./devops/supervisor-scheduling.md) — Supervisor 定时任务

### #对象存储 #OSS #S3
- [oss-connection-reuse](./devops/oss-connection-reuse.md) — OSS 连接复用
- [s3-copyobject-vs-uploadpartcopy](./devops/s3-copyobject-vs-uploadpartcopy.md) — S3 文件复制方式对比

### #Go #Golang
- [go-panic-recover](./backend/go-panic-recover.md) — Go 异常捕获
- [go-semaphore-weighted](./backend/go-semaphore-weighted.md) — Go 加权信号量

### #Kafka #消息队列
- [kafka-consumer-group](./middleware/kafka-consumer-group.md) — Kafka 消费者组机制

### #AI #具身智能 #机器人 #ROS
- [embodied-ai-overview](./ai/embodied-ai-overview.md) — 具身智能（Embodied AI）学习路线图
- [ros-introduction](./ai/ros-introduction.md) — ROS（Robot Operating System）简介

---

## 📋 按技术分类

### 🗄️ Database（数据库）

| 文件 | 描述 | 标签 |
|------|------|------|
| [elasticsearch-architecture-principles.md](./database/elasticsearch-architecture-principles.md) | Elasticsearch 架构原理：分片、副本、集群、选主与优化实战 | #Elasticsearch #搜索引擎 #Lucene #分布式系统 |
| [mysql-union-syntax.md](./database/mysql-union-syntax.md) | MySQL UNION 和 UNION ALL 语法详解 | #MySQL #UNION #SQL 语法 #查询优化 |
| [mysql-join-update.md](./database/mysql-join-update.md) | MySQL 跨表更新语句 | #MySQL #JOIN #子查询 #数据更新 |
| [mysql-if-else.md](./database/mysql-if-else.md) | MySQL IF ELSE 条件判断 | #MySQL #IF #条件判断 #存储过程 |
| [mysql-batch-bypass-gateway-timeout.md](./database/mysql-batch-bypass-gateway-timeout.md) | MySQL 分批执行绕过网关超时 | #MySQL #超时 #分批 #性能优化 |
| [mysql-table-backup.md](./database/mysql-table-backup.md) | MySQL 备份单表操作 | #MySQL #备份 #SQL |

### 🤖 AI（人工智能）

| 文件 | 描述 | 标签 |
|------|------|------|
| [embodied-ai-overview.md](./ai/embodied-ai-overview.md) | 具身智能（Embodied AI）学习路线图 | #具身智能 #Embodied AI #机器人 #强化学习 #VLA |
| [ros-introduction.md](./ai/ros-introduction.md) | ROS（Robot Operating System）简介 | #ROS #机器人 #ROS2 #运动控制 |

### 💻 Frontend（前端）

*待添加知识点*

**计划主题**：React Hooks、JavaScript 闭包、Vue 3 Composition API、TypeScript 类型系统、前端性能优化

### ⚙️ Backend（后端）

| 文件 | 描述 | 标签 |
|------|------|------|
| [go-panic-recover.md](./backend/go-panic-recover.md) | Go 异常捕获（panic/recover）机制 | #Go #Golang #panic #recover #错误处理 |
| [go-semaphore-weighted.md](./backend/go-semaphore-weighted.md) | golang.org/x/sync/semaphore 加权信号量 | #Go #semaphore #并发控制 #限流 |

### 🔌 Middleware（中间件）

| 文件 | 描述 | 标签 |
|------|------|------|
| [kafka-consumer-group.md](./middleware/kafka-consumer-group.md) | Kafka 消费者组（Consumer Group）机制 | #Kafka #消费者组 #Consumer Group #消息队列 |

### 🚀 DevOps（运维）

| 文件 | 描述 | 标签 |
|------|------|------|
| [git-config-username-email.md](./devops/git-config-username-email.md) | Git 配置用户名和邮箱 | #Git #配置 #版本控制 |
| [linux-dns-config.md](./devops/linux-dns-config.md) | Linux DNS 服务器配置 | #Linux #DNS #resolv.conf #systemd-resolved |
| [linux-network-traffic-monitoring.md](./devops/linux-network-traffic-monitoring.md) | Linux 网络流量查看方法 | #Linux #网络 #流量监控 #iftop #nload |
| [oss-connection-reuse.md](./devops/oss-connection-reuse.md) | OSS 对象存储连接复用机制 | #OSS #HTTP #Keep-Alive #连接池 #性能优化 |
| [s3-copyobject-vs-uploadpartcopy.md](./devops/s3-copyobject-vs-uploadpartcopy.md) | S3 CopyObject vs UploadPartCopy 区别 | #AWS #S3 #对象存储 #CopyObject #UploadPartCopy |
| [shell-env-detection.md](./devops/shell-env-detection.md) | Shell 环境探测 | #Shell #Bash #Zsh #进程管理 |
| [shell-env-shared-across-shells.md](./devops/shell-env-shared-across-shells.md) | Shell 环境变量跨 shell 共享 | #shell #zsh #bash #环境变量 #macOS |
| [supervisor-process-management.md](./devops/supervisor-process-management.md) | Supervisor 进程管理工具使用指南 | #Supervisor #进程管理 #Linux #运维 |
| [supervisor-scheduling.md](./devops/supervisor-scheduling.md) | Supervisor 定时任务调度 | #Supervisor #Cron #定时任务 #进程管理 |
| [tmux-shortcuts.md](./devops/tmux-shortcuts.md) | tmux 分屏、窗口操作快捷键 | #tmux #终端 #分屏 #快捷键 |
| [tmux-resize-pane.md](./devops/tmux-resize-pane.md) | tmux 调整窗口和 Pane 大小 | #tmux #终端复用 #pane 管理 |
| [tmux-session-create.md](./devops/tmux-session-create.md) | tmux Session 创建和管理 | #tmux #终端复用 #session 管理 |
| [tmux-session-management.md](./devops/tmux-session-management.md) | tmux Session 关闭和退出 | #tmux #终端复用 #session 管理 |
| [vim-delete.md](./devops/vim-delete.md) | Vim 内容删除操作 | #Vim #编辑器 #删除 #快捷键 |
| [wc-l-line-count.md](./devops/wc-l-line-count.md) | wc -l 统计行数少一行的原因 | #Linux #wc #命令行 #换行符 |

### ⚡ Performance（性能优化）

*待添加知识点*

**计划主题**：MySQL 查询优化、Redis 性能调优、前端性能优化、算法优化

---

## 🔍 快速查找

### 按关键词搜索

```bash
# 搜索所有文件中的关键词
grep -ri "关键字" knowledge-base/

# 在特定类别中搜索
grep -ri "索引优化" knowledge-base/database/

# 使用 rg (ripgrep) 更快搜索
rg "Elasticsearch" knowledge-base/database/ --type md
```

### 按标签搜索

```bash
# 搜索包含特定标签的文件
grep -rl "tags:.*MySQL" knowledge-base/database/

# 查看文件的标签
grep "^tags:" knowledge-base/database/*.md
```

### 按文件名查找

```bash
# 查找特定主题的文件
find knowledge-base/ -name "*mysql*.md"

# 查看某个类别的所有文件
ls -la knowledge-base/database/
```

### 统计知识点数量

```bash
# 统计各类别知识点数量
for dir in knowledge-base/*/; do
  count=$(find "$dir" -name "*.md" | wc -l)
  echo "$(basename $dir): $count 个知识点"
done
```

---

## 📝 笔记文件格式

每个知识点文件包含 YAML frontmatter 和标准内容结构：

```markdown
---
title: 知识点标题
date: YYYY-MM-DD HH:MM
tags: [标签 1, 标签 2, 标签 3]
category: 类别名称
---

# 知识点标题

## 核心概念
## 使用场景
## 代码示例
## 注意事项
## 最佳实践
## 参考资料
```

---

## 💡 使用建议

### 添加新知识点

1. **确定类别**：选择对应的技术目录（database/frontend/backend 等）
2. **命名文件**：使用 `技术名-知识点.md` 格式（如 `mysql-union-syntax.md`）
3. **添加 frontmatter**：包含 title、date、tags、category
4. **编写内容**：按照标准格式填写核心概念、代码示例等
5. **更新索引**：在 README.md 的分类表格中添加条目

### 查找知识点

1. **按类别浏览**：进入对应目录查看文件列表
2. **关键词搜索**：使用 `grep` 或 `rg` 搜索内容
3. **标签筛选**：搜索 frontmatter 中的 tags 字段
4. **查看索引**：在本文件的分类表格中查找

### 维护知识库

- **定期整理**：每月检查一次，合并重复内容
- **更新标签**：为新知识点添加详细标签
- **建立关联**：在相关文件之间添加交叉引用
- **清理过时**：标记或归档淘汰的技术笔记

---

## 🔗 交叉引用示例

在知识点文件中可以添加相关知识点链接：

```markdown
📌 **相关知识点**：
- [[mysql-join-update.md]] — MySQL 关联更新
- [[mysql-if-else.md]] — MySQL 条件判断
- [[../performance/mysql-query-optimization.md]] — MySQL 查询优化
```

---

## 📊 当前统计

| 类别            | 知识点数量  |
| ------------- | :----: |
| 🤖 ai         |   2    |
| 🗄️ database  | **6**  |
| 💻 frontend   |   0    |
| ⚙️ backend    |   2    |
| 🔌 middleware |   1    |
| 🚀 devops     | **15** |
| ⚡ performance |   0    |
| **总计**        | **26** |

---

**最后更新**：2026-05-27