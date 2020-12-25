---
title: Percolator
date: 2021-05-10 11:05:59
summary: Percolator乐观事务实现
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co145-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co145.jpg
---

## 简介
> Percolator 是 Google 的上一代分布式事务解决方案，构建在 BigTable 之上，在 Google 内部用于网页索引更新的业务。原理比较简单，总体来说就是一个经过优化的 2PC 的实现，依赖一个单点的授时服务 TSO 来实现单调递增的事务编号生成，提供 SI 的隔离级别。

### 过程
Percolator的前提是本地事务的数据库支持多版本并发控制协议，也就是mvcc。



## 参考文章
1. [聊聊分布式数据库对2PC的优化](https://database.51cto.com/art/202101/640577.htm)
2. [Google Percolator 的事务模型](https://github.com/ngaut/builddatabase/blob/master/percolator/README.md)
3. [Percolator 2PC模型](https://blog.csdn.net/hnwyllmm/article/details/99988463)
4. [percolator：在线增量处理系统 中文翻译](https://karellincoln.github.io/2018/04/05/percolator-translate/)
5. [Percolator中的两阶段提交实现分析](https://blog.csdn.net/maray/article/details/6978958)
6. [Percolator 中的分布式事务](https://www.daimajiaoliu.com/daima/471ac65cb10040c)
7. 《拉钩教育-24讲吃透分布式数据库》

http://research.google.com/pubs/pub36726.html

https://www.cnblogs.com/foxmailed/p/3887430.html

https://www.jianshu.com/p/05194f4b29dd

https://blog.csdn.net/maray/article/details/6978958

http://mysql.taobao.org/monthly/2018/11/02/

https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Peng.pdf

https://karellincoln.github.io/2018/04/05/percolator-translate/

https://blog.csdn.net/linuxheik/article/details/77800659

https://pingcap.com/blog-cn/tidb-transaction-model