---
title: golang 调度器
date: 2021-07-20 13:35:03
summary: golang 调度器的实现原理
tags:
    - golang
    - coroutine
categories:
    - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co70.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co70.jpg
---

## 设计原理
### 基于协作的抢占式调度
> Go 语言会在分段栈的机制上实现抢占调度，利用编译器在分段栈上插入的函数，所有 Goroutine 在函数调用时都有机会进入运行时检查是否需要执行抢占。

## 数据结构


## 调度器启动

## 创建Goroutine

## 调度循环

## 为什么协程切换的代价比线程切换低?
> 协程切换非常简单，就是把当前协程的 CPU 寄存器状态保存起来，然后将需要切换进来的协程的 CPU 寄存器状态加载的 CPU 寄存器上就 ok 了。而且完全在用户态进行，一般来说一次协程上下文切换最多就是几十ns 这个量级。
> 
> 线程的调度只有拥有最高权限的内核空间才可以完成，所以线程的切换涉及到用户空间和内核空间的切换，也就是特权模式切换，然后需要操作系统调度模块完成线程调度（taskstruct），而且除了和协程相同基本的 CPU 上下文，还有线程私有的栈和寄存器等，说白了就是上下文比协程多一些，其实简单比较下 task_strcut 和 任何一个协程库的 coroutine 的 struct 结构体大小就能明显区分出来。
> 恶化cache命中率，提升sys cpu的执行占比，导致侵占用户态的cpu利用率。

## 进程的开销比线程大在了哪里?
创建成本:
> Linux 中创建一个进程自然会创建一个线程，也就是主线程。创建进程需要为进程划分出一块完整的内存空间，有大量的初始化操作，比如要把内存分段（堆栈、正文区等）。创建线程则简单得多，只需要确定 PC 指针和寄存器的值，并且给线程分配一个栈用于执行程序，同一个进程的多个线程间可以复用堆栈。因此，创建进程比创建线程慢，而且进程的内存开销更大。

切换成本:
> 进程切换涉及虚拟地址空间的切换而线程不会。
> 
> 由于每个进程都有自己的虚拟地址空间，那么显然每个进程都有自己的页表，那么当进程切换后页表也要进行切换，页表切换后TLB就失效了

## 参考文章
1. [刘丹冰Aceld-Golang的协程调度器原理及GMP设计思想？](https://www.kancloud.cn/aceld/golang/1958305)
2. [Tony Bai-图解Go运行时调度器](https://tonybai.com/2020/03/21/illustrated-tales-of-go-runtime-scheduler/)
3. [Golang 协程Goroutine到底是怎么回事？（一）](https://blog.csdn.net/qiya2007/article/details/105447358?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162753851016780255226838%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162753851016780255226838&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-105447358.pc_v2_rank_blog_default&utm_term=Goroutine&spm=1018.2226.3001.4450)
4. [Golang 协程Goroutine到底是怎么回事？（二）](https://blog.csdn.net/qiya2007/article/details/105447383?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162753851016780255226838%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162753851016780255226838&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-105447383.pc_v2_rank_blog_default&utm_term=Goroutine&spm=1018.2226.3001.4450)
5. [go夜读-golang 中 goroutine 的调度](https://talkgo.org/t/topic/31)
6. [Go 语言原本-并发调度](https://golang.design/under-the-hood/zh-cn/part2runtime/ch06sched/)
7. [draveness-调度器](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/)
8. [[典藏版] Golang 调度器 GMP 原理与调度全分析](https://learnku.com/articles/41728)
9. 《Golang 源码剖析 1.5.1 雨痕》
10. [为什么协程切换的代价比线程切换低?](https://www.zhihu.com/question/308641794)
11. [Netty中的Reactor模型详解](https://segmentfault.com/a/1190000021492983)
12. [进程/线程上下文切换会用掉你多少CPU？](https://zhuanlan.zhihu.com/p/79772089)
13. [《重学操作系统》进程和线程：进程的开销比线程大在了哪里？](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E9%87%8D%E5%AD%A6%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F-%E5%AE%8C/17%20%20%E8%BF%9B%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%EF%BC%9A%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%BC%80%E9%94%80%E6%AF%94%E7%BA%BF%E7%A8%8B%E5%A4%A7%E5%9C%A8%E4%BA%86%E5%93%AA%E9%87%8C%EF%BC%9F.md)
14. [为什么运行时程序要分成内核态和用户态](https://www.jianshu.com/p/16b83e7facb9)
15. [什么是ring0-ring3](https://blog.csdn.net/htxhtx123/article/details/104595175)
16. [怎样去理解Linux用户态和内核态？](https://zhuanlan.zhihu.com/p/69554144)
17. [Context Switch Definition](http://www.linfo.org/context_switch.html)
18. [Linux系统调用详解（实现机制分析）--linux内核剖析（六）](https://blog.csdn.net/gatieme/article/details/50779184)
19. [关于go语言协程调度的一个问题(具体请看问题描述)?](https://www.zhihu.com/question/32264167)
20. [深度解密Go语言之 scheduler](https://juejin.cn/post/6844903930497859591)
21. [Linux核心调度器之周期性调度器scheduler_tick--Linux进程的管理与调度(十八）](https://cloud.tencent.com/developer/article/1367908)
22. [Golang - 调度剖析【第一部分】](https://segmentfault.com/a/1190000016038785)
23. [Golang - 调度剖析【第二部分】](https://segmentfault.com/a/1190000016611742)
24. [Golang - 调度剖析【第三部分】](https://segmentfault.com/a/1190000017333717)
25. [linux线程调度策略](https://www.cnblogs.com/charlieroro/p/12133100.html)
26. [为什么Go服务容器化之后延迟变高](https://mp.weixin.qq.com/s/7bjefYPhI03t9HRny42zkA)
27. [线程比进程更快，吞吐更强，本文从切换方面介绍](https://blog.csdn.net/qq_34417408/article/details/110393655)

http://www.linfo.org/context_switch.html

https://note.youdao.com/ynoteshare1/index.html?id=18c9f8f5d9fa76f11dbee46c48954835&type=note

https://xargin.com/go-scheduler/

https://www.bilibili.com/video/BV1RK4y187k4/?spm_id_from=333.788.recommend_more_video.4

https://www.bilibili.com/video/BV1pb411v7nu?from=search&seid=2638724496956885629

https://www.bilibili.com/video/BV19r4y1w7Nx?p=17

https://www.bilibili.com/video/BV1MZ4y1V7SP/?spm_id_from=333.788.recommend_more_video.-1

https://www.bilibili.com/video/BV1Ky4y1r78H/?spm_id_from=333.788.recommend_more_video.1

https://www.bilibili.com/video/BV1zT4y1F7XF?from=search&seid=6090279844654181905

https://github.com/Zeb-D/my-review/blob/master/go/%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86--golang-runtime.md

https://github.com/Zeb-D/my-review/blob/master/go/%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E5%8D%8F%E7%A8%8B%E6%A0%88.md

https://www.luozhiyun.com/archives/448

https://www.luozhiyun.com/archives/485

https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit

https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/

https://tiancaiamao.gitbooks.io/go-internals/content/zh/05.0.html

https://zhuanlan.zhihu.com/p/216118842

https://segmentfault.com/a/1190000039052089

https://zhuanlan.zhihu.com/p/244054940

https://www.infoq.cn/article/r6wzs7bvq2er9kuelbqb

https://www.infoq.cn/article/NuvRPz1cPk9Cw0HP3bkY?utm_source=related_read&utm_medium=article

1. [深入解析Go-抢占式调度](https://tiancaiamao.gitbooks.io/go-internals/content/zh/05.5.html)

https://zhuanlan.zhihu.com/p/42057783

http://xiaorui.cc/archives/6535

https://juejin.cn/post/6844903846825705485

https://zboya.github.io/post/go_scheduler/

https://lessisbetter.site/2019/03/26/golang-scheduler-2-macro-view/

https://yizhi.ren/2019/06/03/goscheduler/

https://jingwei.link/2019/05/26/golang-routine-scheduler.html

https://blog.haohtml.com/archives/18352

http://austsxk.com/2020/08/18/Golang%E8%B0%83%E5%BA%A6%E5%99%A8GMP%E5%8E%9F%E7%90%86%E4%B8%8E%E8%B0%83%E5%BA%A6%E5%85%A8%E5%88%86%E6%9E%90/

https://qcrao.com/2019/09/06/dive-into-go-scheduler-source-code/

https://qcrao.com/2020/04/03/talk-about-g0/

https://medium.com/a-journey-with-go/go-how-does-a-goroutine-start-and-exit-2b3303890452

https://medium.com/a-journey-with-go/go-how-does-go-recycle-goroutines-f047a79ab352

https://medium.com/a-journey-with-go/go-goroutine-and-preemption-d6bc2aa2f4b7

https://medium.com/a-journey-with-go/go-work-stealing-in-go-scheduler-d439231be64d

https://www.tyx.pub/archives/86

https://www.pengrl.com/p/39569/

https://feiybox.com/2020/03/14/Golang-%E5%8D%8F%E7%A8%8B%E8%B0%83%E5%BA%A6%E5%8E%9F%E7%90%86/

http://odin.show/2020/04/25/golang-GPM

https://www.huweihuang.com/golang-notes/principle/go-scheduler.html

http://qinqiyao.com/2019/12/25/Golang%E8%B0%83%E5%BA%A6%E5%99%A8%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/

https://blog.crazytaxii.com/posts/scheduling_in_go_part2/

https://www.liuvv.com/p/c8d0853c.html

https://www.iminho.me/wiki/blog-21.html

https://mdnice.com/writing/98e27d97c9fd43b3aed92f36d7566794

https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit#