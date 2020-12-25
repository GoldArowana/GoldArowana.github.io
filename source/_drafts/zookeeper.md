---
title: zookeeper
date: 2021-05-06 12:11:13
summary: zookeeper
tags:
    - zab
    - consistency
categories:
    - distribute
img:  https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co88.jpg
tinyImg:  https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co88.jpg
---

## 设计的zab算法
> Zab协议是为分布式协调服务Zookeeper专门设计的一种 支持崩溃恢复 的 原子广播协议 ，是Zookeeper保证数据一致性的核心算法。Zab借鉴了Paxos算法，但又不像Paxos那样，是一种通用的分布式一致性算法。它是特别为Zookeeper设计的支持崩溃恢复的原子广播协议。

### discovery阶段

### synchronization阶段

### broadcast阶段

### 崩溃恢复
#### leader选举
> 一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。
#### 数据恢复

### 缺点
需要全量发送history:
follower -> leader -> follower

## 参考文章
1. https://www.youtube.com/watch?v=pbmyrNjzdDk
2. [ZooKeeper源码分析与实战](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/ZooKeeper%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%8E%E5%AE%9E%E6%88%98-%E5%AE%8C)
3. [Zab协议：一致性协议](https://www.huaweicloud.com/articles/b696779df9cf3272db9695a83efae29f.html)
4. [ZAB和Paxos算法的联系与区别](https://www.huaweicloud.com/articles/ca89fb0a9b4ada8bd5784a5d5992fa6b.html)
5. [算法ZAB与Paxos
   ](https://www.jianshu.com/p/78cdf955ceca)