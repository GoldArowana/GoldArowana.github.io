---
title: jvm g1 gc
date: 2021-12-15 18:50:42
summary: 学习一下jvm的g1垃圾收集器
tags:
    - java
    - gc
categories:
    - java
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co61-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co61.jpg
---


## 参考文章
1. [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html)
2. <<JVM G1GC的的算法与实现>>
3. [面试官:你说你熟悉jvm?那你讲一下并发的可达性分析](https://segmentfault.com/a/1190000021820577)
4. [面试官问我G1回收器怎么知道你是什么时候的垃圾？](https://segmentfault.com/a/1190000021878102)
5. [[资料合集] RednaxelaFX写的文章/回答的导航帖（work in progress）](https://zhuanlan.zhihu.com/p/25042028)
6. [自动的内存管理系统实操手册——Java垃圾回收篇](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ%3D%3D&chksm=eaa89de5dddf14f39b4172adb6bf1f964252a244f9887286a6cae30aefe2b27833db9407d58f&idx=1&mid=2247518133&scene=21&sn=186ef93a6add47b01cac2f355211f6cd#wechat_redirect)