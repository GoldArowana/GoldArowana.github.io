---
title: 2PC 和 3PC
date: 2021-08-18 11:05:59
summary: 2PC、3PC 了解一下？
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co182-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co182.jpg
---

## 2PC
2PC 方案实现起来简单，实际项目中使用比较少，主要因为以下问题：

1. 性能问题：所有参与者在事务提交阶段处于同步阻塞状态，占用系统资源，容易导致性能瓶颈。
2. 可靠性问题：如果协调者存在单点故障问题，如果协调者出现故障，参与者将一直处于锁定状态。
3. 数据一致性问题：在阶段 2 中，如果发生局部网络问题，一部分事务参与者收到了提交消息，另一部分事务参与者没收到提交消息，那么就导致了节点之间数据的不一致。

### 情况一：协调者挂了，参与者没挂
这种情况其实比较好解决，只要找一个协调者的替代者。当他成为新的协调者的时候，询问所有参与者的最后那条事务的执行情况，他就可以知道是应该做什么样的操作了。所以，这种情况不会导致数据不一致。

### 情况二：参与者挂了，协调者没挂

这种情况其实也比较好解决。如果参与者挂了。那么之后的事情有两种情况：

第一个是挂了就挂了，没有再恢复。那就挂了呗，反正不会导致数据一致性问题。

第二个是挂了之后又恢复了，这时如果他有未执行完的事务操作，直接取消掉，然后询问协调者目前我应该怎么做，协调者就会比对自己的事务执行记录和该参与者的事务执行记录，告诉他应该怎么做来保持数据的一致性。

### 情况三：参与者挂了，协调者也挂了

## 3PC
优点：相比二阶段提交，三阶段提交降低了阻塞范围，在等待超时后协调者或参与者会中断事务。避免了协调者单点问题，阶段 3 中协调者出现问题时，参与者会继续提交事务。

缺点：在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，会在等待超时之后，会继续进行事务的提交。 所以，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。


## TCC 和两阶段分布式事务处理的区别
当讨论 2PC 时，我们只专注于事务处理阶段，因而只讨论 prepare 和 commit，所以，可能很多人都忘了，使用 2PC 事务管理机制时也是有业务逻辑阶段的。正是因为业务逻辑的执行，发起了全局事务，这才有其后的事务处理阶段。实际上，使用 2PC 机制时，以提交为例

一个完整的事务生命周期是：begin -> 业务逻辑 -> prepare -> commit。


再看 TCC，也不外乎如此。我们要发起全局事务，同样也必须通过执行一段业务逻辑来实现。该业务逻辑一来通过执行触发 TCC 全局事务的创建；二来也需要执行部分数据写操作；此外，还要通过执行来向 TCC 全局事务注册自己，以便后续 TCC 全局事务 commit/rollback 时回调其相应的 confirm/cancel 业务逻辑。所以，使用 TCC 机制时，以提交为例

一个完整的事务生命周期是：begin -> 业务逻辑 (try 业务) -> commit (comfirm 业务)。

综上，我们可以从执行的阶段上将二者一一对应起来：
1. 2PC 机制的业务阶段 等价于 TCC 机制的 try 业务阶段；
2. 2PC 机制的提交阶段（prepare & commit） 等价于 TCC 机制的提交阶段（confirm）；
3. 2PC 机制的回滚阶段（rollback） 等价于 TCC 机制的回滚阶段（cancel）。

## 参考文章
1. [关于分布式事务，XA协议的学习笔记（整理转载）](https://www.cnblogs.com/monkeyblog/p/10449363.html)
2. [分布式事务中2PC与3PC的区别(转)](https://zhuanlan.zhihu.com/p/38177650)
3. [关于分布式事务、两阶段提交协议、三阶提交协议](http://www.hollischuang.com/archives/681#rd?sukey=3997c0719f1515205acb269da14295ad50b0186483fbd0a402a566f45b33525978b375ccc44dba3e85c4d645a320ba47)
4. [分布式事务（上）：除了 XA，还有哪些原子提交算法吗？](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/24%E8%AE%B2%E5%90%83%E9%80%8F%E5%88%86%E5%B8%83%E5%BC%8F%E6%95%B0%E6%8D%AE%E5%BA%93-%E5%AE%8C/18%20%20%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E9%99%A4%E4%BA%86%20XA%EF%BC%8C%E8%BF%98%E6%9C%89%E5%93%AA%E4%BA%9B%E5%8E%9F%E5%AD%90%E6%8F%90%E4%BA%A4%E7%AE%97%E6%B3%95%E5%90%97%EF%BC%9F.md)
https://zhuanlan.zhihu.com/p/343226202

https://zhuanlan.zhihu.com/p/343224188

1. [分布式事务(1)---2PC和3PC原理](https://www.cnblogs.com/qdhxhz/p/11167025.html)
2. [分布式事务](https://github.com/doocs/advanced-java/blob/main/docs/distributed-system/distributed-transaction.md)
3. [分布式事务（二）、刚性事务之 2PC、3PC](https://www.modb.pro/db/69275) 
4. [分布式事务（三）、柔性事务之 TCC、Saga、本地消息表、事务消息、最大努力通知](https://www.modb.pro/db/69276)
5. [正确理解二阶段提交（Two-Phase Commit）](https://blog.csdn.net/lengxiao1993/article/details/88290514)
6. [聊聊分布式数据库对2PC的优化](https://database.51cto.com/art/202101/640577.htm)
7. [TCC与两阶段分布式事务处理的区别](https://juejin.cn/post/6844903951477768205)
8. [分布式事务下的刚性事务和柔性事务](https://baijiahao.baidu.com/s?id=1668643758581970944&wfr=spider&for=pc)
9. [2PC & 3PC](https://zhenghe.gitbook.io/open-courses/mit-6.824/2pc-and-3pc)
1. [漫话分布式系统共识协议: 2PC/3PC篇](https://zhuanlan.zhihu.com/p/35298019)

https://www.youtube.com/watch?v=aDp99WDIM_4

https://www.youtube.com/watch?v=S4FnmSeRpAY

https://www.youtube.com/watch?v=aDp99WDIM_4

https://www.youtube.com/watch?v=VtUpBtAOizc

http://oceanbase.org.cn/?p=195

https://coolshell.cn/articles/10910.html

https://maimai.cn/article/detail?fid=1122653559&efid=jewbG8cL4Wikr2CJ3r_fAw

https://www.the-paper-trail.org/post/2008-11-29-consensus-protocols-three-phase-commit/

https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-two-phase-commit/