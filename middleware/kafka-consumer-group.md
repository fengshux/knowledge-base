---
title: Kafka 消费者组（Consumer Group）机制
date: 2026-05-25 12:04
tags: [Kafka, 消费者组, Consumer Group, 消息队列]
category: middleware
---

# Kafka 消费者组（Consumer Group）机制

## 核心概念

Kafka 消费者组是 Kafka 实现**点对点消费**和**水平扩展**的核心机制。同一个 `groupId` 下的所有消费者实例被视为一个逻辑组，组内每个分区只会被分配给一个消费者实例，确保每条消息只被消费一次。

## 使用场景

- 微服务多副本部署，希望每条消息只被一个副本处理
- 需要水平扩展消费能力（增加消费者实例来提高吞吐）
- 需要分区内严格有序消费

## 基本原理

```text
Topic: 4 个分区
消费者组 "service-group": 2 个消费者

分配结果：
  consumer-1 → partition-0, partition-1
  consumer-2 → partition-2, partition-3
```

### 核心特性

| 特性 | 说明 |
|------|------|
| 分区独占 | 每个分区在组内只会分配给一个消费者 |
| 分区内有序 | 同一分区内的消息按 offset 顺序投递 |
| 消息不重复 | 消息只在一个分区，分区只给一个消费者 |
| Rebalance | 消费者增减时触发分区重新分配 |

### 分区数与消费者数的关系

| 场景 | 结果 |
|------|------|
| 分区数 = 消费者数 | 完美 1:1 分配 |
| 分区数 > 消费者数 | 部分消费者处理多个分区（推荐） |
| 分区数 < 消费者数 | 部分消费者空转，永远收不到消息 |

## 如果希望所有副本都收到同一条消息

- **方案一（不同 groupId）**：每个副本使用不同的 groupId，每个组独立消费全部消息
- **方案二（发布订阅）**：利用每个消费者组独立消费的特性实现广播

## 注意事项

- 分区数建议设为预期最大消费者数的 2~3 倍，便于未来扩容
- 消费者组新增或减少实例会触发 Rebalance，期间所有消费者暂停
- Kafka 只保证分区内有序，不保证全局有序
- 手动提交 + Rebalance 可能导致少量重复消费，业务侧需做幂等
- `auto.offset.reset=earliest` 新消费者组从未提交过 offset 时从最早消息开始消费

## 最佳实践

1. **分区数预留余量**：`预期最大消费者数 × 2~3`
2. **启用 Cooperative Rebalance**（Kafka 2.4+）：
   ```java
   props.put("partition.assignment.strategy",
       "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
   ```
3. **业务幂等**：消费逻辑具有幂等性，抵消可能的重复投递
4. **监控 Consumer Lag**：lag 持续增长说明消费能力不足

## 实际应用场景

- 订单处理服务 3 副本部署，topic 12 个分区，每条订单消息被其中 1 个副本处理
- 日志收集服务多个实例，使用同一 groupId 实现负载均衡消费
- 数据同步任务，多分区并行消费提高吞吐

## 参考资料

- [Kafka 官方文档 - Consumer Group](https://kafka.apache.org/documentation/#consumerapi)
- [Kafka 官方文档 - Rebalance Protocol](https://kafka.apache.org/37/javadoc/org/apache/kafka/clients/consumer/CooperativeStickyAssignor.html)

---

*最后更新：2026-05-25*