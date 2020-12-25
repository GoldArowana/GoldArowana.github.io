---
title: java synchronized 实现原理
date: 2021-11-27 18:52:02
summary: java的synchronized锁机制
tags:
    - java
    - concurrent
categories:
    - java
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co60-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co60.jpg
---


## 参考文章
1. [线程安全(中)--彻底搞懂synchronized(从偏向锁到重量级锁)](https://www.cnblogs.com/kubidemanong/p/9520071.html)
2. [synchronized锁升级优化](https://zhuanlan.zhihu.com/p/92808298)
3. [Synchronization](https://wiki.openjdk.java.net/display/HotSpot/Synchronization)
4. [深度分析：锁升级过程和锁状态，看完这篇你就懂了！](https://segmentfault.com/a/1190000022904663)