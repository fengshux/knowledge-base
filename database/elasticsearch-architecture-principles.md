---
title: Elasticsearch 架构原理与优化实战
date: 2026-05-27 18:00
tags: [Elasticsearch, 搜索引擎, 分布式系统, Lucene, 架构原理]
category: database
---

# Elasticsearch 架构原理与优化实战

> 📚 笔记来源：《Elasticsearch源码解析与优化实战》— 张超，电子工业出版社
> 阅读进度：20%

---

## 一、ES 基础架构：索引、分片与 Lucene

### 层级结构

```
ES 索引 (Index)
  ├── 分片 (Shard) — 一个 Lucene 索引
  │     ├── 分段 (Segment) — 一个倒排索引
  │     │     ├── 倒排列表
  │     │     └── 正排存储（doc values）
  │     └── 事务日志 (Translog)
  └── ...
```

- **一个 ES 索引由多个分片组成**，每个分片在物理上就是一个 Lucene 索引
- **Lucene 索引由多个分段（Segment）组成**，每个分段都是一个独立的倒排索引
- **Refresh 操作**：ES 每次 refresh 都会生成一个新的分段，包含若干文档的数据。这意味着 refresh 频率越高，分段越多，但搜索可见性越及时
- **_type 字段**：用于区分同一索引中不同细分类型，数据的整体模式应当相同或相似，不适合完全不同类型的数据

### 关键原理

分段是不可变的（immutable），写入的数据先进入内存 buffer，refresh 后生成不可变分段写入磁盘。这种设计带来了：
- 不需要加锁，读性能高
- 分段可以缓存，提高查询速度
- 可以利用操作系统的 page cache

---

## 二、数据副本模型与一致性

### 主从模式

ES 的数据副本模型基于 **主从模式（Primary-Backup）**，区别于 HDFS 和 Cassandra 的对等模式（Peer-to-Peer）。

```
协调节点 → Primary Shard（主分片） → Replica Shards（从分片）
```

### 副本一致性不变式

令 P 为主副本，R 为从副本，以下不变式始终保持：

```
committed_R ⊆ committed_P ⊆ prepared_R
```

含义：
- 从副本已提交的数据集 ⊆ 主副本已提交的数据集 ⊆ 从副本已预处理的数据集
- 保证**主从数据最终一致**
- 确保故障切换时**不丢失已提交的数据**

### 版本号与乐观锁

ES 使用版本号机制实现乐观锁：
- 写请求中指定文档的版本号
- 如果文档当前版本与请求中指定的版本号不同，请求会失败
- 类似于其他数据库（如 MySQL 的乐观锁）的实现方式

### Primary Terms 机制

当主分片发生变更（如被提升）时，**Primary Terms 递增**：
- 由主节点统一分配和递增
- 用于区分不同的主分片任期
- 配合 Sequence Numbers 实现精确的故障恢复和读写一致性

---

## 三、集群管理与选主

### 防止脑裂

```
discovery.zen.minimum_master_nodes
```

这是**防止脑裂、防止数据丢失的极其重要的参数**。

推荐值：`N/2 + 1`（N 为候选主节点数）
- 例如 3 个候选主节点 → 设置为 2
- 例如 5 个候选主节点 → 设置为 3

### 节点发现

每个节点都知道主分片和副分片分配到了哪里，集群元信息在所有节点间同步。

---

## 四、分片分配与恢复

### Allocation ID

- 每个分片有自己**唯一的 Allocation ID**
- 集群元信息中维护一个列表，记录哪些分片拥有最新数据
- 当节点重启或网络分区恢复时，通过 Allocation ID 判断数据是否有效

### 调试工具

**cluster allocation explain API**：调试分片分配问题的好工具，可以查看某个分片为什么未被分配、被延迟分配的原因等。

### 陈旧分片恢复

```
POST /_cluster/reroute
{
  "commands": [
    {
      "allocate_stale_primary": {
        "index": "my-index",
        "shard": 0,
        "node": "node-1",
        "accept_data_loss": true
      }
    }
  ]
}
```

`allocate_stale_primary` 用于将一个陈旧的分片分配为主分片。**谨慎使用**——会接受数据丢失的风险。

---

## 五、写入与刷新机制

### 写入流程

```
客户端请求 → 协调节点 → 路由到主分片 → 写入Lucene + Translog → 同步到副本 → 返回成功
```

### Refresh 与 Segment

- **Refresh**（默认 1s）：将 buffer 中的文档生成新的分段，使数据可见
  - 代价：生成新分段，消耗 I/O
  - 收益：近实时搜索
- **Flush**：将分段从内存刷到磁盘，清空 translog
- **Merge**：后台合并小分段为大分段，控制分段数量

### 优化建议

| 场景 | 建议 |
|:----|:----|
| 写入吞吐优先 | 调大 refresh_interval（如 30s），或写入完成后再 refresh |
| 搜索实时性要求高 | 保持默认 1s 或更小 |
| 大量写入 | 使用 bulk API，调整线程池和队列大小 |

---

## 六、优化实战

### 写入速度优化

- 使用 bulk API 批量写入（建议每批 5-15MB）
- 增大 refresh_interval
- 禁用副分片（`index.number_of_replicas: 0`），写入完成后再启用
- 调整 `index.translog.sync_interval` 和 `index.translog.durability`
- 使用 SSD 磁盘

### 搜索速度优化

- 合理设计分片数量（每个分片 20-40GB）
- 使用过滤器（filter）代替 query 来利用缓存
- 启用 doc values 做聚合排序
- 关闭不需要的 _source 字段
- 使用别名实现零停机索引重建

### 内存与 GC 优化

- **堆内存**：不超过物理内存的 50%，且不超过 32GB（避免压缩指针失效）
- **预留内存**：另一半留给 OS 做 page cache（Lucene 重度依赖）
- GC 策略：默认 CMS → G1GC（JDK 9+）

---

## 七、诊断工具

| 工具/API | 用途 |
|:---------|:-----|
| `_cat/health` | 集群健康状态 |
| `_cat/nodes` | 节点信息 |
| `_cat/indices` | 索引级别信息 |
| `_cat/shards` | 分片分配详情 |
| `_cluster/allocation/explain` | 分片未分配原因诊断 |
| `_node/stats` | 节点级统计数据 |
| `_nodes/hot_threads` | 热点线程栈 |
| `_cluster/pending_tasks` | 等待中的集群任务 |

---

## 参考资料

- 《Elasticsearch源码解析与优化实战》张超 著
- [Elasticsearch 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Elasticsearch: The Definitive Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)

---

*最后更新：2026-05-27*