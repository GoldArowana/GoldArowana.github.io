---
title: kafka
date: 2021-6-02 13:54:45
summary: kafka原理学习
tags:
    - kafka
    - MQ
categories:
    - MQ
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co129-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co129.jpg
---

## 生产者(producer)
![生产者整体架构](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/producer-arch.jpg)

### 元数据获取
元数据记录了集群中有哪些主题、主题的分区、每个分区的leader副本配在哪个节点上、follower副本分配在哪些节点上, 哪些副本在AR、ISR集合中、集群中有哪些几点、控制器节点又是哪一个等等。

### 重要的生产者参数
1. acks
2. max.request.size
3. retries

## 消费者(consumer)
一个分区只会被分配给一个消费者的一个线程

![消费进度的保存和恢复](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-offset-save.jpg)
![消费者客户端的消费消息流程](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consume-flow.jpg)

左图: 一个消费者进程里开了3个线程。
右图: 三个消费者进程分别开了1个线程。
两种都是开了3线程消费, 只不过对zk来说看到的消费组成员列表是不同的。
![消费者进程和线程](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-threads-vs-procs.jpg)

### 消费者连接器
![](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-connect-text.jpg)
![ConsumerConnect连接器各组件协调完成消息的消费](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-connect.jpg)

### 消费者客户端的线程模型
使用队列作为消息的缓存
![消费者客户端线程模型](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-threads-model.jpg)

### 消费组重平衡
#### 重平衡的条件
1. 组成员数量发生变化。
1. 订阅主题数量发生变化。
1. 订阅主题的分区数发生变化。
![触发消费者连接器执行再平衡操作的两种方式](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-rebalance.jpg)
   
#### 重平衡的过程
![拉取线程再平衡中的关闭和更新](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-pull-thread-rebalance-flow.jpg)


#### 通知机制, 消费端着端的心跳线程(Heartbeat Thread)
重平衡过程是如何通知到其他消费者实例的？答案就是，靠消费者端的心跳线程（Heartbeat Thread）

重平衡的通知机制正是通过心跳线程来完成的。当协调者决定开启新一轮重平衡后，它会将“REBALANCE_IN_PROGRESS”封装进心跳请求的响应中，发还给消费者实例。当消费者实例发现心跳响应中包含了“REBALANCE_IN_PROGRESS”，就能立马知道重平衡又开始了，这就是重平衡的通知机制。

### 消费者消费消息
![消费者消费消息的主流程](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-main-flow.jpg)
![](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/temp2.jpg)
![拉取线程和消费线程分别更新分区信息的状态](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/consumer-pull-consume-state.jpg)


### 位移的管理
![](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/temp1.jpg)

#### 提交偏移量到ZK

#### 提交偏移量到内部位移主题
__consumer_offsets 叫位移主题, 是kafka的内部主题。该主题的默认分区数是50, 副本数是3.

#### 自动提交

#### 手动提交

## Coordinator(协调者)
专门为 Consumer Group 服务，负责为 Group 执行 Rebalance 以及提供位移管理和组成员管理等

Consumer 端应用程序在提交位移时，其实是向 Coordinator 所在的 Broker 提交位移。同样地，当 Consumer 应用启动时，也是向 Coordinator 所在的 Broker 发送各种请求，然后由 Coordinator 负责执行消费者组的注册、成员管理记录等元数据管理操作。

所有 Broker 在启动时，都会创建和开启相应的 Coordinator 组件。也就是说，所有 Broker 都有各自的 Coordinator 组件。那么，Consumer Group 如何确定为它服务的 Coordinator 在哪台 Broker 上呢？

确定由位移主题的哪个分区来保存该 Group 数据, 然后找出该分区 Leader 副本所在的 Broker，该 Broker 即为对应的 Coordinator。


### StickyAssignor(有粘性的分区分配策略)

## 消息代理(broker)

### 存储层

#### 数据文件
#### 索引文件
> Kafka 中的索引文件以稀疏索引（sparse index）的方式构造消息的索引，它并不保证每个消息在索引文件中都有对应的索引项。

#### zero-copy(零拷贝)
![零拷贝](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/zero-copy.jpg)


### 控制器(controller)

### 分区、副本
#### 消息分区机制
- 轮循策略(未指定key的默认策略)
- 随机策略(老版本)
- 根据key计算hash(指定key时的默认策略)

#### 消息压缩
> Producer和Broker都可以配置压缩算法。其实大部分情况下 Broker 从 Producer 端接收到消息后仅仅是原封不动地保存而不会对其进行任何修改.
>
> 有两种例外情况就可能让 Broker 重新压缩消息:
>
> 情况一：Broker 端指定了和 Producer 端不同的压缩算法。
>
> 情况二：Broker 端发生了消息格式转换(Kafka集群中同时保存了多种版本的消息格式, 为了兼容老版本的格式，Broker 端会对新版本消息执行向老版本格式的转换。这个过程中会涉及消息的解压缩和重新压缩。)


### 网络模型
#### Reactor
#### Kafka里的网络模型
![kafka网络模型](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/kafka/kafka-network-thread.jpg)



## 内部元数据
### zookeeper
ZK存储了Kafka的内部元数据, 还记录了消费组的成员列表, 分区的消费进度, 分区的所有者。节点、主题、分区、消费者、偏移量(offset)、所有权(ownership)

### kraft
2.8开始kafka去掉了zookeeper, 

## kafka stream

## 调优
### 消息丢失
1. Producer要使用带callback的函数, 这样可以处理失败。
1. 不开启自动提交, 而是手动提交位移
1. 设置 acks = all。acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
1. 设置 retries 为一个较大的值。这里的 retries 同样是 Producer 的参数，对应前面提到的 Producer 自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。
1. 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。
1. 设置 replication.factor >= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。
1. 设置 min.insync.replicas > 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
1. 确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。




## 参考资料
1. 《极客时间-Kafka核心技术与实战》
2. 《极客时间-Kafka核心源码解读》
3. 《深入理解Kafka：核心设计与实践原理》
4. 《Kafka技术内幕》
5. [Kafka分区分配策略分析——重点：StickyAssignor](https://blog.csdn.net/u4110122855/article/details/103616791)
6. [Kafka分区分配策略（2）——RoundRobinAssignor和StickyAssignor](https://blog.csdn.net/u013256816/article/details/81123625)
7. https://www.cnblogs.com/huxi2b/
8. https://www.confluent.io/blog/
9. https://cwiki.apache.org/confluence/display/KAFKA/Index
10. https://cwiki.apache.org/confluence/display/KAFKA/Kafka+papers+and+presentations
11. https://blog.csdn.net/yjh314/article/details/78863875
12. https://www.jianshu.com/p/afd5da6a92ab
13. https://blog.csdn.net/nazeniwaresakini/article/details/116085573
14. https://zhuanlan.zhihu.com/p/368600560
15. [Kafka: a Distributed Messaging System for Log Processing (2011)](https://zhenghe-md.github.io/blog/2020/03/15/Kafka-a-Distributed-Messaging-System-for-Log-Processing-2011/)
16. [拿来就能用！去哪儿网消息中间件 QMQ 详解 | 技术头条](https://blog.csdn.net/csdnnews/article/details/99256826)
17. [关于kafka消费者处理消息异常实验](http://blog.chinaunix.net/uid-26111972-id-5845593.html)
18. [kafka之消费超时死循环](https://www.jianshu.com/p/2effb654e1f4)
19. [消息幂等（去重）通用解决方案，RocketMQ](https://jaskey.github.io/blog/2020/06/08/rocketmq-message-dedup/)
20. [如何提高消息处理效率](https://support.huaweicloud.com/bestpractice-kafka/kafka-bp-190605002.html)

http://notes.stephenholiday.com/Kafka.pdf

https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e

