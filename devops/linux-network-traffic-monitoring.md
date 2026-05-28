---
title: Linux 网络流量查看方法
date: 2026-05-15 12:00
tags: [Linux, 网络, 流量监控, iftop, nload, nethogs, vnstat, sar]
category: devops
---

# Linux 网络流量查看方法

## 核心概念
Linux 提供了多种查看网络流量的工具和方法，涵盖实时监控、长期统计、进程级分析和数据包级捕获等不同粒度的需求。大多数工具需要 root 权限读取网络接口。

## 常用工具一览

| 工具 | 特点 | 安装命令 |
|------|------|---------|
| `iftop` | 按连接显示实时带宽，类 top 风格 | `apt install iftop` |
| `nload` | 按网卡显示实时流量面板，界面美观 | `apt install nload` |
| `nethogs` | 按进程（PID）显示带宽占用 | `apt install nethogs` |
| `vnstat` | 长期流量统计（时/天/月/年），支持持久化 | `apt install vnstat` |
| `sar -n DEV` | 系统内置性能统计，支持历史回溯 | `apt install sysstat` |
| `/proc/net/dev` | 零依赖，内核直出累计值 | 无需安装 |
| `tcpdump` | 数据包级抓包分析 | `apt install tcpdump` |

## 详细用法

### iftop — 按连接查看带宽
```bash
sudo iftop                    # 默认所有接口
sudo iftop -i eth0            # 指定网卡
sudo iftop -n                 # 不解析主机名
sudo iftop -P                 # 显示端口号
```

### nload — 按网卡查看流量面板
```bash
nload                         # 默认所有网卡
nload eth0                    # 指定网卡
nload -m                      # 只显示 Mbps
```

### nethogs — 按进程查看流量
```bash
sudo nethogs                  # 按进程显示流量
sudo nethogs eth0             # 指定网卡
sudo nethogs -d 2             # 每 2 秒刷新
```

### vnstat — 长期流量统计
```bash
sudo vnstat -u -i eth0        # 初始化数据库
vnstat                        # 摘要
vnstat -h                     # 小时统计
vnstat -d                     # 天统计
vnstat -m                     # 月统计
vnstat -l                     # 实时流量
```

### sar — 系统性能统计（内建）
```bash
sar -n DEV 1 5                # 每秒刷新，显示5次
```
关键列：rxkB/s（接收 KB/s）、txkB/s（发送 KB/s）

### /proc/net/dev — 内核原始数据
```bash
cat /proc/net/dev
awk '/eth0/{print $1, $2, $10}' /proc/net/dev
```

## 场景推荐
- 看连接级占用 → `iftop`
- 看进程级占用 → `nethogs`
- 看网卡总带宽 → `nload`
- 长期统计趋势 → `vnstat`
- 历史回溯 → `sar -n DEV`
- 快速瞄一眼 → `/proc/net/dev`
- 数据包抓取 → `tcpdump`

## 注意事项
- 多数工具需要 `sudo` 权限
- iftop/nload/nethogs 默认可能未安装，需先 yum/apt install
- 云服务器建议配合云厂商流量监控面板一起看

---

*最后更新：2026-05-15*
