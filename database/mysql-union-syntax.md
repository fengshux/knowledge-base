---
title: MySQL UNION 语法
date: 2026-04-21 19:05
tags: [MySQL, UNION, SQL 语法，查询优化，数据合并]
category: database
---

# MySQL UNION 语法

## 核心概念

UNION 操作用于合并两个或多个 SELECT 语句的结果集。UNION 自动去重，UNION ALL 保留所有行（包括重复）。

## 使用场景

- 合并多个结构相似的表的数据
- 从同一表的不同查询条件合并结果
- 多表全文搜索
- 历史数据与当前数据合并查询
- 数据归档场景

## 基本语法

```sql
-- UNION（自动去重）
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;

-- UNION ALL（保留重复）
SELECT column1, column2 FROM table1
UNION ALL
SELECT column1, column2 FROM table2;
```

## UNION vs UNION ALL

| 特性 | UNION | UNION ALL |
|------|-------|-----------|
| 去重 | ✅ 自动去除重复行 | ❌ 保留所有行（包括重复） |
| 性能 | 较慢（需要排序去重） | 较快（直接合并） |
| 使用场景 | 需要唯一结果集 | 允许重复或确定无重复 |

## 代码示例

### 添加来源标识

```sql
SELECT name, age, 'CN' AS country FROM users_cn
UNION ALL
SELECT name, age, 'US' AS country FROM users_us;
```

### ORDER BY 排序（放在最后）

```sql
SELECT name, age FROM users_cn WHERE age >= 28
UNION ALL
SELECT name, age FROM users_us WHERE age >= 28
ORDER BY age DESC, name ASC;
```

### 每个 SELECT 单独 LIMIT（需要括号）

```sql
(SELECT * FROM table1 ORDER BY id DESC LIMIT 10)
UNION ALL
(SELECT * FROM table2 ORDER BY id DESC LIMIT 10)
LIMIT 20;
```

### 结合 WHERE 过滤

```sql
SELECT name, age FROM (
    SELECT name, age FROM users_cn
    UNION ALL
    SELECT name, age FROM users_us
) AS all_users
WHERE age > 28
ORDER BY age;
```

## 注意事项

- [ ] **列数必须相同**：所有 SELECT 语句必须返回相同数量的列
- [ ] **列类型必须兼容**：对应位置的列数据类型必须兼容
- [ ] **列名以第一个 SELECT 为准**：结果集的列名使用第一个 SELECT 语句的列名
- [ ] **ORDER BY 位置**：如果需要排序，ORDER BY 必须放在最后一个 SELECT 之后
- [ ] **LIMIT 使用括号**：如果每个 SELECT 都需要 LIMIT，要用括号包裹每个 SELECT
- [ ] **性能考虑**：UNION 需要排序去重，数据量大时优先使用 UNION ALL
- [ ] **NULL 值处理**：NULL 值在去重时被视为相等

## 最佳实践

1. **优先使用 UNION ALL**：避免不必要的去重开销
2. **添加来源标识列**：便于区分数据来源
3. **统一列别名**：多个表列名不一致时使用别名统一
4. **先合并再分页**：全局排序分页时，先 UNION 再 ORDER BY + LIMIT

## 常见错误

### ❌ 错误：列数不匹配

```sql
SELECT id, name FROM table1
UNION
SELECT id FROM table2;
```

### ✅ 正确：列数相同

```sql
SELECT id, name FROM table1
UNION
SELECT id, NULL AS name FROM table2;
```

### ❌ 错误：ORDER BY 位置错误

```sql
SELECT * FROM table1 ORDER BY id
UNION
SELECT * FROM table2;
```

### ✅ 正确：ORDER BY 放在最后

```sql
SELECT * FROM table1
UNION
SELECT * FROM table2
ORDER BY id;
```

## 实际应用场景

### 场景 1：合并历史表

```sql
-- 合并 2023 年和 2024 年的订单数据
SELECT order_id, user_id, amount, '2023' AS year FROM orders_2023
UNION ALL
SELECT order_id, user_id, amount, '2024' AS year FROM orders_2024
WHERE amount > 1000;
```

### 场景 2：多表搜索

```sql
-- 在多个表中搜索关键词
SELECT 'user' AS type, id, name FROM users WHERE name LIKE '%张三%'
UNION ALL
SELECT 'product' AS type, id, name FROM products WHERE name LIKE '%张三%'
UNION ALL
SELECT 'article' AS type, id, title AS name FROM articles WHERE title LIKE '%张三%';
```

### 场景 3：数据归档查询

```sql
-- 查询活跃表 + 归档表的数据
SELECT * FROM active_logs WHERE created_at > '2024-01-01'
UNION ALL
SELECT * FROM archived_logs WHERE created_at > '2024-01-01';
```

## 参考资料

- [MySQL 官方文档 - UNION](https://dev.mysql.com/doc/refman/8.0/en/union.html)
- [W3Schools MySQL UNION](https://www.w3schools.com/mysql/mysql_union.asp)

---

*最后更新：2026-04-21*
