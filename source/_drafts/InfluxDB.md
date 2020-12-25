---
title: InfluxDB
date: 2021-05-17 11:05:59
summary: InfluxDB内部实现原理
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co146.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co146.jpg
---

## TSM tree
### 背景
1. 相比于底层的levelDB, 作为时序数据库需要更多的删除操作: "我们的用户需要一种自动管理数据保留的方法。这意味着我们需要大量的删除。"
2. 在最开始的时候， influxdb 采用的方案每个shard都是一个独立的数据库实例，底层都是一套独立的LevelDB存储引擎。 这时带来的问题是，LevelDB底层采用level compaction策略，每个存储引擎都会打开比较多的文件，随着shard的增多，最终进程打开的文件句柄会很快触及到上限。
3. 为了避免删除操作，我们将数据分割成我们称之为shard的数据. influxdb 采用的方案每个shard都是一个独立的数据库实例，底层都是一套独立的LevelDB存储引擎。 这时带来的问题是，LevelDB底层采用level compaction策略，每个存储引擎都会打开比较多的文件，随着shard的增多，最终进程打开的文件句柄会很快触及到上限。（在 InfluxDB 中按照数据的时间戳所在的范围,会去创建不同的 shard）
4. 由于遇到大量的客户反馈文件句柄过多的问题，InfluxDB在新版本的存储引擎选型中选择了BoltDB替换LevelDB。BoltDB底层数据结构是mmap B+树。 但由于B+ 树会产生大量的随机写。 所以写入性能较差。
5. 相比于BoltDB: "我们发现了写入吞吐量的一大问题。在数据库超过几GB之后，IOPS开始成为瓶颈"
6. 我们的计划是在Bolt面前写一个WAL，这样我们可以减少随机插入到keyspace的数量。相反，我们会缓冲彼此相邻的多个写入，然后一次flush它们 但是，这仅仅是为了延缓了这个问题。高IOPS仍然成为一个问题，对于任何在适度工作负荷的场景下，它都会很快出现
7. 之后Influxdb 最终决定仿照LSM 的思想自研TSM ，主要改进点是基于时序数据库的特性作出一些优化，包含Cache、WAL以及Data File等各个组件，也会有flush、compaction等这类数据操作。

### 解决问题
> TSM的设计目标一是解决LevelDB的文件句柄过多问题，二是解决BoltDB的写入性能问题


## 参考文章
1. [时间序列数据库调研之InfluxDB](http://blog.fatedier.com/2016/07/05/research-of-time-series-database-influxdb/)
2. [InfluxDB详解之TSM存储引擎解析（一）](http://blog.fatedier.com/2016/08/05/detailed-in-influxdb-tsm-storage-engine-one/)
3. [InfluxDB详解之TSM存储引擎解析（二）](http://blog.fatedier.com/2016/08/15/detailed-in-influxdb-tsm-storage-engine-two/)
4. [InfluxDB Storage Engine Internals | Metamarkets](https://www.youtube.com/watch?v=rtEalnKT25I)
5. [OpenTSDB 时序数据引擎介绍 【 Go 夜读 】](https://www.bilibili.com/video/BV1ht411873N?from=search&seid=10631955173242485046)
6. [InfluxDB详解之TSM存储引擎解析（一）](http://blog.fatedier.com/2016/08/05/detailed-in-influxdb-tsm-storage-engine-one/)
7. [InfluxDB详解之TSM存储引擎解析（二）](http://blog.fatedier.com/2016/08/15/detailed-in-influxdb-tsm-storage-engine-two/)
8. [influxdb 源码解析-tsdb](https://lrita.github.io/2017/06/12/influxdb-tsdb/)
9. [InfluxDB中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/)
10. [InfluxDB的存储引擎和TSM](https://jasper-zhang1.gitbooks.io/influxdb/content/Concepts/storage_engine.html)
11. https://liujiacai.net/blog/2021/01/21/thoughts-of-influxdb-iox/
12. http://hbasefly.com/2018/01/13/timeseries-database-4/
13. [InfluxDB详解之TSM存储引擎解析（一）](https://blog.fatedier.com/2016/08/05/detailed-in-influxdb-tsm-storage-engine-one/)
14. [TSM-tree](https://xiazemin.github.io/MyBlog/storage/2020/06/30/tsm.html)