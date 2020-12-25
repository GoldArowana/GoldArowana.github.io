---
title: go内存管理
date: 2021-04-1 18:50:39
summary: 学习一下go的内存分配和管理机制
tags:
    - golang
categories:
    - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co24.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co24.jpg
---

## 内存管理组件
### mspan
> runtime.mspan 是 Go 语言内存管理的基本单元

> 每个 runtime.mspan 都管理 npages 个大小为 8KB 的页

> runtime.spanClass 是 runtime.mspan 的跨度类，它决定了内存管理单元中存储的对象大小和个数

### 线程缓存(mcache)
> runtime.mcache 是 Go 语言中的线程缓存，它会与线程上的处理器一一绑定，主要用来缓存用户程序申请的微小对象。每一个线程缓存都持有 68 * 2 个 runtime.mspan，这些内存管理单元都存储在结构体的 alloc 字段中。其中68个scan, 另外68个noscan


### 中心缓存(mcentral)
> runtime.mcentral 是内存分配器的中心缓存，与线程缓存不同，访问中心缓存中的内存管理单元需要使用互斥锁

> 每个中心缓存都会管理某个跨度类的内存管理单元，它会同时持有两个 runtime.spanSet，分别存储包含空闲对象和不包含空闲对象的内存管理单元。


### 页堆(mheap)
> runtime.mheap 是内存分配的核心结构体，Go 语言程序会将其作为全局变量存储，而堆上初始化的所有对象都由该结构体统一管理，该结构体中包含两组非常重要的字段，其中一个是全局的中心缓存列表 central，另一个是管理堆区内存区域的 arenas 以及相关字段。

> 页堆中包含一个长度为 136 的 runtime.mcentral 数组，其中 68 个为跨度类需要 scan 的中心缓存，另外的 68 个是 noscan 的中心缓存



## 相关资料
1. [内存分配器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/)
2. [【Golang】这个内存对齐呀！？](https://www.bilibili.com/video/BV1Ja4y1i7AF/?spm_id_from=333.788.recommend_more_video.7)
3. [从源码讲解 golang 内存分配](https://studygolang.com/articles/22652?fr=sidebar)
4. [详解Go中内存分配源码实现](https://www.luozhiyun.com/archives/434)
5. [🚀 Visualizing memory management in Golang](https://deepu.tech/memory-management-in-golang/)
6. [A visual guide to Go Memory Allocator from scratch (Golang)](https://medium.com/@ankur_anand/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed)
7. [TCMalloc : Thread-Caching Malloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)
8. [Golang 内存管理](http://legendtkl.com/2017/04/02/golang-alloc/)
9. [带你领略Go源码的魅力----Go内存原理详解](https://zhuanlan.zhihu.com/p/93838586)
10. [The Go Memory Model](https://golang.org/ref/mem)
11. [Go: Memory Management and Allocation](https://medium.com/a-journey-with-go/go-memory-management-and-allocation-a7396d430f44)
12. [可视化Go内存管理](https://tonybai.com/2020/03/10/visualizing-memory-management-in-golang/)
13. [图解Go内存分配器](https://tonybai.com/2020/02/20/a-visual-guide-to-golang-memory-allocator-from-ground-up/)
14. [go 内存管理](https://qiankunli.github.io/2020/11/22/go_mm.html)
15. [Go runtime剖析系列（一）：内存管理](https://zhuanlan.zhihu.com/p/323915446)
16. [内存管理](https://tiancaiamao.gitbooks.io/go-internals/content/zh/06.0.html)
17. [Go 内存管理](https://github.com/LeoYang90/Golang-Internal-Notes/blob/master/Go%20%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.md)
18. [go夜读-图解 Go 之内存对齐](https://talkgo.org/t/topic/103)
19. [栈的运行速度比堆快？ 栈堆详解](https://bbs.huaweicloud.com/blogs/254749)

https://xie.infoq.cn/article/ee1d2416d884b229dfe57bbcc

https://juejin.cn/post/6844904005215207432

https://juejin.cn/post/6844903795739082760

https://cloud.tencent.com/developer/article/1771373

https://www.jianshu.com/p/1f0a0ec2d661

https://www.techug.com/post/manual-memory-management-in-go.html

https://golang.design/under-the-hood/zh-cn/part2runtime/ch07alloc/

https://wudaijun.com/2019/09/go-performance-optimization/

https://gfw.go101.org/article/memory-block.html

http://blog.newbmiao.com/2018/08/20/go-source-analysis-of-memory-alloc.html

https://ilifes.com/golang/memory-alloc/

https://www.diglog.com/story/1035817.html

https://gohalo.me/post/golang-concept-memory-management-module-introduce.html

http://www.djaigo.com/golang/golang-nei-cun-guan-li.html

https://omen.ltd/archives/12/

https://www.happyxhw.cn/memory/

http://www.zhangfuguan.top/2020/08/16/go%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/index.html

https://mp.weixin.qq.com/s/rydO2JK-r8JjG9v_Uy7gXg

https://andblog.cn/?p=2887

https://www.bookstack.cn/read/GoExpertProgramming/chapter04-4.1-memory_alloc.md

[Understanding Allocations: the Stack and the Heap - GopherCon SG 2019](https://www.youtube.com/watch?v=ZMZpH4yT7M0)
[GopherCon UK 2018: Andre Carvalho - Understanding Go's Memory Allocator](youtube.com/watch?v=3CR4UNMK_Is)
[The Go Memory Model: GoSF Meetup, 1/23/19](https://www.youtube.com/watch?v=NzhH0p32fMY)

https://tonybai.com/2020/03/10/visualizing-memory-management-in-golang/

https://tonybai.com/2020/02/20/a-visual-guide-to-golang-memory-allocator-from-ground-up/

https://github.com/golang/proposal/blob/master/design/12800-sweep-free-alloc.md

https://docs.google.com/document/d/1un-Jn47yByHL7I0aVIP_uVCMxjdM5mpelJhiKlIqxkE/edit#heading=h.bvezjdnoi4no

https://medium.com/a-journey-with-go/go-how-does-the-goroutine-stack-size-evolve-447fc02085e5

https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub