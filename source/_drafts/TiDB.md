---
title: TiDB
date: 2021-05-02 11:05:59
summary: TiDB
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co144.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co144.jpg
---

## NewSQL
1. 强一致事务是否必须在数据库层解决？
2. 数据的增长速度是否不可预估的？
3. 扩容的频率是否已超出了自身运维能力？
4. 相比响应时间更看重吞吐？
5. 是否必须做到对应用完全透明？
6. 是否有熟悉NewSQL数据库的DBA团队？


优点：

1. 高度兼容MySQL: 大多数情况下，无需修改代码即可从 MySQL 轻松迁移至 TiDB，分库分表后的 MySQL 集群亦可通过 TiDB 工具进行实时迁移。
2. 水平弹性扩展: 通过简单地增加新节点即可实现 TiDB 的水平扩展，按需扩展吞吐或存储，轻松应对高并发、海量数据场景。
3. 分布式事务: TiDB 100% 支持标准的 ACID 事务。
4. 真正金融级高可用: 相比于传统主从 (M-S) 复制方案，基于 Raft 的多数派选举协议可以提供金融级的 100% 数据强一致性保证，且在不丢失大多数副本的前提下，可以实现故障的自动恢复 (auto-failover)，无需人工介入。
5. 一站式HTAP解决方案: TiDB 作为典型的 OLTP 行存数据库，同时兼具强大的 OLAP 性能，配合 TiSpark，可提供一站式 HTAP解决方案，一份存储同时处理OLTP & OLAP无需传统繁琐的 ETL 过程。
6. 云原生SQL数据库: TiDB 是为云而设计的数据库，同 Kubernetes深度耦合，支持公有云、私有云和混合云，使部署、配置和维护变得十分简单。

## TiDB Server
> SQL 层，对外暴露 MySQL 协议的连接 endpoint，负责接受客户端的连接，执行 SQL 解析和优化，最终生成分布式执行计划。TiDB 层本身是无状态的，实践中可以启动多个 TiDB 实例，通过负载均衡组件（如 LVS、HAProxy 或 F5）对外提供统一的接入地址，客户端的连接可以均匀地分摊在多个 TiDB 实例上以达到负载均衡的效果。TiDB Server 本身并不存储数据，只是解析 SQL，将实际的数据读取请求转发给底层的存储节点 TiKV（或 TiFlash）。

## PD (Placement Driver Server)
> 整个 TiDB 集群的元信息管理模块，负责存储每个 TiKV 节点实时的数据分布情况和集群的整体拓扑结构，提供 TiDB Dashboard 管控界面，并为分布式事务分配事务 ID。PD 不仅存储元信息，同时还会根据 TiKV 节点实时上报的数据分布状态，下发数据调度命令给具体的 TiKV 节点，可以说是整个集群的“大脑”。此外，PD 本身也是由至少 3 个节点构成，拥有高可用的能力。建议部署奇数个 PD 节点。


对以上的问题和场景进行分类和整理，可归为以下两类：

第一类：作为一个分布式高可用存储系统，必须满足的需求，包括几种：

1. 副本数量不能多也不能少
2. 副本需要根据拓扑结构分布在不同属性的机器上
3. 节点宕机或异常能够自动合理快速地进行容灾


第二类：作为一个良好的分布式系统，需要考虑的地方包括：
1. 维持整个集群的 Leader 分布均匀
2. 维持每个节点的储存容量均匀
3. 维持访问热点分布均匀
4. 控制负载均衡的速度，避免影响在线服务
5. 管理节点状态，包括手动上线/下线节点 

满足第一类需求后，整个系统将具备强大的容灾功能。满足第二类需求后，可以使得系统整体的资源利用率更高且合理，具备良好的扩展性。

为了满足这些需求，首先需要收集足够的信息，比如每个节点的状态、每个 Raft Group 的信息、业务访问操作的统计等；其次需要设置一些策略，PD 根据这些信息以及调度的策略，制定出尽量满足前面所述需求的调度计划；最后需要一些基本的操作，来完成调度计划。

### 信息收集
每个 TiKV 节点会定期向 PD 汇报节点的状态信息

每个 Raft Group 的 Leader 会定期向 PD 汇报 Region 的状态信息

### TSO

### 心跳

### Split/Merge

### 路由


## TiKV Server
> 负责存储数据，从外部看 TiKV 是一个分布式的提供事务的 Key-Value 存储引擎。存储数据的基本单位是 Region，每个 Region 负责存储一个 Key Range（从 StartKey 到 EndKey 的左闭右开区间）的数据，每个 TiKV 节点会负责多个 Region。TiKV 的 API 在 KV 键值对层面提供对分布式事务的原生支持，默认提供了 SI (Snapshot Isolation) 的隔离级别，这也是 TiDB 在 SQL 层面支持分布式事务的核心。TiDB 的 SQL 层做完 SQL 解析后，会将 SQL 的执行计划转换为对 TiKV API 的实际调用。所以，数据都存储在 TiKV 中。另外，TiKV 中的数据都会自动维护多副本（默认为三副本），天然支持高可用和自动故障转移。

## 事务

### ACID
1. Atomicity(原子性): 要么都成功, 要么都失败
1. Consistency(一致性): 任何事务都不破坏事务的完整性
1. Isolation(隔离性): 防止事务交叉执行, 而导致的数据不一致
1. Durability(持久性): 任何修改提交后都是持久的, 不会丢失。

### 隔离级别
1. Read uncommitted: 会有脏读问题
1. Read committed: 不可重复读
1. Repeatable read: 幻读问题。事务A select全表, 发现只有id为1和2的数据, 然后事务B插入id为3的数据后进行了提交, 当事务A也插入id为3的数据时会失败。


### Percolator
#### TSO (TimeStamp Oracle)
#### 2pc

## 参考文章
1. [TiDB 整体架构](https://docs.pingcap.com/zh/tidb/stable/tidb-architecture)
2. [TiDB 数据库的存储](https://docs.pingcap.com/zh/tidb/stable/tidb-storage)
3. [TiDB 数据库的计算](https://docs.pingcap.com/zh/tidb/stable/tidb-computing)
4. [TiDB 数据库的调度](https://docs.pingcap.com/zh/tidb/stable/tidb-scheduling)
5. [go夜读-TiDB 源码阅读之 Transaction](https://talkgo.org/t/topic/68)
6. [TiDB 乐观事务 + SRTT 算法及其并发瓶颈](https://zhenghe.gitbook.io/open-courses/cmu-15-445-645-database-systems/two-phase-locking)
7. [TiDB 新特性漫谈：悲观事务](https://pingcap.com/blog-cn/pessimistic-transaction-the-new-features-of-tidb)
8. [TiKV 事务模型概览，Google Spanner 开源实现](https://pingcap.com/blog-cn/tidb-transaction-model)
9. [TiDB 源码阅读之 Transaction 【 Go 夜读 】](https://www.youtube.com/watch?v=A46VE3aUTKo)
10. [分布式系统的时间](https://blog.csdn.net/chdhust/article/details/74839277)
11. [Percolator - 分布式事务的理解与分析](https://zhuanlan.zhihu.com/p/261115166)
12. [TiKV 功能介绍 - Placement Driver](https://pingcap.com/zh/blog/placement-driver)
13. [Building a Reliable Large-Scale Distributed Database - Principles and Practice](https://pingcap.com/zh/blog/talk-principles-practice)
14. [TiKV 源码解析系列 - multi-raft 设计与实现](https://pingcap.com/zh/blog/the-design-and-implementation-of-multi-raft)
15. [TiDB源码初探](https://yushuangqi.com/blog/2016/tidb-yuan-ma-chu-tan.html)

https://docs.pingcap.com/zh/tidb/stable/tikv-overview

https://zhuanlan.zhihu.com/p/24118962

http://research.google.com/pubs/pub36726.html

https://pingcap.com/blog-cn/percolator-and-txn/

https://www.jianshu.com/p/513fc11cbb1e

https://docs.jdcloud.com/cn/tidb-service/secondary-index

https://docs.pingcap.com/zh/tidb/stable/optimistic-transaction

https://docs.pingcap.com/zh/tidb/stable/pessimistic-transaction

https://zhenghe.gitbook.io/open-courses/cmu-15-445-645-database-systems/two-phase-locking

https://www.jdon.com/55452

https://github.com/ngaut/builddatabase

https://www.iocoder.cn/TiDB/good-collection/

https://blog.csdn.net/TiDB_PingCAP/article/details/99945822

