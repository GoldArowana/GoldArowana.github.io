---
title: LSM-Tree
date: 2021-12-02 10:34:17
summary: SSTable结构 和 LSM树
tags:
  - database
  - data-structure
categories:
  - data-structure
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co97-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co97.jpg
---
## 适用场景
适合写多读少

1. 日志系统
1. 推荐系统
1. 海量数据存储
1. 数据分析


## 三大问题
### 读放大
读操作需要从新值到旧值依次读取, 在此过程中会涉及不止依次IO。尤其是范围查询时, 导致影响很明显 

### 空间放大
LSM通过append方式在末尾追击记录, 过期或者删除的数据不会立即清理, 所以会导致空间放大。

就是指存储引擎中的数据实际占用的磁盘空间比数据的真正大小偏多的情况。例如，数据的真正大小是10MB，但实际存储时耗掉了25MB空间，那么空间放大因子（space amplification factor）就是2.5

为什么会出现空间放大呢？很显然，LSM-based存储引擎中数据的增删改都不是in-place的，而是需要等待compaction执行到对应的key才算完。也就是说，一个key可能会同时对应多个value（删除标记算作特殊的value），而只有一个value是真正有效的，其余那些就算做空间放大。另外，在compaction过程中，原始数据在执行完成之前是不能删除的（防止出现意外无法恢复），所以同一份被compaction的数据最多可能膨胀成原来的两倍，这也算作空间放大的范畴。


### 写放大
为了减少读放大和空间放大, 会进行压缩合并。压缩合并的过程中对一条数据会涉及多次写, 导致写放大。

## Tiering
写优化, 空间放大。Cassandra。

## leveling
读优化, 写放大。RocksDB。

## Monkey: Optimal Navigable Key-Value Store
调整布隆过滤器的容错率

## Dostoevsky
### Lazy Leveling
写优化、点查优化。原理非常简单，也就是混合了 Tiered 以及 Leveled，在最大层使用 Leveled，而其它层使用 Tiered。

### Fluid LSM-Tree
Fluid 使用了一个可调解的方式，在最大层使用最多 Z runs，而其它层最多使用 K runs。

可以发现：
1. Z = 1， K = 1，就是 Leveled Compaction
2. Z = T - 1， K = T - 1，就是 Tiered Compaction
3. Z = 1， K = T - 1，就是 Lazy Leveling


## Wacky
适合大量写入、点读。缺点是范围读

## 实现

### lsm hash
#### bitcask(riak)
#### rosedb
#### nutsdb

### lsm array
#### moss

### lsm tree
#### pebble
#### leveldb
#### rocksdb
##### TTL实现


### HBase
##### TTL实现

### BigTable
### DynamoDB
### cassandra
##### TTL实现

### accumulo
### couchbase



## 参考文章
1. [论文阅读-The Log-Structured Merge-Tree (LSM-Tree)](https://blog.csdn.net/baichoufei90/article/details/84841289)
2. [Monkey: Optimal Navigable Key-Value Store (中文翻译)](https://www.dazhuanlan.com/mongolzhang/topics/1102894)
3. [Dostoevsky: 一种更好的平衡 LSM 空间和性能的方式](https://www.jianshu.com/p/8fb8f2458253)
4. [Log-structured merge-tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree)
5. [Scylla’s Compaction Strategies Series: Space Amplification in Size-Tiered Compaction](https://www.scylladb.com/2018/01/17/compaction-series-space-amplification/)
6. [LSM Tree-Based存储引擎的compaction策略（feat. RocksDB）](https://blog.csdn.net/nazeniwaresakini/article/details/104220140)
7. [深入 LevelDB 数据文件 SSTable 的结构](https://cloud.tencent.com/developer/article/1399365)
8. [深入探讨LSM Compaction机制](https://zhuanlan.zhihu.com/p/141186118)
9. [深入理解什么是LSM-Tree](https://www.toberoot.com/news/2020/12/19/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E4%BB%80%E4%B9%88%E6%98%AFLSM-Tree/)
10. [SSTable 原理](https://niceaz.com/2018/11/27/sstable)
11. [Log Structured Merge Trees](http://www.benstopford.com/2015/02/14/log-structured-merge-trees/)
12. [The Log-Structured Merge-Bush & the Wacky Continuum 中文](https://nan01ab.github.io/2019/06/LSM-Bush.html)
13. [REMIX: Efficient Range Query for LSM-trees](https://zhuanlan.zhihu.com/p/380013595)

https://blog.csdn.net/weixin_32496547/article/details/114098111

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.44.2782&rep=rep1&type=pdf

http://blog.fatedier.com/2016/06/15/learn-lsm-tree/

http://www.blogjava.net/DLevin/archive/2015/09/25/427481.html

https://izualzhy.cn/leveldb-sstable

https://www.cnblogs.com/fxjwind/archive/2012/08/14/2638371.html

https://zhuanlan.zhihu.com/p/103968892

https://zhuanlan.zhihu.com/p/103855686

[WiscKey —— SSD 介质下的 LSM-tree 优化](https://xiaozhuanlan.com/topic/7130689425)

https://zhuanlan.zhihu.com/p/181498475

https://rocksdb.org.cn/doc/Leveled-Compaction.html

https://rocksdb.org.cn/doc/Universal-Compaction.html


https://www.open-open.com/lib/view/open1424916275249.html

http://www.petermao.com/leveldb/leveldb-0-start.html

https://github.com/google/leveldb/blob/master/doc/impl.md

https://www.cnblogs.com/haippy/archive/2011/12/04/2276064.html

http://www.petermao.com/leveldb/leveldb-1-overview.html

http://www.benstopford.com/2015/02/14/log-structured-merge-trees/

http://www.benstopford.com/2015/02/14/log-structured-merge-trees/

https://github.com/facebook/rocksdb/wiki/Leveled-Compaction

https://zhenghe-md.github.io/blog/2020/02/26/Log-Structured-Merge-LSM-Tree-Usages-in-KV-Stores/

https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/

http://daslab.seas.harvard.edu/classes/cs265/project.html

http://www.youtube.com/watch?v=b6SI8VbcT4w

http://www.youtube.com/watch?v=ttebJcN5bgQ

https://www.microsoft.com/en-us/research/uploads/prod/2019/05/Scaling-Write-Intensive-Key-Value-Stores-slides.pdf

https://speakerdeck.com/mschoch/value-store-in-go?slide=3

https://izualzhy.cn/leveldb-table

https://izualzhy.cn/leveldb-sstable

https://scholar.harvard.edu/files/stratos/files/dostoevskykv.pdf

https://www.jianshu.com/p/8fb8f2458253

https://github.com/elithnever/paperreading/blob/master/lsm-tree-1.md

https://github.com/elithnever/paperreading/blob/master/lsm-tree-2.md

https://github.com/elithnever/paperreading/blob/master/lsm-tree-3.md

https://www.jianshu.com/p/3fb899684392


[LSM trees (Log Structured Merge Trees) - Detailed video, 印度口音, 而且没字幕](https://www.youtube.com/watch?v=oUNjDHYFES8)

[GopherCon 2017: Marty Schoch - Building a High-Performance Key/Value Store in Go](https://www.youtube.com/watch?v=ttebJcN5bgQ)

https://zhuanlan.zhihu.com/p/80523835

[Apache Cassandra SSTable 存储格式详解](https://www.iteblog.com/archives/2548.html)

https://blog.csdn.net/qq_21125183/article/details/103915326

https://blog.csdn.net/SweeNeil/article/details/86482781

https://www.cnblogs.com/qwj-sysu/p/5640256.html

https://daemondshu.github.io/2019/03/21/Programming/Data%20Structure/LevelDB_RocksDB/

https://www.shuzhiduo.com/A/amd0LMe1dg/

https://blog.csdn.net/zxpoiu/article/details/116190330

https://zhuanlan.zhihu.com/p/65557081

https://yetanotherdevblog.com/lsm/

https://www.cs.umb.edu/~poneil/lsmtree.pdf

https://tikv.org/deep-dive/key-value-engine/b-tree-vs-lsm/

https://www.usenix.org/conference/atc15/technical-session/presentation/wu

https://zhuanlan.zhihu.com/p/129355502

https://www.youtube.com/watch?v=auI8BTws-f8

https://www.youtube.com/watch?v=_cGtupq0Ts4https://www.youtube.com/watch?v=_cGtupq0Ts4

https://www.youtube.com/watch?v=rnZmdmlR-2M

https://www.bilibili.com/video/BV16X4y1A7TV

https://arxiv.org/pdf/1812.07527.pdf

1. [LSM-论文导读与Leveldb源码解读](https://hardcore.feishu.cn/docs/doccnKTUS5I0qkqYMg4mhfIVpOd#)
1. [Bitcask: A Log-Structured Hash Table for Fast Key/Value Data](https://riak.com/assets/bitcask-intro.pdf)
1. [Bitcask 学习笔记](https://old-panda.com/2020/08/23/bitcask/)

https://pingcap.com/blog-cn/titan-design-and-implementation

https://www.usenix.org/system/files/conference/fast16/fast16-papers-lu.pdf

1. [深入理解什么是LSM-Tree](https://cloud.tencent.com/developer/article/1441835)

https://www.jianshu.com/p/8fb8f2458253