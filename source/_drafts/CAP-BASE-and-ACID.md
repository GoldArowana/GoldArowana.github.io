---
title: CAP/BASE/ACID
date: 2021-08-30 11:05:59
summary: 学习一下CAP、BASE、ACID等概念
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co166-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co166.jpg
---

## 原理
在理论计算机科学中，CAP定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点：

* 一致性（Consistency） （等同于所有节点访问同一份最新的数据副本）
* 可用性（Availability）（每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据）
* 分区容错性（Partition tolerance）（以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。）

根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项。理解CAP理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了C性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了A性质。除非两个节点可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。


## 参考文章
1. [【翻译】Brewer's CAP Theorem CAP定理](https://www.cnblogs.com/13yan/p/9243669.html)

http://duanple.com/?p=163


https://www.julianbrowne.com/article/brewers-cap-theorem

https://zh.wikipedia.org/wiki/CAP%E5%AE%9A%E7%90%86

https://www.infoq.cn/article/cap-twelve-years-later-how-the-rules-have-changed/

http://duanple.com/?p=143

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf

https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf
