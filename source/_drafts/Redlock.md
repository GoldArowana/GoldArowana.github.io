---
title: Redlock
date: 2021-10-15 11:05:59
summary: Redis Distributed Lock
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co128-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co128.jpg
---

## 原理

> 客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为 10 秒，则超时时间应该在 5-50 毫秒之间。这样可以避免服务器端 Redis 已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个 Redis 实例。

> 客户端使用当前时间减去开始获取锁时间就得到获取锁使用的时间。当且仅当从大多数（这里是 3 个节点）的 Redis 节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。

## 优点
1. 相比于单机模式, 更加高可用
2. 相比于主从模式, 不需要在故障时手动将从节点调整为主节点
3. 哨兵模式在主节点挂掉时不需要手动调整节点, 如果Master数据没有同步到Slave, 会导致两个client同时占有锁
4. cluster模式下, 在Master挂掉时, 也有可能没有来得及同步数据给Slave, 导致同时占用两把锁。

## 缺点
1. 依赖本地时钟
2. 当redis实例挂掉后, 可能会丢失持久化数据, 使得另一个client抢锁成功
3. 当client占用锁超过租约期限后, 会有另一个client抢锁成功。这时会2个client同时占用锁。

## 参考文章
https://www.cnblogs.com/rgcLOVEyaya/p/RGC_LOVE_YAYA_1003days.html

https://zhenghe-md.github.io/blog/2020/03/22/Distributed-Locking/

https://redis.io/topics/distlock

https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

http://courses.csail.mit.edu/6.852/08/papers/CT96-JACM.pdf

http://antirez.com/news/101

https://studygolang.com/articles/3126

https://www.cnblogs.com/mkl34367803/p/14649897.html

1. [【求锤得锤的故事】Redis锁从面试连环炮聊到神仙打架。](https://segmentfault.com/a/1190000022024288)