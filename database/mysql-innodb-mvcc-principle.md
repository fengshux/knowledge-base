---
title: InnoDB MVCC 多版本并发控制原理
date: 2026-05-28 12:11
tags: [MySQL, InnoDB, MVCC, 事务, Undo Log, 隔离级别]
category: database
---

# InnoDB MVCC 多版本并发控制原理

## 核心概念

MVCC（Multi-Version Concurrency Control，多版本并发控制）是 InnoDB 引擎实现事务隔离的核心机制。它通过维护数据的多个历史版本，使不同事务在读取同一行数据时能看到各自"一致性快照"，从而实现**读不阻塞写、写不阻塞读**的并发能力。

关键组成要素：
- **trx_id（事务ID）**：每行数据隐藏字段，记录最后一次修改该行的事务编号
- **read-view（读取视图）**：事务开始读取数据时建立的"可见范围快照"
- **Undo Log（回滚日志）**：记录每行数据的历史变更版本，形成版本链
- **回滚指针（rollback pointer）**：行中隐藏字段，指向 Undo 日志中的前一版本

## 适用隔离级别

| 隔离级别 | 是否兼容 MVCC | 说明 |
|---------|:------------:|------|
| READ UNCOMMITTED | ❌ 不兼容 | 查询不读版本快照，直接读最新版本 |
| READ COMMITTED | ✅ 兼容 | 每条语句执行时重新生成 read-view（不可重复读） |
| REPEATABLE READ | ✅ 兼容 | 事务首次读取时生成 read-view，整个事务复用（可重复读） |
| SERIALIZABLE | ❌ 不兼容 | 读取时锁定每一行返回的记录 |

## 基本原理

### 1. 行的隐藏字段

InnoDB 聚簇索引的每行记录包含两个关键的隐藏列：
- **`DB_TRX_ID`**：最近修改该行的事务 ID
- **`DB_ROLL_PTR`**：回滚指针，指向 Undo 日志中该行的前一版本

### 2. 事务ID分配

- 事务 ID 在事务**首次执行任何读取或写入操作时**分配（而不是事务开始时）
- 事务 ID 全局递增，新事务的 ID 一定大于旧事务的 ID

### 3. Read-View（读取视图）

当事务执行 SELECT 时，InnoDB 会生成一个 read-view，其中包含：

- **`creator_trx_id`**：创建该视图的事务 ID
- **`m_ids`**：生成 read-view 时，当前系统中活跃（未提交）的事务 ID 列表
- **`min_trx_id`**：`m_ids` 中的最小值（最小活跃事务 ID）
- **`max_trx_id`**：生成 read-view 时，系统已分配给下一个事务的 ID（即最大事务 ID + 1）

### 4. 可见性判断规则

读取一行时，比较该行的 `trx_id` 与 read-view 中的信息：

```
trx_id == creator_trx_id       → ✅ 可见（自己修改的数据）
trx_id < min_trx_id            → ✅ 可见（已提交的旧事务）
min_trx_id ≤ trx_id < max_trx_id
  ├── trx_id 在 m_ids 中      → ❌ 不可见（活跃未提交的事务）
  └── trx_id 不在 m_ids 中    → ✅ 可见（已提交的事务）
trx_id ≥ max_trx_id           → ❌ 不可见（未来事务，生成视图后启动的）
```

### 5. Undo Log 版本链

当行数据对当前事务不可见时，InnoDB 沿着 `DB_ROLL_PTR` 链向前查找：

```
当前行 (trx_id=150) → Undo v1 (trx_id=120) → Undo v2 (trx_id=90) → ...
```

沿着版本链**逐版本回滚**，直到找到一个 `trx_id` 满足 read-view 可见性规则的版本。

### 6. 删除标记

- 删除操作不会立即物理删除行记录
- 在行记录的 `info flags` 中设置 **"deleted" 位**作为删除标记
- Undo 日志中也会跟踪该删除标记
- 如果版本链一路回到某个被标记为"删除"的版本，且所有后续修改都已回滚，InnoDB 最终会向读取视图返回"此行不存在"的信号

## 工作流程

### 查询一致性快照流程

```
1. 事务 T 执行 SELECT
       ↓
2. InnoDB 为 T 生成 read-view（或复用已有的）
       ↓
3. 扫描聚簇索引，读取一行
       ↓
4. 检查该行 DB_TRX_ID 与 read-view 的可见性
       ↓
   ├── 可见 → 直接返回该行数据
   │
   └── 不可见 → 沿 DB_ROLL_PTR 链回溯 Undo Log
           ↓
        找到可见版本 → 返回该版本的数据
           ↓
        一直回退到删除标记 → 返回"此行不存在"
```

### 更新流程

```
1. 事务 T 修改一行数据
       ↓
2. 将当前行的旧版本写入 Undo Log
       ↓
3. 修改行的 DB_TRX_ID = T 的事务 ID
       ↓
4. DB_ROLL_PTR 指向刚才写入的 Undo Log 记录
       ↓
5. 提交（commit）：
   └── 释放锁，标记事务结束
   
6. 回滚（rollback）：
   └── 沿 Undo Log 链恢复到修改前的状态
```

## MVCC 与隔离级别的交互

### REPEATABLE READ（可重复读）

- 事务中**第一次 SELECT** 时生成 read-view
- **整个事务期间**复用该 read-view
- 保证了同一事务内多次读取同一行数据结果一致（可重复读）
- 这就是 InnoDB 默认隔离级别

### READ COMMITTED（读已提交）

- **每条 SELECT 语句**执行时都重新生成 read-view
- 因此同一事务内，后续 SELECT 可能看到其他事务已提交的更改（不可重复读）
- 无需加锁即可读取已提交的最新数据

## Undo Log 的清理（Purge）

- Undo Log 不能无限增长
- 当所有需要某个历史版本的事务都已结束时，该版本就可以被物理清理
- InnoDB 的后台线程（Purge Thread）负责清理不再需要的 Undo Log
- 长事务会阻止 Undo Log 的清理，可能导致 Undo 表空间膨胀

## 注意事项

- **长事务避免**：长时间运行的事务会阻止 Undo Log 的清理，导致 Undo 表空间膨胀，影响性能
- **READ UNCOMMITTED 的脏读**：由于不遵循 MVCC 可见性规则，可能会读到其他事务未提交的修改（脏读）
- **SERIALIZABLE 的性能代价**：行级锁会降低并发性能，仅在强一致性要求时使用
- **唯一索引与 MVCC**：唯一索引的冲突检测不受 MVCC 保护，插入时需加行锁（next-key lock）

## 最佳实践

- 默认使用 **REPEATABLE READ**（InnoDB 默认）或 **READ COMMITTED**，充分利用 MVCC 的并发能力
- 避免长事务，合理拆分大事务为多个小事务
- 监控 Undo 表空间大小，及时发现异常增长
- 对于报表类只读查询，使用 `START TRANSACTION READ ONLY` 明确声明只读，InnoDB 可进一步优化

## 参考资料

- [MySQL 官方文档 - InnoDB MVCC](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)
- [MySQL 官方文档 - InnoDB 事务隔离级别](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)

---

*最后更新：2026-05-28*