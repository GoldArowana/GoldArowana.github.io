---
title: kafka
date: 2021-11-02 13:54:45
summary: kafka原理学习
tags:
    - kafka
    - MQ
categories:
    - MQ
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co129-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co129.jpg
---

## 消息分区机制
- 轮循策略(未指定key的默认策略)
- 随机策略(老版本)
- 根据key计算hash(指定key时的默认策略)

## 消息压缩
> Producer和Broker都可以配置压缩算法。其实大部分情况下 Broker 从 Producer 端接收到消息后仅仅是原封不动地保存而不会对其进行任何修改.
> 
> 有两种例外情况就可能让 Broker 重新压缩消息:
> 
> 情况一：Broker 端指定了和 Producer 端不同的压缩算法。
> 
> 情况二：Broker 端发生了消息格式转换(Kafka集群中同时保存了多种版本的消息格式, 为了兼容老版本的格式，Broker 端会对新版本消息执行向老版本格式的转换。这个过程中会涉及消息的解压缩和重新压缩。)

## 消息丢失
1. Producer要使用带callback的函数, 这样可以处理失败。
1. 不开启自动提交, 而是手动提交位移
1. 设置 acks = all。acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
1. 设置 retries 为一个较大的值。这里的 retries 同样是 Producer 的参数，对应前面提到的 Producer 自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。
1. 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。
1. 设置 replication.factor >= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。
1. 设置 min.insync.replicas > 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
1. 确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。

## 位移主题
__consumer_offsets 叫位移主题, 是kafka的内部主题。


## 参考资料
1. 《极客时间-Kafka核心技术与实战》
1. 《极客时间-Kafka核心源码解读》