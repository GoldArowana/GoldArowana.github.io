---
title: golang 垃圾回收
date: 2021-04-07 18:50:57
summary: golang的垃圾回收器的实现原理
tags:
    - golang
    - gc
categories:
    - golang
    - gc
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co25.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co25.jpg
---

## GC roots
根对象在垃圾回收的术语中又叫做根集合，它是垃圾回收器在标记过程时最先检查的对象，包括：

全局变量：程序在编译期就能确定的那些存在于程序整个生命周期的变量。
执行栈：每个 goroutine 都包含自己的执行栈，这些执行栈上包含栈上的变量及指向分配的堆内存区块的指针。
寄存器：寄存器的值可能表示一个指针，参与计算的这些指针可能指向某些赋值器分配的堆内存区块。

## 三色标记法
* 白色对象 — 潜在的垃圾，其内存可能会被垃圾收集器回收；
* 黑色对象 — 活跃的对象，包括不存在任何引用外部指针的对象以及从根对象可达的对象；
* 灰色对象 — 活跃的对象，因为存在指向白色对象的外部指针，垃圾收集器会扫描这些对象的子对象；

### 三色不变性
> 想要在并发或者增量的标记算法中保证正确性，我们需要达成以下两种三色不变性（Tri-color invariant）中的一种：

* 强三色不变性 — 黑色对象不会指向白色对象，只会指向灰色对象或者黑色对象；
* 弱三色不变性 — 黑色对象指向的白色对象必须包含一条从灰色对象经由多个白色对象的可达路径

### 插入写屏障
> 插入写屏障会将有存活可能的对象都标记成灰色以满足强三色不变性。

> 将指针指向的新值置为灰色


### 删除写屏障
> 在老对象的引用被删除时，将白色的老对象涂成灰色，这样删除写屏障就可以保证弱三色不变性，老对象引用的下游对象一定可以被灰色对象引用。


### 混合写屏障

## 参考文章
1. [Go语言设计与实现-垃圾收集器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)
2. [[典藏版]Golang三色标记、混合写屏障GC模式图文全分析](https://segmentfault.com/a/1190000022030353)
3. [golang设计-垃圾回收](https://golang.design/under-the-hood/zh-cn/part2runtime/ch08gc/)
4. [legendtkl-Golang 垃圾回收剖析](http://legendtkl.com/2017/04/28/golang-gc/)
5. [Golang中GC回收机制三色标记与混合写屏障](https://www.bilibili.com/video/BV1wz4y1y7Kd)
6. [Golang三色标记+混合写屏障GC模式全分析](https://www.kancloud.cn/aceld/golang/1958308)
7. [golang 垃圾回收（四）删除写屏障](https://blog.csdn.net/qiya2007/article/details/107291523?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162753862316780366532920%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162753862316780366532920&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-3-107291523.pc_v2_rank_blog_default&utm_term=golang+%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6&spm=1018.2226.3001.4450)
8. [golang 垃圾回收（五）深入剖析混合写屏障](https://blog.csdn.net/qiya2007/article/details/107622603)
9. [Go 语言原本-写屏障技术](https://golang.design/under-the-hood/zh-cn/part2runtime/ch08gc/barrier/)
10. [深度剖析 Golang 的 GC 扫描对象实现](https://blog.csdn.net/qiya2007/article/details/107738258?spm=1001.2014.3001.5501)
11. [自动的内存管理系统实操手册——Golang垃圾回收篇](https://blog.csdn.net/QcloudCommunity/article/details/119397102)

https://making.pusher.com/golangs-real-time-gc-in-theory-and-practice/

https://blog.csdn.net/weixin_34248849/article/details/88987666

http://www.52codes.net/develop/shell/56961.html

https://www.cnblogs.com/badcw/p/14192895.html

https://weibo.com/ttarticle/p/show?id=2309404620360724382113#_loginLayer_1623600085589

https://weibo.com/ttarticle/p/show?id=2309404620723120373979

http://www.360doc.com/content/21/0416/22/15690396_972688492.shtml

https://www.cnblogs.com/saryli/p/10105393.html

https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md

https://talks.golang.org/2015/go-gc.pdf

https://www.luozhiyun.com/archives/475

https://www.oschina.net/translate/go-gc-solving-the-latency-problem-in-go-1-5?comments&p=1

https://www.jianshu.com/p/bfc3c65c05d1?utm_source=wechat_session

https://www.bilibili.com/video/BV1Ui4y1F7n3/?spm_id_from=333.788.recommend_more_video.0

https://xie.infoq.cn/article/67cfd494e6e10cd0b40de95ab

https://zhuanlan.zhihu.com/p/297177002

https://lessisbetter.site/2019/10/20/go-gc-1-history-and-priciple/

https://learnku.com/docs/go-blog/ismmkeynote/6499

https://tiancaiamao.gitbooks.io/go-internals/content/zh/06.2.html

https://www.bookstack.cn/read/GoExpertProgramming/chapter04-4.2-garbage_collection.md

https://docs.kilvn.com/go-internals/06.2.html

[Garbage Collection Semantics - GopherCon SG 2019](https://www.youtube.com/watch?v=q4HoWwdZUHs)

https://medium.com/a-journey-with-go/go-memory-management-and-memory-sweep-cc71b484de05

https://medium.com/a-journey-with-go/go-how-does-the-garbage-collector-mark-the-memory-72cfc12c6976

https://medium.com/a-journey-with-go/go-how-does-the-garbage-collector-watch-your-application-dbef99be2c35

https://medium.com/a-journey-with-go/go-keeping-a-variable-alive-c28e3633673a

https://www.tyx.pub/archives/148

https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)

https://talks.golang.org/2015/go-gc.pdf

https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md

https://github.com/golang/proposal/blob/master/design/11970-decentralized-gc.md

https://github.com/golang/proposal/blob/master/design/44167-gc-pacer-redesign.md

https://github.com/golang/proposal/blob/master/design/12800-sweep-free-alloc.md