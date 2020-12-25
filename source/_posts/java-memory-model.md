---
title: java memory model(java内存模型)
date: 2019-09-10 18:50:42
summary: java内存模型
tags:
    - java
categories:
    - java
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co82.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co82.jpg
---
## 简介
在并发编程中, 我们需要处理两个关键问题: 
- 线程之间如何通信。通信是指线程之间交换信息。(由JMM控制)
- 线程之间如何同步。同步是指程序操作发生的相对顺序。(由并发工具支持, synchronized同步锁、CyclicBarrier等)

在命令式编程中, 线程之间的通信机制有两种: 
- 共享内存(Java)
- 消息传递(Go)

## 重排序

Java源代码到最终实际执行的指令序列, 会经理下面三种重排序：
1. 源代码 -> 编译器优化重排序
1. -> 指令级并行重排序
1. -> 内存系统重排序 -> 最终执行的指令序列

禁止编译器重排序: JMM的编译器重排序规则会禁止特定类型的编译器重排序
禁止处理器重排序: JMM通过插入内存屏障


## happens-before
JMM属于语言级的内存模型, 它确保在不同的编译器和处理器上 为程序员员提供一致的内存可见性(通过禁止特定类型的编译器重排序和处理器重排序)。

- 程序顺序规则: 一个线程中的每个操作, happens-before于该线程中的任意操作
- 监视器锁规则: 对一个监视器的解锁, happens-before于随后对这个监视器的加锁


## cpu写缓冲区
现代的cpu使用写缓冲区来临时保存向内存写入的数据。

### 写缓冲区的优点
1. cpu不必停顿下来等待向内存写入数据, 这种停顿等待会打断流水线的持续运行。
1. 通过以批处理的方式刷新写缓冲区, 以及合并写缓冲区对同一内存地址的多次写, 减少对内存总线的占用

### 写缓冲区带来的问题
处理器对内存的读/写操作的执行顺序, 不一定与内存实际发生的读/写操作顺序一致

### 缓存一致性协议
常见的缓存一致性协议有：MESI，MESIF（MESIF是缓存行的状态标识，M:Modified, E: Exclusive, S:Shared, I:Invalid, F: Forwad），通过标记缓存行的状态和处理器间的通讯来实现。

## volatile



## 参考文章
1. [happens-before](http://ifeve.com/tag/happens-before/)
2. [指令重排、内存屏障概念解析](https://www.cnblogs.com/duanxz/archive/2013/01/15/2861606.html)
3. [Intel文档](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html)
4. [cmpxchg指令](https://www.felixcloutier.com/x86/cmpxchg)
5. [x86寄存器](http://www.cs.virginia.edu/~evans/cs216/guides/x86.html)
6. [ZF标志](https://en.wikipedia.org/wiki/Zero_flag)
7. [The JSR-133 Cookbook for Compiler Writers](http://gee.cs.oswego.edu/dl/jmm/cookbook.html)
8. [Java并发编程常识](https://segmentfault.com/a/1190000039091865)
9. [[资料合集] RednaxelaFX写的文章/回答的导航帖（work in progress）](https://zhuanlan.zhihu.com/p/25042028)
10. [讲真，我发现这本书有个地方写错了! 溢出 or 逸出?](https://segmentfault.com/a/1190000020673082)
11. [聊聊JMM](https://albk.tech/%E8%81%8A%E8%81%8AJMM.html)
12. [面试官告诉你什么是JMM和常考面试题](https://mp.weixin.qq.com/s/_zmhLhEDgLggejUdF1c9gw)
