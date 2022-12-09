# MQ
## 为什么使用 MQ?
> 等价问题: MQ 有什么好处?  

*期望回答:*  
业务场景 + 技术挑战 + 使用 MQ 带来的好处  

*MQ 使用场景?*  
  - 解耦: 发布/订阅模型有效将业务逻辑解耦
  - 异步: 提高响应速度, 降低用户等待时间
  - 削峰: 应对大流量、高并发时间段，用 MQ 做缓冲

*MQ 优缺点?*  
- 优点: 解耦、异步、削峰
- 缺点: 
  - 降低系统可用性: MQ 崩溃系统随之无法正常工作
  - 提高系统复杂度: 需要解决重复消费, 维护数据顺序等问题
  - 一致性问题: 订阅数据的多个模块可能有一些出现异常没有执行成功, 数据出现一致性问题

*Kafka 和 RocketMQ 的区别?*
- Kafka 功能简单, 更多用于大数据领域的实时计算、日志采集场景
- RocketMQ 功能相较完善，分布式架构，扩展性好

*还了解哪些 MQ?*
- RabbitMQ, ActiveMQ  

针对项目:  
*为什么要使用 MQ?*  
使用 Kafka 能够实现订单生成和消费的解耦, 有利于后期更新和扩展.
同时系统的订单请求量时间分布不均衡, 主要集中在举办活动或者开学季, 需要使用 Kafka 进行流量削峰, 避发的大量请求使系统崩溃.  

*为什么不使用 RocketMQ 而是使用 Kafka?*  
因为当时听到最多的就是 Kafka, 加上前期知识了解不充足, 就选择先学习 Kafka. 后续准备学习 RocketMQ 的使用.  

## 如何保证 MQ 高可用?
> 既然 MQ 会带来可用性问题, 那么如何解决呢?

*回答策略:*  
用过哪个 MQ, 就说哪个 MQ 的解决方案  

*Kafka:*
  - 基础架构: Kafka 包含多个 Broker, 每个 Broker 是一个节点. 一个 Topic 包含多个 Partition, 每个 Partition 可以保存在不同的 Broker 上. 每个 Partition 有多个 Replica 副本保存在不同的节点上.
  - 天然的分布式消息队列, 每个 Topic 的数据存放在多个节点上.  
  - Replica 选举出 Leader, 只能读取 Leader 的数据(保证数据一致性). 
  - Leader 宕机应对: Follower 选举出新的 Leader, 但是可能出现丢失数据的情况.
  - 写数据: Leader 数据落盘时, Follower 从 Leader 处 pull 数据, 发送 ack 给 Leader, Leader 发送写成功消息(一种模式)
  - 消费数据: 只能读 Leader 数据, 只能读取高水位数据(最低的 Follower 的最新数据, 通过 pull 时 ack 传递)

## 如何保证 MQ 数据不被重复消费?
*问题本质:*  
如何保证消息队列幂等性?  

*Kafka:*  
  - offset: 保存 Consumer 上次消费到的数据 index.
    - 存在问题: Consumer 已经消费数据但是提交 offset 时出现异常, 导致重复消费.
  - 其实是考虑拿出来数据之后的处理如何保证幂等, **也就是业务逻辑如何做到幂等**.
    - 如果是数据库操作, 判断是否已经存在数据, 存在的话, 将 insert 改为 update
    - 如果是 Redis, 天然幂等
    - 复杂逻辑: 向消息中添加唯一 id, 使用唯一 id 保证幂等


## 如何保证消息可靠性传输?
> 问题本质: 消息不能多, 也不能少.  

消息多的问题在避免重复消费中已经解决了, 现在在于解决消息少的问题.  

*丢失消息的情况:*
- Consumer 丢失消息
  - 情况: Consumer 从 Kafka 消费消息, 自动提交了 offset, 但是在处理消息的过程中挂掉, Kafka 认为已经消费, 消息丢失.
  - 解决: 手动提交 offset
- Kafka 丢失消息
  - 情况: Kakfa 某个 broker 宕机, Leader 挂掉, 重新选举. 但是 Follower 都没有完全同步 Leader 的消息, 消息丢失.
  - 解决(保证 Leader 切换时, 消息不会丢失): 
    - 为每个 Partition 至少设置两个副本, `replication.factor > 1`.
    - `min.insync.replicas > 1`, 要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系, 保证 Leader 挂掉还有 Follower 可用.
    - Producer 设置 `acks = all`, 要求数据写入所有的 Replica 之后, 才认为写成功.
    - Producer 设置 `retries = MAX`, 要求如果写入失败, 就无限重试.
- Producer 丢失消息
  - 情况: 生产者发送消息, 但是 Kafka 接收失败, 部分数据未同步到 Kafka(其实还是 Kafka 数据同步的问题)
  - 解决: Producer 设置 `acks = all`
  
> *Kafka replica 的同步/选举机制:*  
> - 一个 Partition 的所有 Replica 都被称为 Async Replica(AR), 分为一个 Leader 和剩余的 Follower
> - 与 Leader 同步较好的 Follower 称为 In-Async Replica(ISR)
> - 与 Leader 同步较差的 Follower 成为 Out-Async Replica(OSR)
> - AR = ISR + OSR
> - 默认情况下, 如果 Leader 宕机, 只会从 AR 中选出新的 Follower 作为新的 Leader, 如果 AR 中没有, 就会报错, 停止工作.
> - 设置 `min.insync.replicas > 1` 会保证无论 Follower 同步情况如何, 都至少有一个 Follower 在 AR 中, 如果 Leader 宕机, 也能够进行正常的选举和工作, 只是可能会发生部分数据丢失问题.
> - 但是在 `acks = all` 的配置下, 保证数据在所有 Replica 落盘之后才认为写成功, 可以保证所有的 Replica 与 Leader 的数据在同一水位. 

## 如何保证 MQ 消息的顺序性?
> 为什么要有顺序?  
> 事务, 执行流程 ...

*Kafka 问题:*  
- 消息送入多个 topic, 无法保证顺序.  
  
*Consumer 问题:*  
- 多线程消费数据, 不能保证顺序性.  
  
*解决:*
- Kafka:
  - 指定消息唯一 id 为 key, 使得对应数据被发送到同一个 topic, 保证消息有序.
  - 一个 topic, 一个 partition, 一个 consumer (不推荐, 效率过低)
- 消费者:
  - N 个内存 Queue, 根据消息唯一 id 进行 hash, 相同 hash 的消息放入一个 Queue; N 个线程进行消费, 一个线程只消费一个 Queue 中的数据.

## 如何解决消息积压或消息失效问题?
> 本质上是消费端问题导致消息积压, 消费端可能消费停止或者缓慢, 然后有些消息可能就失效了.

*解决大量消息积压:*  
- 修复 Consumer, 使其能正常进行消费
- 对 Consumer 和 Queue 进行扩容, 快速消费完积压消息.
- 恢复原有部署
  
*解决 MQ 中消息失效:*  
- 数据设置了 TTL, 失效后被 MQ 删除, 数据丢失.
- 等待高峰期过, (从日志、备份等)批量重导丢失的数据, 重新写入 MQ
  
*解决 MQ 写满问题:*
- 放弃 MQ, 使用临时程序消费当前消息, 等待高峰期过按照消息失效的方式进行消息重导.
  
*优化:*
- 适度提高消费并发度(并发度过高反而速度慢)
- 使用批量消费模式, 可以大幅提高消费速度.
- 跳过非重要消息.
- 针对业务逻辑优化消费过程.

## 设计一个消息队列? (真的是魔鬼)
> 考察对消息队列原理的理解

*考虑角度:*  
- 伸缩性: 动态快速扩容(分布式系统)
- 数据落盘: 硬盘顺序写效率更高
- 可用性: 选举机制?
- 数据0丢失: 多重保障