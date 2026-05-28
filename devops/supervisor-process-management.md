---
title: Supervisor 进程管理工具使用指南
date: 2026-04-24 10:30
tags: [Supervisor, 进程管理，Linux, 运维，服务部署]
category: devops
---

# Supervisor 进程管理工具使用指南

## 核心概念

Supervisor 是一个用 Python 编写的进程管理系统，它允许用户监控和控制类 Unix 系统上的多个进程。主要功能包括：
- 自动启动和重启进程
- 进程状态监控
- 日志管理
- 统一的命令行控制界面

## 使用场景

- 需要长期运行的后台服务（如 Web 应用、API 服务）
- 需要自动重启的守护进程
- 需要集中管理多个进程的场景
- 生产环境的服务部署和管理

## 基本语法/原理

### 配置文件位置

- 主配置文件：`/etc/supervisord.conf`
- 子配置文件目录：`/etc/supervisor/conf.d/`
- 日志文件：`/var/log/supervisor/`

### 配置文件格式

```ini
[program:example-app]
command=/path/to/your/app.py
directory=/path/to/your/app
autostart=true
autorestart=true
stderr_logfile=/var/log/supervisor/example-app.err.log
stdout_logfile=/var/log/supervisor/example-app.out.log
user=www-data
environment=PYTHONPATH="/path/to/your/app"
```

## 代码示例

### 1. 添加新应用服务

```bash
# 步骤 1: 将 supervisor 配置文件移动到 conf.d 目录
sudo mv /path/to/example-app.conf /etc/supervisor/conf.d/

# 步骤 2: 重新读取配置
sudo supervisorctl reread

# 步骤 3: 添加服务（更新配置）
sudo supervisorctl update
# 或者使用 add 命令添加特定服务
sudo supervisorctl add example-app

# 步骤 4: 启动服务
sudo supervisorctl start example-app

# 步骤 5: 查看服务状态
sudo supervisorctl status
```

### 2. 常用命令示例

```bash
# 查看所有服务状态
supervisorctl status

# 查看单个服务状态
supervisorctl status example-app

# 启动服务
supervisorctl start example-app

# 停止服务
supervisorctl stop example-app

# 重启服务
supervisorctl restart example-app

# 重新加载配置
supervisorctl reread
supervisorctl update

# 进入交互式控制台
supervisorctl

# 在控制台内执行命令
> status
> start example-app
> stop all
> exit
```

### 3. 完整配置文件示例

```ini
; /etc/supervisor/conf.d/example-app.conf
[program:example-app]
; 启动命令
command=/usr/bin/python3 /opt/example-app/main.py

; 工作目录
directory=/opt/example-app

; 是否自动启动
autostart=true

; 自动重启策略
autorestart=true

; 重启延迟
startsecs=10

; 最大重启次数
startretries=3

; 日志配置
stderr_logfile=/var/log/supervisor/example-app.err.log
stdout_logfile=/var/log/supervisor/example-app.out.log
stderr_logfile_maxbytes=10MB
stdout_logfile_maxbytes=10MB
stderr_logfile_backups=10
stdout_logfile_backups=10

; 运行用户
user=www-data

; 环境变量
environment=PYTHONPATH="/opt/example-app",ENV="production"

; 进程组（可选）
group=example

; 停止等待时间
stopwaitsecs=10

; 停止信号
stopsignal=TERM
```

## 注意事项

- [ ] **配置文件权限**：确保配置文件属于 root 用户，权限为 644
- [ ] **日志目录权限**：确保 `/var/log/supervisor/` 目录存在且可写
- [ ] **配置生效**：修改配置后必须执行 `reread` 和 `update` 才能生效
- [ ] **服务命名**：服务名称不能包含空格和特殊字符
- [ ] **路径问题**：配置文件中的路径必须使用绝对路径
- [ ] **环境变量**：在 `environment` 中设置的环境变量只对当前进程有效
- [ ] **权限问题**：使用 `user` 参数指定运行用户，避免以 root 身份运行应用

### 常见错误和解决方案

**错误 1：配置不生效**
```bash
# 解决：重新读取并更新配置
sudo supervisorctl reread
sudo supervisorctl update
```

**错误 2：服务启动失败**
```bash
# 查看错误日志
tail -f /var/log/supervisor/example-app.err.log

# 检查配置文件语法
sudo python3 -c "import configparser; c = configparser.ConfigParser(); c.read('/etc/supervisord.conf')"
```

**错误 3：权限不足**
```bash
# 确保使用 sudo 执行
sudo supervisorctl start example-app

# 或者将当前用户加入 supervisor 组
sudo usermod -a -G supervisor $USER
```

## 最佳实践

### 1. 配置文件管理

- 将每个应用的配置独立放在 `/etc/supervisor/conf.d/` 目录下
- 使用有意义的文件名，如 `example-app.conf`
- 在配置文件中添加注释说明关键参数

### 2. 日志管理

- 为每个服务配置独立的日志文件
- 设置合理的日志轮转（maxbytes 和 backups）
- 定期清理旧日志文件

### 3. 进程监控

- 配置合理的 `startsecs` 和 `startretries` 参数
- 使用 `autorestart=true` 确保服务异常时自动重启
- 配置邮件或 webhook 告警（需要额外配置）

### 4. 安全配置

- 避免以 root 用户运行应用进程
- 限制进程的文件访问权限
- 使用最小权限原则配置运行用户

### 5. 性能优化

- 合理设置进程数（numprocs 参数）
- 配置适当的停止等待时间，避免强制终止导致数据丢失
- 使用进程组管理相关服务

## 实际应用场景

### 场景 1：部署 Python Web 应用

```ini
[program:flask-app]
command=/var/www/flask-app/venv/bin/gunicorn -w 4 -b 127.0.0.1:8000 app:app
directory=/var/www/flask-app
user=www-data
autostart=true
autorestart=true
stderr_logfile=/var/log/supervisor/flask-app.err.log
stdout_logfile=/var/log/supervisor/flask-app.out.log
```

### 场景 2：管理多个微服务

```ini
[program:api-gateway]
command=/opt/microservices/api-gateway/run.sh
autostart=true
autorestart=true
priority=100

[program:user-service]
command=/opt/microservices/user-service/run.sh
autostart=true
autorestart=true
priority=200
depends_on=api-gateway

[program:order-service]
command=/opt/microservices/order-service/run.sh
autostart=true
autorestart=true
priority=200
depends_on=api-gateway
```

### 场景 3：批量管理进程组

```ini
[group:example-services]
programs=example-app,example-api,example-worker
priority=100
```

```bash
# 启动整个服务组
supervisorctl start example-services:

# 停止整个服务组
supervisorctl stop example-services:

# 查看组内所有服务状态
supervisorctl status example-services:
```

## 参考资料

- [Supervisor 官方文档](http://supervisord.org/)
- [Supervisor 配置手册](http://supervisord.org/configuration.html)
- [Linux 进程管理最佳实践](https://linuxhandbook.com/process-management/)

---

*最后更新：2026-04-24*
