---
title: MySQL IF ELSE 语法
date: 2026-04-02
tags: [MySQL, SQL, IF, 条件判断，存储过程]
category: database
---

# MySQL IF ELSE 语法

> MySQL 条件判断语句详解：IF() 函数与 IF 语句的使用

## 目录
- [两种使用场景](#两种使用场景)
- [IF() 函数](#一if-函数常用)
- [IF 语句](#二if-语句存储过程专用)
- [IF() vs CASE WHEN](#if-函数-vs-case-when)
- [相关函数](#相关函数)

---

## 两种使用场景

MySQL 中有两种 IF 相关语法，使用场景不同：

| 类型 | 语法 | 使用场景 |
|------|------|----------|
| **IF() 函数** | `IF(条件，真值，假值)` | SELECT、UPDATE 等 SQL 语句中 |
| **IF 语句** | `IF...THEN...ELSE...END IF` | 存储过程、函数、触发器中 |

---

## 一、IF() 函数（常用）⭐

### 基本语法

```sql
IF(条件，条件为真时的值，条件为假时的值)
```

### 简单示例

```sql
SELECT 
    name,
    score,
    IF(score >= 60, '及格', '不及格') AS result
FROM students;
```

**结果：**

| name | score | result |
|------|-------|--------|
| 张三 | 85 | 及格 |
| 李四 | 55 | 不及格 |

---

### 嵌套 IF（多层判断）

```sql
SELECT 
    name,
    score,
    IF(score >= 90, '优秀',
        IF(score >= 60, '及格', '不及格')
    ) AS grade
FROM students;
```

> ⚠️ **注意**：嵌套不建议超过 2 层，可读性会变差，多层判断推荐用 `CASE WHEN`

---

### 实际案例

#### 1. 判断库存状态

```sql
SELECT 
    product_name,
    stock,
    IF(stock > 0, '有货', '缺货') AS stock_status
FROM products;
```

#### 2. 计算折扣价格

```sql
SELECT 
    product_name,
    price,
    IF(member_level = 'VIP', price * 0.8, price) AS final_price
FROM orders;
```

#### 3. UPDATE 中使用

```sql
UPDATE employees
SET bonus = IF(performance >= 90, 5000, 1000);
```

#### 4. 配合 NULL 处理

```sql
SELECT 
    name,
    IF(phone IS NULL, '未登记', phone) AS phone_number
FROM users;
```

---

## 二、IF 语句（存储过程专用）

### 基本语法

```sql
IF 条件 THEN
    -- 条件为真时执行的语句
ELSEIF 条件 2 THEN
    -- 条件 2 为真时执行的语句
ELSE
    -- 以上条件都不满足时执行的语句
END IF;
```

### 存储过程示例

```sql
DELIMITER $$

CREATE PROCEDURE check_score(IN score INT)
BEGIN
    IF score >= 90 THEN
        SELECT '优秀' AS result;
    ELSEIF score >= 60 THEN
        SELECT '及格' AS result;
    ELSE
        SELECT '不及格' AS result;
    END IF;
END$$

DELIMITER ;

-- 调用
CALL check_score(85);  -- 输出：及格
```

---

### 完整案例：订单处理

```sql
DELIMITER $$

CREATE PROCEDURE process_order(IN order_status INT)
BEGIN
    IF order_status = 0 THEN
        UPDATE orders SET status = 1 WHERE status = 0;
        SELECT '订单已支付' AS message;
    ELSEIF order_status = 1 THEN
        UPDATE orders SET status = 2 WHERE status = 1;
        SELECT '订单已发货' AS message;
    ELSE
        SELECT '订单状态异常' AS message;
    END IF;
END$$

DELIMITER ;
```

---

## IF() 函数 vs CASE WHEN

| 特性 | IF() 函数 | CASE WHEN |
|------|----------|-----------|
| 语法简洁度 | ⭐⭐⭐ 更简洁 | ⭐⭐ 稍复杂 |
| 多层判断 | 嵌套写，可读性差 | 清晰，推荐 |
| 适用场景 | 简单二元判断 | 复杂多条件 |
| 可读性 | 一般 | **好** |

### 对比示例

```sql
-- IF() 嵌套（不推荐超过 2 层）
IF(score >= 90, 'A', IF(score >= 60, 'B', 'C'))

-- CASE WHEN（推荐）
CASE 
    WHEN score >= 90 THEN 'A'
    WHEN score >= 60 THEN 'B'
    ELSE 'C'
END
```

---

## 相关函数

### 1. IFNULL()

```sql
-- 如果值为 NULL，返回替代值
SELECT IFNULL(phone, '未登记') FROM users;
```

### 2. NULLIF()

```sql
-- 如果两个值相等，返回 NULL
SELECT NULLIF(status, 0) FROM orders;
```

### 3. COALESCE()

```sql
-- 返回第一个非 NULL 值
SELECT COALESCE(phone, mobile, '无联系方式') FROM users;
```

---

## 快速参考表

```sql
-- ✅ SQL 语句中用 IF() 函数
SELECT IF(条件，真值，假值) FROM 表;

-- ✅ 存储过程中用 IF 语句
IF 条件 THEN
    语句;
ELSE
    语句;
END IF;

-- ✅ 多条件推荐 CASE WHEN
CASE WHEN 条件 1 THEN 结果 1 ELSE 结果 2 END
```

---

## 使用建议

| 场景 | 推荐语法 |
|------|----------|
| 简单二元判断 | `IF()` 函数 |
| 多层条件判断 | `CASE WHEN` |
| 存储过程/函数 | `IF...THEN` 语句 |
| NULL 处理 | `IFNULL()` 或 `COALESCE()` |

---

## 标签

- #mysql #sql #if #条件判断 #存储过程 #数据库

---

*最后更新：2026-04-02*