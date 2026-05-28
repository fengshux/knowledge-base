---
title: MySQL 备份单表
date: 2026-05-20 14:00
tags: [MySQL, 备份, SQL]
category: database
---

# MySQL 备份单表

## 核心概念
MySQL 备份一张表有纯 SQL 方式和命令行工具两种路线。纯 SQL 方式适合快速在同一个库中创建副本；mysqldump 适合跨环境迁移和正式归档。

## 使用场景
- 修改表结构前做安全快照
- 将表数据复制出来做分析
- 跨服务器/跨环境迁移单表数据
- 导出特定条件的数据子集

## 基本语法

### 1. 纯 SQL —— 复制结构 + 数据（推荐）

```sql
-- 完整保留索引、主键、自增属性
CREATE TABLE table_backup LIKE original_table;

-- 复制全部数据
INSERT INTO table_backup SELECT * FROM original_table;
```

### 2. 纯 SQL —— 只复制数据（不保留索引）

```sql
-- 结构（简化版）+ 数据一步到位
CREATE TABLE table_backup AS SELECT * FROM original_table;
```

### 3. 纯 SQL —— 带条件备份

```sql
CREATE TABLE table_backup LIKE original_table;
INSERT INTO table_backup SELECT * FROM original_table WHERE create_time > '2024-01-01';
```

### 4. 快速重命名归档

```sql
RENAME TABLE original_table TO original_table_bak_20260520;
```

使用 `RENAME TABLE` 原表会消失，恢复方式同样快：

```sql
RENAME TABLE original_table_bak_20260520 TO original_table;
```

### 5. mysqldump（命令行，最完整）

```bash
# 导出单表结构和数据
mysqldump -u username -p database_name table_name > /path/to/backup.sql

# 只导出结构
mysqldump -u username -p --no-data database_name table_name > /path/to/schema.sql

# 只导出数据
mysqldump -u username -p --no-create-info database_name table_name > /path/to/data.sql
```

恢复：
```bash
mysql -u username -p database_name < /path/to/backup.sql
```

## 代码示例

```sql
-- 完整演示：备份 users 表
-- 1. 创建结构副本
CREATE TABLE users_bak_20260520 LIKE users;

-- 2. 复制数据
INSERT INTO users_bak_20260520 SELECT * FROM users;

-- 3. 验证
SELECT COUNT(*) AS original_count FROM users;
SELECT COUNT(*) AS backup_count FROM users_bak_20260520;
```

## 注意事项
- `CREATE TABLE AS SELECT` 不会复制索引、主键、自增、默认值
- `RENAME TABLE` 操作是原子的，但原表会不可用
- mysqldump 对大表耗时较长，建议在业务低峰期执行
- 千万级大表建议分批导出或使用 pt-archiver

## 最佳实践
- **日常快速备份** → `LIKE` + `INSERT SELECT`
- **操作前安全快照** → `RENAME TABLE`
- **正式归档/跨环境** → `mysqldump`
- **跨服务器迁移** → `mysqldump` + `mysql` 管道或 `SELECT INTO OUTFILE` + `LOAD DATA INFILE`

---

*最后更新：2026-05-20*
