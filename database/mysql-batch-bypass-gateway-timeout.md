---
title: MySQL 分批执行绕过网关超时 [gw]
date: 2026-05-20 18:00
tags: [MySQL, 超时, 网关, 分批, UPDATE, 性能优化]
category: database
---

# MySQL 分批执行绕过网关超时 [gw]

## 核心概念

当数据库前端有网关/代理（如阿里云 RDS 代理、ProxySQL、MaxScale），执行大表全量 `INSERT SELECT` 或全表 `UPDATE` 时会报 `The upstream server is timing out [gw]`。这是因为：

```
客户端 → [网关/代理 gw：有硬超时限制，通常 60~300s] → MySQL 实例
```

调大客户端超时（`net_read_timeout` 等）没用，因为 **网关层的超时用户无法修改**。唯一的绕过方式是把一个大 SQL 拆成多个**能在超时限制内跑完的小 SQL 分批执行**。

## 使用场景

- `INSERT INTO bak SELECT * FROM big_table` 超时
- `UPDATE big_table SET col = xxx` 超时
- 任何全表 DML 操作被网关拦截
- 云数据库（阿里云 RDS、AWS RDS Proxy 等）的大表操作

## 核心原则

1. **按主键范围分批** — 利用自增主键 `id BETWEEN ? AND ?` 做范围切割
2. **每批 < 网关超时** — 先试 5000~10000 行，看执行时间，调整批次大小
3. **幂等保护** — 加上条件判断，防止脚本重跑后数据被重复处理
4. **批间暂停** — `sleep 0.5~1s` 让网关和数据库喘口气

## 代码示例

### 分批 UPDATE（带幂等保护）

```sql
-- 每批 10000 行，只处理还未加前缀的行
UPDATE nvwa_oss_path
SET video_path = CONCAT('ods-robot-nvwa/', video_path)
WHERE id BETWEEN 1 AND 10000
  AND (video_path IS NOT NULL)
  AND video_path NOT LIKE 'ods-robot-nvwa/%';
```

### Shell 脚本自动分批

```bash
#!/bin/bash
# 按 id 范围分批执行，自动重试，带幂等保护

TABLE="your_table"
BATCH_SIZE=10000

# 获取 id 范围
ID_RANGE=$(mysql -h"$DB_HOST" -u"$DB_USER" -p"$DB_PASS" "$DB_NAME" \
  -N -e "SELECT MIN(id), MAX(id) FROM $TABLE;")
MIN_ID=$(echo "$ID_RANGE" | awk '{print $1}')
MAX_ID=$(echo "$ID_RANGE" | awk '{print $2}')

START=$MIN_ID
while [ $START -le $MAX_ID ]; do
  END=$((START + BATCH_SIZE - 1))
  mysql -h"$DB_HOST" -u"$DB_USER" -p"$DB_PASS" "$DB_NAME" -e "
    UPDATE $TABLE SET col = xxx
    WHERE id BETWEEN $START AND $END
      AND (安全条件防止重复处理);
  "
  START=$((END + 1))
  sleep 0.5
done
```

### 分批 INSERT SELECT

```sql
-- 方法 1：按主键范围（推荐，有索引走 range scan）
INSERT INTO bak_table SELECT * FROM src_table
WHERE id BETWEEN 1 AND 5000;

-- 方法 2：按 MAX(id) 递增（适合没有自增主键但有有序字段）
INSERT INTO bak_table SELECT * FROM src_table
WHERE id > (SELECT MAX(id) FROM bak_table)
ORDER BY id
LIMIT 5000;
```

### 分批后验证完整性

```sql
-- 原表行数
SELECT COUNT(*) FROM src_table;

-- 备份表行数
SELECT COUNT(*) FROM bak_table;

-- 两者一致则完整
SELECT CASE
  WHEN (SELECT COUNT(*) FROM src_table) = (SELECT COUNT(*) FROM bak_table)
  THEN '✅ 完整'
  ELSE '❌ 不完整，需要补漏'
END AS result;
```

## 注意事项

- **必须有有序主键**：分批依赖主键范围，建议用自增 `BIGINT` 主键
- **幂等条件不能少**：不加条件的话，脚本中断重跑会把已处理的行再次处理（如前缀叠加）
- **批次大小需要调优**：从 5000 开始试，如果很快（<5s）就翻倍；如果报超时就减半
- **业务低峰期执行**：UPDATE 会对行加锁，生产环境建议凌晨跑
- **检查是否有触发器**：批量 UPDATE 可能触发不必要的触发器
- **binlog 压力**：大事务会写入大量 binlog，分批后每批较小，binlog 同步压力更分散

## 其他绕过网关超时的方法

| 方法 | 说明 | 适用场景 |
|------|------|---------|
| **分批 SQL** | 主键范围切割 | **最通用，推荐首选** |
| **直连实例** | 绕过网关直接连 MySQL 实例 | 有私网直连地址时可用 |
| **mysqldump + mysql** | dump 成小 INSERT 后再导入 | 备份/迁移场景 |
| **LOAD DATA INFILE** | 文件导入最快 | 数据量大、能导出文件时 |
| **DTS/数据迁移服务** | 云平台官方工具 | 云数据库用户 |

---

*最后更新：2026-05-20*