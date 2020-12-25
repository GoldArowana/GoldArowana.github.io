---
title: redis基础知识
date: 2021-04-04 16:06:02
summary: redis基础概念、实现原理
tags: 
    - redis
    - cache
    - database
categories:
    - redis
    - database
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co3.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co3.jpg
---
# 思维导图
![Redis思维导图](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/xmind/redis.xmind.svg)


## 高性能IO模型
请参考我的另一篇文章:[高性能网络模型](/high-performance-network-model/)


## 持久化
### AOF
#### AOF基本概念
相比于数据库的写前日志(Write Ahead Log, WAL), AOF是在执行完命令后, 写入的日志。

传统的数据库日志如redo log(重做日志), 记录的是修改后的数据, 而AOF里记录的是Redis服务器收到的每一条redis通信协议格式的命令。

为了避免开销, 写AOF之前并不会进行语法校验。所以先记日志再执行的话，可能会记录错误的指令。
而如果先执行命令, 成功后再记录AOF的话，就不会存在这样的问题。

#### AOF三种写回策略
可以选择appendfsync配置的三个可选值, 来选择对应的写回策略
- always, 每个写命令都立即将日志写回磁盘
- everysec, 每秒写回
- no, redis不主动写回磁盘, 只是先把日志写到AOF文件的内存缓冲区中, 由操作系统决定何时写回磁盘。

#### AOF重写机制
在重写时, 创建一个新的AOF文件, 扫描当前数据库中的所有键值对, 并写入到这个新的AOF文件中。
这样就可以减少AOF文件里key的无用历史记录。

重写是由后台线程bgrewriteof来完成的，被主线程fork出来的时候会有主线程的一份内存拷贝, 这里就包含了数据库的最新数据。
bgrewriteof子进程就可以在不影响主线程的情况下, 写入重写日志。
如果redis在这期间内有新的写入, 在写入AOF缓冲的时候, 还会往AOF重写缓冲里写一份数据。等重写日志完成后, 这些新的操作记录也会写入到新的AOF文件中。
之后就可以用新的AOF文件替代旧文件了。

> fork子进程时，子进程是会拷贝父进程的页表，即虚实映射关系，而不会拷贝物理内存。子进程复制了父进程页表，也能共享访问父进程的内存数据了，此时，类似于有了父进程的所有内存数据。我的描述不太严谨了，非常感谢指出！

> Kaito同学还提到了Huge page。这个特性大家在使用Redis也要注意。Huge page对提升TLB命中率比较友好，因为在相同的内存容量下，使用huge page可以减少页表项，TLB就可以缓存更多的页表项，能减少TLB miss的开销。

> 但是，这个机制对于Redis这种喜欢用fork的系统来说，的确不太友好，尤其是在Redis的写入请求比较多的情况下。因为fork后，父进程修改数据采用写时复制，复制的粒度为一个内存页。如果只是修改一个256B的数据，父进程需要读原来的内存页，然后再映射到新的物理地址写入。一读一写会造成读写放大。如果内存页越大（例如2MB的大页），那么读写放大也就越严重，对Redis性能造成影响。

> Huge page在实际使用Redis时是建议关掉的。

### RDB
可以使用save命令让主进程执行全量快照备份, 也可以使用bgsave命令fork出子进程进行快照备份。
在快照过程中如果有写操作, 那么会触发fork的copy on write机制。

### 混合快照
混合RDB和AOF

## 主从
可以通过replicaof命令(Redis5.0之前是slaveof), 设置主从关系。
比如在B实例上执行如下命令：
```shell
replicaof <A实例的IP地址> <A实例的Redis端口>
```
就可以让B作为A的从库, 并从A上复制数据。

### 主从间同步数据
#### 第一阶段: 建立连接, 协商同步
从实例会给主实例发送 psync {runID} {offset} 。
第一次建立主从关系的话, 从实例是不知道主实例的runID的, 所以runID部分会是'?'。
之前没有同步过数据, 所以offset部分是-1。

主实例收到建立的请求后, 会向从实例发送`FULLRESYNC` {runID} {offset}。
`FULLRESYNC` 表示第一次复制采用的全量复制, 也就是说, 主库会把当前所有的数据都复制给从库。

#### 第二阶段: 主库同步数据给从库
主库执行bgsave命令, 生成RDB文件传输给从库。
从库收到RDB文件后, 会先清空当前数据库, 在本地完成RDB加载。
在同步期间如果有新的数据产生, 主库会在内存中用专门的replication buffer 记录RDB文件生成后收到的所有写操作。

#### 第三阶段: 主库发送新写命令给从库
主库把replication buffer发送给从库。

#### 主从级联模式
`主-从`模式下, 如果从库数量很多, 那么会给主库带来压力。所以可以考虑`主-从-从`模式。

### 主从库间网络断了怎么办？
在Redis2.8之前, 在网络断了后, 主从会重新进行一次全量复制。

在Redis2.8开始, 网络断了后, 会先采用增量复制的方式继续同步。

repl_backlog_buffer是一个环形缓冲区, 主库会记录自己写到的位置, 从库则会记录自己已经读到的位置。
大小一般设置为：`(主库写入命令速度*操作大小 - 主从库间网络传输命令速度*操作大小) * 2`

### 哨兵机制
哨兵主要负责三个任务: 监控、选主、通知。
- 监控: 哨兵进程在运行时, 周期性地给所有的主从库发送PING命令, 检测他们是否仍然在线运行。
  如果再规定的时间内没有响应PING命令, 那么会标记为"下线状态"。如果主库下线的话, 就会开始自动切换主库的流程。
- 选主: 主库挂了后, 哨兵就需要从很多歌从库里, 选择一个新主库。选主的过程, 又分为 `筛选` 和 `打分`两个阶段。
    - 筛选。不在线的从库需要筛选出去。网络总是段连的从库需要筛选出去。(根据配置项down-after-milliseconds * 10判定。
      down-after-milliseconds是主从库锻炼的最大连接超时时间。如果断连次数超过10次, 那么说明这个从库的网络状况不好)
    - 打分。打分会经过多轮, 如果某一轮出现了得分最高的从库, 那么那么它就是主库了, 选主过程到此结束。否则继续下一轮选举。
        - 第一轮: 优先级最高的从库的最高分。
        - 第二轮: 和旧主库同步程度最接近的从库得最高分。
        - 第三轮: ID号小的从库得高分。
- 通知: 选好新的主库后, 哨兵会把新主库的链接信息发送给其他从库, 让他们执行eplicaof命令, 
  和新主库建立连接, 并进行数据复制。同时, 哨兵会把新主库的链接信息通知给客户端, 让他们把请求操作发送到新主库上。


在redis哨兵集群中, 有N/2 + 1 个哨兵认为主库下线了(这个值由redis管理员设定), 才能最终判定为"客观下线", 进行新主库的选举。

在redis主库中, 有一个名为"__sentinel__:hello"的`pub/sub`频道, 哨兵把自己的IP和端口发布到
"__sentinel__:hello"频道上, 其他哨兵订阅了这个频道后就可以获取到该哨兵的IP和端口, 以此来建立连接, 组成哨兵集群。

哨兵除了彼此之间建立连接形成集群意外, 还需要和从库建立连接, 对主从库都进行心跳判断。

哨兵通过给主库发送INFO命令来获取从库的信息。

本质上说, 哨兵就是一个运行在特定模式下的Redis实例, 只不过它并不服务其他请求操作, 只是完成监控、选主和通知的任务。
每个哨兵实例也提供pub/sub机制, 客户端可以从哨兵订阅消息。哨兵提供的消息大致有一下几个：

|  事件   | 相关频道  |
|  ----  | ----  |
| 实例进入"主观下线"装填  | +sdown |
| 实例退出"主观下线"装填  | -sdown |
| 实例进入"客观下线"装填  | +odown |
| 实例退出"客观下线"装填  | -0down |
| 哨兵发送SLAVEOF命令重新配置从库   | +slave-reconf-sent |
| 从库配置了新主库, 但尚未进行同步   | +slave-reconf-inprog |
| 从库配置了新主库, 且和新主库完成同步   | +slave-reconf-done |
| 新主库切换   | +switch-master  |

客户端可以订阅这些消息来获取相信的信息。比如订阅所有所实例的客观下线状态的事件：
```shell
SUBSCRIBE +odown
```

或者订阅所有的事件：
```shell
PSUBSCRIBE *
```

当哨兵把新主库选择出来后, 客户端就会看到下面的switch-master事件, 这样就会知道新主库的地址和端口了。
```shell
switch-master <master name> <oldip> <oldport> <newip> <newport>
```

leader哨兵才可以执行主从切换的过程。哨兵leader选举需要满足2个条件：
    1. 拿到半数以上的票
    1. 拿到的票数大于等于哨兵配置文件中的quorum值

## Redis Cluster

### 横向扩容？纵向扩容？
纵向机器成本高, 也会导致单实例存储的数据过多, RDB有压力

### Redis Cluster是什么
采用哈希槽(Hash Slot)来处理数据和实例之间的映射关系。
在Redis Cluster方案中, 一个切片集群共有16384个哈希槽，
这些哈希槽类似于数据分区, 每个键值对都会根据key映射到一个哈希槽中。

### 槽是怎么计算的
按照CRC16算法计算一个16bit的值, 然后再由这个16bit值对16384取模, 得到哈希槽的编号。

如果集群中有N个实例, 那么实例上的槽的个数为16384/N个。

也可以使用`cluster addslots`命令指定每个实例上的哈希槽个数。如果总共设置了5个槽, 那么槽的编号就是`CRC16(key)%5`。


### 客户端怎么知道哪个key在哪个实例上？
Redis会把自己的槽信息发送给和它相连的其他信息, 这样每个实例就有所有的槽信息了。

当客户端访问任意一个实例时, 便可以把槽的信息缓存在本地, 这样就知道对应的key应该发给哪个实例了。

### 在集群中, 实例和哈希槽的对应关系发生变化了怎么办？
Redis Cluster提供了一种重定向机制: 当客户端给一个Redis实例发送数据读写操作时, 如果这个实力上并没有对应的数据, 
这个实例就会给客户端返回MOVED重定向命令, 其中就带着对应实例的IP和端口
```shell
GET hello:key
(error) MOVED 13320 172.16.19.5:5379
```

如果slot正好再迁移中, 那么可能会返回ASK命令
```shell
GET hello:key
(error) ASK 13320 172.16.19.5:5379
```

### 为什么对key做CRC计算？直接key的slot的对应关系存下来可以吗？
不行。
1. 表的量会随着key增加而增大
2. 这个表只存一份的话会有单点问题, 如果部署多份的话会有一致性问题, 提高了系统复杂度。
3. CRC类似于hash, 可以使哈希结果更加分散



## 参考文章
1. 极客时间, 《Redis核心技术与实战》
2. 《Redis 5 设计与源码分析》
3. 《Redis设计与实现》
4. [解析Redis网络模型的源码](https://blog.csdn.net/qingyangcc123/article/details/107644885)
5. [Redis源码剖析（十一）--AOF持久化](https://www.cnblogs.com/lizhimin123/p/10197431.html)
6. [redis aof持久化源码分析](https://blog.csdn.net/bruce128/article/details/104579288)
7. [redis aof持久化的源码分析](https://blog.csdn.net/hangbo216/article/details/68925644)
8. [kqueue的用法](https://blog.csdn.net/Namcodream521/article/details/83032615)
9. [Redis 中的事件驱动模型](https://my.oschina.net/xilidou/blog/4381183)
10. [Redis时间事件源码解析](https://blog.csdn.net/qq_35181209/article/details/80353299)
11. [Redis事件](https://redisbook.readthedocs.io/en/latest/internal/ae.html)
12. [Redis 客户端与服务器连接流程实例](https://www.dazhuanlan.com/2019/12/16/5df6dec38a0fa/)
13. [深入浅出 Redis client/server交互流程](https://blog.csdn.net/fishmai/article/details/78515355)
14. [Linux -- fork() 写时拷贝（copy-on-write）](https://blog.csdn.net/qq_32095699/article/details/99689830)
15. [Redis的RDB持久化](https://vlambda.com/wz_5hpcZyGM8O2.html)
16. [Java NIO(7): Epoll版的Selector](https://zhuanlan.zhihu.com/p/27441342)
17. [石杉的架构笔记一杯茶的功夫，上手Redis持久化机制](https://juejin.cn/post/6916042914837561351)
18. https://jiekun.dev/posts/redis-tio-implementation/
19. https://jiekun.dev/posts/2020-03-14-redis-6-0-acl%E5%9F%BA%E4%BA%8Ebitmap%E5%AE%9E%E7%8E%B0/
20. https://jiekun.dev/posts/2020-01-31-redis%E5%93%A8%E5%85%B5%E6%95%85%E9%9A%9C%E8%BD%AC%E7%A7%BB/
21. https://blog.csdn.net/cywosp/article/details/8767327
22. https://blog.csdn.net/xinghuah/article/details/80487525
23. https://www.iocoder.cn/Redis/good-collection/
24. https://draveness.me/whys-the-design-redis-single-thread/
25. https://draveness.me/whys-the-design-redis-bgsave-fork/
26. https://redis.io/topics/distlock
27. [一个基于运气的数据结构, zset](https://segmentfault.com/a/1190000038398292)
28. https://coolshell.cn/articles/17416.html
29. https://zhenghe-md.github.io/blog/2020/03/08/Scaling-Memcache-at-Facebook-2013/
30. [300分钟吃透分布式缓存](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/300%E5%88%86%E9%92%9F%E5%90%83%E9%80%8F%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98-%E5%AE%8C)
31. [Redis 核心原理与实战](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Redis%20%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E6%88%98)
32. [深入学习Redis（2）：持久化](https://www.cnblogs.com/kismetv/p/9137897.html)
33. [深入学习Redis（3）：主从复制](https://www.cnblogs.com/kismetv/p/9236731.html)
34. [深入学习Redis（4）：哨兵](https://www.cnblogs.com/kismetv/p/9609938.html)
35. [深入学习Redis（5）：集群](https://www.cnblogs.com/kismetv/p/9853040.html)
36. [Redis 到底是单线程还是多线程？我要吊打面试官！](https://www.cnblogs.com/javastack/p/12848446.html)
37. [正式支持多线程！Redis 6.0与老版性能对比评测](https://blog.csdn.net/weixin_45583158/article/details/100143587)
38. [为什么Redis集群有16384个槽](https://zhuanlan.zhihu.com/p/80335611)
39. [你了解 Redis 的三种集群模式吗？](https://zhuanlan.zhihu.com/p/145186839)
40. [深入剖析Redis系列(二) - Redis哨兵模式与高可用集群](https://juejin.cn/post/6844903663362637832)
41. [redis 主从同步过程原理 以及 RDB/AOF/混合模式 持久化日志](https://blog.csdn.net/u013378306/article/details/109707279)
42. [P2P 网络核心技术：Gossip 协议](https://zhuanlan.zhihu.com/p/41228196)
43. [Redis源码分析](https://www.kancloud.cn/digest/redis-code)
