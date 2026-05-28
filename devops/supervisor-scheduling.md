---
title: Supervisor 定时任务调度
date: 2026-04-22 10:45
tags: [Supervisor, Cron, 定时任务，进程管理，systemd]
category: devops
---

# Supervisor 定时任务调度

## 核心概念

Supervisor 是一个进程管理工具，**本身不支持定时启动任务**。它主要用于启动、停止、重启和监控进程，但不具备 Cron 那样的定时调度功能。

## 使用场景

- 需要定时启动/停止由 Supervisor 管理的进程
- 需要结合进程管理和定时调度功能
- 需要在固定时间执行一次性任务

## 基本方案

### 方案一：Supervisor + Cron 组合（推荐）

```ini
# /etc/supervisor/conf.d/my-task.conf
[program:my-task]
command=/path/to/script.sh
autostart=false
autorestart=false
stdout_logfile=/var/log/my-task.log
```

```bash
# crontab -e
# 每天凌晨 2 点启动任务
0 2 * * * /usr/bin/supervisorctl start my-task
```

### 方案二：systemd timers（Linux）

```ini
# /etc/systemd/system/my-task.service
[Unit]
Description=My Task
After=network.target

[Service]
Type=oneshot
ExecStart=/path/to/script.sh

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/my-task.timer
[Unit]
Description=Run daily at 2 AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### 方案三：APScheduler（Python）

```python
from apscheduler.schedulers.blocking import BlockingScheduler
from apscheduler.triggers.cron import CronTrigger

scheduler = BlockingScheduler()

@scheduler.scheduled_job(CronTrigger.from_crontab('0 2 * * *'))
def my_task():
    print("Task executed")

scheduler.start()
```

## 代码示例

### 包装脚本示例

```bash
#!/bin/bash
# /usr/local/bin/scheduled-task-wrapper.sh

LOG_FILE="/var/log/scheduled-task.log"

echo "[$(date)] Starting task" >> $LOG_FILE

# 启动任务
supervisorctl start my-task

# 等待任务完成
sleep 3600

# 停止任务
supervisorctl stop my-task

echo "[$(date)] Task completed" >> $LOG_FILE
```

## 注意事项

- [ ] Supervisor 配置的进程应设置 `autostart=false` 避免开机自启
- [ ] 确保执行 supervisorctl 的用户有足够权限
- [ ] 定时任务要考虑幂等性，避免重复执行导致问题
- [ ] 配置日志轮转避免日志文件过大
- [ ] 添加错误处理和告警机制

## 最佳实践

1. **明确分工**：Supervisor 负责进程管理，Cron 负责定时调度
2. **监控告警**：为定时任务添加失败告警
3. **日志集中**：统一管理 Supervisor 和 Cron 日志
4. **配置版本化**：使用 Git 管理配置文件
5. **测试验证**：生产部署前在测试环境充分验证

## 实际应用场景

**数据备份任务**：

```ini
# supervisor 配置
[program:backup-task]
command=/path/to/backup.sh
autostart=false
autorestart=false
```

```bash
# crontab 配置
0 2 * * * /usr/bin/supervisorctl start backup-task
```

## 工具对比

| 工具 | 进程管理 | 定时调度 | 日志管理 | 自动重启 |
|------|---------|---------|---------|---------|
| Supervisor | ✅ | ❌ | ✅ | ✅ |
| Cron | ❌ | ✅ | ⚠️ | ❌ |
| systemd | ✅ | ✅ | ✅ | ✅ |
| Celery Beat | ⚠️ | ✅ | ✅ | ✅ |

---

*最后更新：2026-04-22*
