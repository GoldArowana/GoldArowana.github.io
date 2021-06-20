---
title: jmm(java 内存管理)
date: 2021-06-08 18:50:42
summary:
tags:
    - java
categories:
    - java
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co82.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co82.jpg
---
## 简介
在命令式编程中, 线程之间的通信机制有两种: 共享内存和消息传递。Java的并发采用的是共享内存模型。

Java线程之间的通信由JMM控制, JMM决定一个线程对共享变量的写入何时对另一个线程可见。

Java源代码到最终实际执行的指令序列, 会经理下面三种重排序：
1. 源代码 -> 编译器优化重排序
1. -> 指令级并行重排序
1. -> 内存系统重排序 -> 最终执行的指令序列

禁止编译器重排序: JMM的编译器重排序规则会禁止特定类型的编译器重排序
禁止处理器重排序: JMM通过插入内存屏障

JMM属于语言级的内存模型, 它确保在不同的编译器和处理器上 为程序员员提供一致的内存可见性(通过禁止特定类型的编译器重排序和处理器重排序)。


## cpu写缓冲区
现代的cpu使用写缓冲区来临时保存向内存写入的数据。

### 写缓冲区的优点
1. cpu不必停顿下来等待向内存写入数据, 这种停顿等待会打断流水线的持续运行。
1. 通过以批处理的方式刷新写缓冲区, 以及合并写缓冲区对同一内存地址的多次写, 减少对内存总线的占用

### 写缓冲区带来的问题
处理器对内存的读/写操作的执行顺序, 不一定与内存实际发生的读/写操作顺序一致





常见的缓存一致性协议有：MESI，MESIF（MESIF是缓存行的状态标识，M:Modified, E: Exclusive, S:Shared, I:Invalid, F: Forwad），通过标记缓存行的状态和处理器间的通讯来实现。


[happens-before](http://ifeve.com/tag/happens-before/)
[指令重排、内存屏障概念解析](https://www.cnblogs.com/duanxz/archive/2013/01/15/2861606.html)

Intel文档：https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html

cmpxchg指令：https://www.felixcloutier.com/x86/cmpxchg

x86寄存器：http://www.cs.virginia.edu/~evans/cs216/guides/x86.html

ZF标志：https://en.wikipedia.org/wiki/Zero_flag

The JSR-133 Cookbook for Compiler Writers  http://gee.cs.oswego.edu/dl/jmm/cookbook.html


