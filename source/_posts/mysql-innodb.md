---
title: mysql-innodb
date: 2021-02-18 12:19:28
summary: 学习mysql数据库InnoDB存储引擎
tags: 
    - mysql
    - database
    - innodb
categories:
    - database
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co121-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co121.jpg
---

## mysql体系结构
1. 连接池组件
2. 管理服务和工具组件
3. SQL接口组件
4. 查询分析器组件
5. 缓冲组件
6. 插件式存储引擎
7. 物理文件

![mysql体系结构](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/mysql-arch.jpg)

### InnoDB特点概述
1. 主要面向`OLTP`
1. 通过使用`MVCC`来获得高并发性, 提供一致性非锁定读, 并实现了`SQL`标准的4中隔离级别
1. 支持行锁
1. 使用`next-key locking`的策略来避免幻读(`phantom`)现象
1. 提供了插入缓冲(`insert buffer`)、二次写(`double write`)、自适应哈希索引(`adptive hash index`)、预读(`read ahead`)等高性能和高可用功能 

## InnoDB体系结构
InnoDB线程:
![InnoDB体系结构](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/innodb-arch.jpg)
InnoDB内存:
![InnoDB内存数据对象](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/innodb-mem.jpg)

### Master Thread

### IO Thread
主要负责IO请求的回调处理, 有4类`IO Thread`:
- `write`: 默认`innodb_write_io_threads = 4`, 即默认是4线程
- `read`: 默认`innodb_read_io_threads = 4`
- `insert buffer thread`
- `log IO thread`

### Purge Thread
从`InnoDB 1.1 版本`开始, `purge`操作可以独立到单独的线程中进行。用于回收`undo`页。

### Page Cleaner Thread
在`InnoDB 1.2版本`中引入的(之前在`Master Thread`中)。用于脏页的刷新操作。

### 缓冲池(Buffer Pool)
缓冲池的设计目的是为了协调CPU速度与磁盘速度的鸿沟。 

把磁盘上的数据加载到缓冲池，避免每次访问都进行磁盘IO，起到加速访问的作用。

#### 内部组成
1. Buffer Pool中默认的缓存页大小和在磁盘上默认的页大小是一样的，都是16KB
2. 为了管理Pool中的缓存页，Innodb为每一个缓存页创建了一些所谓的控制信息。--控制信息也是写在页上面的。
3. 控制信息包括该页所属的表空间编号、页号、缓存页在Buffer Pool中的地址、链表节点信息、一些锁信息以及LSN信息。
4. 因为每个缓存页对应的控制信息占用的内存大小是相同的，因此从buffer pool中分配一块内存专门记录控制信息--控制块
5. 控制块和缓存页是一一对应的，它们都被存放到 Buffer Pool 中，其中控制块被存放到 Buffer Pool 的前边，缓存页被存放到 Buffer Pool 后边
6. 控制块和缓存页之间是有碎片的--当然如果大小分配合理也有可能没有碎片。
7. 每个控制块大约占用缓存页大小的5%,innodb_buffer_pool_size设置的大小并不包含控制块的大小，也就是说InnoDB在为Buffer Pool向操作系统申请连续的内存空间时，这片连续的内存空间一般会比innodb_buffer_pool_size的值大5%左右

[时延参考数据](https://gist.github.com/jboner/2841832) :
```text
Latency Comparison Numbers (~2012)
----------------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy             3,000   ns        3 us
Send 1K bytes over 1 Gbps network       10,000   ns       10 us
Read 4K randomly from SSD*             150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from disk    20,000,000   ns   20,000 us   20 ms  80x memory, 20X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```

#### 缓冲池对读写操作的关系
- 读: 判断要读取的页是否在缓冲池中, 如果不在则会将页加载到缓冲池中, 然后再从缓冲池中读取。
- 写: 对于数据库中页的修改操作, 首先修改在缓冲池中的页, 然后再通过`CheckPoint`机制刷新回磁盘上。

#### 缓冲池中的数据页类型
- 索引页
- 数据页
- undo页
- 插入缓冲(`insert buffer`)
- 自适应哈希索引(`adaptive hash index`)
- InnoDB存储的锁信息
- 数据字典信息(`data dictionary`)

#### LRU List
InnoDB存储引擎中, 缓冲池中页的大小默认为16KB, 是通过使用LRU算法来进行管理的。

InnoDB存储引擎对传统的LRU算法做了`midpoint insertion strategy`优化, 不会直接插入到列表的头部, 而是会插入到`midpoint`位置。默认情况下该位置在LRU列表的5/8处(63%)。

`midpoint insertion strategy` 可以防止热点数据被移出LRU列表。比如全表扫描的时候, 很可能新的页并不会频繁使用, 显然这些页替换调LRU的热点数据是不明智的。

#### Free List
`LRU List`新增节点的时候, 会先从`Free list`申请, 如果`Free list`里没有可用的空闲页, 那么`LRU list`将淘汰末尾的页, 然后将该也空间分配给新的页。

#### Flush List
`LRU list`列表中的页被修改后, 这些脏页就会被维护到`Flush list`中, 数据库会根据`CheckPoint`机制将脏页刷新回磁盘。

### 重做日志缓冲(redo log buffer)
默认大小为`innodb_log_buffer_size = 838869`, 即 8MB。

通常情况下, 8MB足以满足大部分的应用。因为重做日志在下列三种情况会刷到外部磁盘的重做日志文件中:
- Master Thread每秒会将`redo log buffer`刷到 `redo log file`中
- 每个事物提交时会将`redo log buffer`刷到 `redo log file`中
- 当重做日志缓冲池剩余空间小于1/2时会将`redo log buffer`刷到 `redo log file`中

### 额外的内存池
对一些数据结构本身的内存进行分配时, 需要重额外的内存池进行申请。例如缓冲池中的帧缓冲(`frame buffer`)还有缓冲控制对象(`buffer control block`)等。

### Checkpoint
InnoDB存储引擎内部, 有两种`Checkpoint`, 分别是`Sharp Checkpoint`, `Fuzzy Checkpoint`。`Checkpoint`技术的目的是解决一下几个问题:
1. 缩短数据库的恢复时间: 当数据库发生宕机时, 数据库不需要重做所有的日志, 只需对`Checkpoint`之后的重做日志进行恢复, 这样就大大缩短了恢复的时间。
1. 缓冲池不够时, 将脏页刷新到磁盘: 当缓冲池不够用时, 根据LRU算法会淘汰掉一些页, 若淘汰的页为脏页, 那么需要强制执行`CheckPoint`, 将脏页刷回磁盘。
1. 重做日志不可用时, 刷新脏页: 重做日志的空间是有限的, 是循环使用的。当被覆盖的时候, 需要强制执行`Checkpoint`将脏页刷新到磁盘。
   
#### Sharp Checkpoint
`Sharp Checkpoint`会将所有的脏页刷新回磁盘。发生在数据库关闭的时候。

#### Fuzzy Checkpoint
InnoDB存储引擎内部使用`Fuzzy Checkpoint`进行页的刷新, 即只刷新一部分脏页。分为以下几种情况:
1. Master Thread Checkpoint: 在 `Master Thread` 的 `flush loop` 中, 每秒或每十秒触发。
1. FLUSH_LRU_LIST checkpoint: `Page Cleaner`线程会检查LRU列表中是否有足够的可用空闲页, 默认为`innodb_lru_scan_depth = 1024`。倘若如果没有1024个可用空闲页, 那么会将LRU列表尾端的页移除, 如果这些页中有脏页, 那么需要进行`Checkpoint`
1. Async/Sync Flush Checkpoint: 当重做日志文件不可用的时候, 会触发。主要是为了保证重做日志的循环使用。
1. Dirty Page too much Checkpoint: 当脏页数太多的时候, 会导致InnoDB存储引擎强制进行`Checkpoint`. 默认是当超过`innodb_max_dirty_pages_pct = 75`的时候, 即脏页数量占据75%的时候, 会强制进行`Checkpoint`

## InnoDB关键特性

### 插入缓冲(Insert Buffer)
目的: 为了提升辅助索引的插入性能

#### 插入缓冲的作用
辅助索引的插入是较为离散的, 为了避免更新辅助索引的时候要频繁地离散读取数据, 
所以InnoDB存储引擎设计了`Insert Buffer`, 对于非聚集索引的插入或更新操作, 不是每次都直接插入到索引页中, 
而是先判断插入的非聚集索引页是否在缓冲池中, 若存在, 则直接插入;
若不存在, 那么先放入到一个`Insert Buffer`对象中, 假装已经插入到叶子节点, 再以一定的频率合并(merge)到辅助索引叶子节点中。 

#### 插入缓冲的缺点
1. 如果数据库发生了宕机, 有大量的`Insert Buffer`没有合并到实际的非聚集索引中去, 那么此时恢复可能需要很长的时间, 极端情况下甚至要几个小时。
1. 在写密集的情况下, 插入缓冲会占用过多的缓冲池内存(innodb_buffer_pool), 默认最大可以占用到1/2的缓冲池内存。

#### 使用插入缓冲需要满足的条件
`Insert Buffer`的使用需要同时满足一下两个条件:
- 索引是辅助索引(secondary index)
- 索引不是唯一(unique)的

#### 唯一索引为什么不能使用插入缓冲
为什么不能是唯一的? 如果要校验唯一性, 那么就还是需要进行读取加载对应的数据页才能进行判断。

#### 变更缓冲(Change buffer)
InnoDB从1.0.x版本开始引入了`Change Buffer`, 可以看做是`Insert Buffer`的升级版, 支持`INSERT`、`DELETE`、`UPDATE`。

InnoDB1.2.x版本开始可以通过参数`innodb_change_buffer_max_size`来控制`Change Buffer`最大使用内存的数量, 默认值为25, 表示最多使用1/4的缓冲池内存空间。

#### Insert/Change Buffer的实现
`Insert/Change Buffer`的数据结构是一颗`B+树`, 且全局只有一颗`Insert/Change Buffer B+树`。

#### 什么时候合并(Merge) Insert Buffer
- 辅助索引页被读取到缓冲池时: 例如在执行SELECT查询时, 此时需要将辅助索引页读取到缓冲池中, 此时需要检查`Insert Buffer Bitmap`页, 确认该索引是否有记录存放于`Insert Buffer B+树`中。如果有, 则将`Insert Buffer B+树`中的记录插入到该辅助索引页中。
- `Insert Buffer Bitmap` 页追踪到该辅助索引页已无可用空间时: `Insert Buffer Bitmap`里会记录可用空间, 若检测到插入记录后不够1/32页的剩余空间, 则会强制进行一次合并操作。
- `Master Thread`: 每秒或每10秒进行一次`Merge Insert Buffer`的操作。会随机地选择`Insert Buffer B+树`中连续的几页, 进行merge。

### 两次写(double write)
目的: 提供数据页的可靠性

可能某个16KB的页只写了前4KB时, 数据库发生了宕机, 这种情况被称为部分写失效(`partial page write`)。

### 自适应哈希索引(Adaptive Hash Index, AHI)
### 异步IO
### 刷新邻接页

## Mysql数据库的文件

## InnoDB存储引擎表的文件

## InnoDB存储引擎表的逻辑存储及实现

## 索引

### 索引下推优化

## 锁
### MySQL表级锁
`MySQL`里面表级别的锁有两种:
1. 表锁: InnoDB支持行锁, 所以一般不使用`lock tables`命令来控制并发。
1. 元数据锁(meta data lock, MDL): 公平的读写锁。 不需要显示使用, 在访问一个表的时候会被自动加上。MDL的作用是保证读写的正确性, 防止查询的过程中表结构发生变更。

## 事务
### read view

## binlog

## redo log

## undo log

## 分布式事务

## 主从复制

### 主从复制的步骤
1. `master`把数据更改记录到`binlog`中
1. `slave`把主服务器的`binlog`复制到自己的中继日志(`relay log`)中
1. `slave`重做中继日志中的日志, 把更改应用到自己的数据库上, 以达到最终一致性。

### 主从复制工作原理
1. `slave`的I/O线程负责读取`master`的`binlog`, 并将其保存为`relay log`
1. `slave`的SQL线程负责执行中继日志

![mysql主从复制](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/mysql-slave-binlog.jpg)

## 分库分表

### 为什么要分库分表
1. 大量请求阻塞: 在高并发场景下，大量请求都需要操作数据库，导致连接数不够了，请求处于阻塞状态。
2. 存储出现问题: 业务量剧增，单库数据量越来越大，给存储造成巨大压力。

#### 分表
分表：是为了解决由于单张表数据量多大，而导致查询慢的问题。大致三、四千万行数据就得拆分，不过具体还是得看每一行的数据量大小，有些字段都很小的可能支持更多行数，有些字段大的可能一千万就顶不住了。

#### 分库
分库：是为了解决服务器资源受单机限制，顶不住高并发访问的问题，把请求分配到多台服务器上，降低服务器压力。

### 垂直拆分

### hash路由
大众点评订单

通过UserId后四位mod 32分到32个库中，同时再将UserId后四位Div 32 Mod 32将每个库分为32个表

按2^n拆分(类比HashMap里的2^n。对比一下一致性哈希。)

### 路由表
![查询切分-分库](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/sharding-hash-table.jpg)
将ID和库的Mapping关系记录在一个单独的库中。

优点：ID和库的Mapping算法可以随意更改。
缺点：引入额外的单点。

### 范围路由
比如按照时间区间或ID区间来切分。

优点：单表大小可控，天然水平扩展。不需要做数据迁移
缺点：有热点问题, 一段时间的数据会集中到一张表上。无法解决集中写入瓶颈的问题。

### range+hash
微信红包场景可用

![range+hash分库分表](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mysql/sharding-range-hash.jpg)

snowflake模式下要注意, 每毫秒第一个id要随机生成, 例如leaf, 否则hash分库会不均匀。

### 分库分表带来的复杂性
#### 跨库关联查询
在单库未拆分表之前，我们可以很方便使用 join 操作关联多张表查询数据，但是经过分库分表后两张表可能都不在一个数据库中，如何使用 join 呢？

有几种方案可以解决：
1. 字段冗余：把需要关联的字段放入主表中，避免 join 操作；
2. 数据抽象：通过ETL等将数据汇合聚集，生成新的表；
3. 全局表：比如一些基础表可以在每个数据库中都放一份；
4. 应用层组装：将基础数据查出来，通过应用程序计算组装；
#### 分布式事务
单数据库可以用本地事务搞定，使用多数据库就只能通过分布式事务解决了。
常用解决方案有：基于可靠消息（MQ）的解决方案、两阶段事务提交、柔性事务等。

#### 排序、分页、函数计算问题
在使用 SQL 时 order by， limit 等关键字需要特殊处理，一般来说采用分片的思路：

先在每个分片上执行相应的函数，然后将各个分片的结果集进行汇总和再次计算，最终得到结果。

#### 分布式 ID
如果使用 Mysql 数据库在单库单表可以使用 id 自增作为主键，分库分表了之后就不行了，会出现id 重复。
常用的分布式 ID 解决方案有：
1. UUID
2. 基于数据库自增单独维护一张 ID表
3. 号段模式
4. Redis
5. 雪花算法（Snowflake）
6. 百度uid-generator
7. 美团Leaf
8. 滴滴Tinyid

#### 多数据源
分库分表之后可能会面临从多个数据库或多个子表中获取数据，一般的解决思路有：客户端适配和代理层适配。
业界常用的中间件有：
1. shardingsphere（前身 sharding-jdbc）
2. Mycat

## 参考文章
1. [cmu讲lock/latch 例子比较详细](https://www.youtube.com/watch?v=x5tqzyf0zrk&list=PLSE8ODhjZXjbohkNBWQs_otTrBTrjyohi&index=9)
2. [数据库事务原子性、一致性是怎样实现的？](https://www.zhihu.com/question/30272728)
3. 《Mysql技术内幕 InnoDB存储引擎》
4. [InnoDB关键特性之double write](https://www.cnblogs.com/geaozhang/p/7241744.html)
5. [InnoDB 重要特性 Double Write 实现原理](https://juejin.cn/post/6890830002162499598)
6. [mysql 为何需要Double Write？有redo log还不够吗？](https://blog.csdn.net/synchronizing/article/details/109025093)
7. [MySQL：数据库宕机以后恢复的过程？如何保证事务的ACID特性？](https://www.zhihu.com/question/59729819/answer/284180647)
8. [On learning InnoDB: A journey to the core](https://blog.jcole.us/innodb/)
9. [innodb_support_xa的作用](http://blog.itpub.net/15498/viewspace-2153760/)
10. [MySQL学习（二）索引与锁](https://www.cnblogs.com/AlmostWasteTime/p/10330151.html)
11. [MySQL 8.0 Reference Manual-InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
12. [Innodb行锁源码学习(一)](https://www.cnblogs.com/cchust/p/4255499.html)
13. [MySQL Innodb行锁剖析](https://zhuanlan.zhihu.com/p/139489272)
14. [MySQL 加锁处理分析](https://github.com/hedengcheng/tech/blob/master/database/MySQL/MySQL%20%E5%8A%A0%E9%94%81%E5%A4%84%E7%90%86%E5%88%86%E6%9E%90.pdf)
15. [分库分表？如何做到永不迁移数据和避免热点？](https://cloud.tencent.com/developer/article/1441250)
16. [MySQL · 源码分析 · InnoDB的read view，回滚段和purge过程简介](http://mysql.taobao.org/monthly/2018/03/01/)
17. [MySQL总结--MVCC（read view和undo log）](https://blog.csdn.net/huangzhilin2015/article/details/115195777)
18. [从ReadView深入理解MySql MVCC原理](https://blog.csdn.net/qq_42651904/article/details/110622818)
19. [MVCC多版本并发控制](https://www.jianshu.com/p/8845ddca3b23)
20. [mysql Innodb_buffer_pool的原理](https://blog.csdn.net/h2604396739/article/details/100280126)
21. [MySQL幻读](https://www.jianshu.com/p/c53c8ab650b5)
22. [InnoDB MVCC何时创建read view](https://mp.weixin.qq.com/s/850S0HZ5SlwSFuJK9BLgrw)
23. [美团DB数据同步到数据仓库的架构与实践](https://tech.meituan.com/2018/12/06/binlog-dw.html)
24. [面试题：我们为什么要分库分表？](https://www.jianshu.com/p/a053abfd52c4)
25. [数据库分库分表事务解决方案](https://www.cnblogs.com/lizo/p/8035036.html)
26. [MTDDL——美团点评分布式数据访问层中间件](https://tech.meituan.com/2016/12/19/mtddl.html)
27. [大众点评订单系统分库分表实践](https://zhuanlan.zhihu.com/p/24036067)
28. [mysql分表的3种方法](http://blog.51yip.com/mysql/949.html)
29. [MySQL分库分表方案](https://zhuanlan.zhihu.com/p/84224499)
30. [在面试时被问到，为什么MySQL数据库数据量大了要进行分库分表？](https://www.zhihu.com/question/459955079)
31. [数据库分库分表解决方案汇总](https://segmentfault.com/a/1190000023914691)
32. [MYSQL单表数据达2000万性能严重下降，为什么？](https://zhuanlan.zhihu.com/p/355302417)
33. [一次分表踩坑实践的探讨](https://crossoverjie.top/2019/04/16/framework-design/sharding-db/)
34. [分表后需要注意的二三事](https://crossoverjie.top/2019/06/13/framework-design/sharding-db-02/)
35. [一次难得的分库分表实践](https://crossoverjie.top/2019/07/24/framework-design/sharding-db-03/)
36. [数据库分库分表基础和实践](https://www.infoq.cn/article/zmlcbpihothwjeqmzd4i)
37. [数据库分库分表思路 （1）（数据库分区、分表、分库、分片）](https://www.jianshu.com/p/7d8e2c02b07b)
38. [innodb源码分析](https://www.kancloud.cn/digest/innodb-zerok)
39. [Mysql数据库常用分库和分表方式](https://www.kancloud.cn/digest/mysqlsummary/132848)
40. [MySQL索引前世今生](https://mp.weixin.qq.com/s/1ZWOLPV4fCqi2EebU_C9YA)
41. [MySQL 2PC & Group Commit](https://segmentfault.com/a/1190000014810628)
42. [深入学习MySQL事务：ACID特性的实现原理](https://www.cnblogs.com/kismetv/p/10331633.html)
43. [MySQL · 引擎特性 · InnoDB redo log漫游](http://mysql.taobao.org/monthly/2015/05/01/)
44. [MySQL · 引擎特性 · InnoDB undo log 漫游](http://mysql.taobao.org/monthly/2015/04/01/)
45. [MySQL · 引擎特性 · InnoDB 崩溃恢复过程](http://mysql.taobao.org/monthly/2015/06/01/)


https://zhuanlan.zhihu.com/p/341317422

https://zhuanlan.zhihu.com/p/343226202

http://kernelmaker.github.io/InnoDB_redo_log

http://mysql.taobao.org/monthly/

https://toutiao.io/posts/2cvy58/preview

https://tech.meituan.com/2016/11/18/dianping-order-db-sharding.html