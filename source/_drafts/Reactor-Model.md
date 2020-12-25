---
title: Reactor-Model
date: 2021-03-15 13:03:33
summary: Reactor模式
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co201-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co201.jpg
---
## 传统多线程


## 单线程Reactor模式
异步非阻塞IO

> 传统方案，也就是一个请求开一个线程在请求数量庞大时是无法接受的，所以我们需要去寻求一种高伸缩性的方案，请求数量低的时候效率很高而且请求数量很庞大的时候依旧不会崩溃。
主要有以下几点 当客户端数量庞大的时候，我们的服务要能够优雅地降低速度但是不至于崩溃。
> 
> 但是任务分发可能会更慢，而且必须手动将事件和对应的处理动作绑定，相应的编程实现也会更难。

1. 当一个NIO线程同时处理成百上千的链路，性能上无法支撑，即使NIO线程的CPU负荷达到100%，也无法完全处理消息
2. 当NIO线程负载过重后，处理速度会变慢，会导致大量客户端连接超时，超时之后往往会重发，更加重了NIO线程的负载。
3. 可靠性低，一个线程意外死循环，会导致整个通信系统不可用

## Reactor线程池模式
使用线程池


> 在绝大多数场景下，该模型都能满足性能需求。但是，在一些特殊的应用场景下，如服务器会对客户端的握手消息进行安全认证。这类场景下，单独的一个Acceptor线程可能会存在性能不足的问题。

## Reactor主从模式
单Reactor缺点:

1. 单Reactor需要响应连接和读写事件，单线程处理任务较多
2. 单Reactor连接和读写事件放在一块处理，会互相影响，而且本身读写事件是一个比较
耗时的操作，当一个读写事件处理事件太长，那么势必会影响下一个连接事件的处理，影响
用户连接，这是个非常不友好的事情，很影响体验


## Netty中Reactor主从模式的变种
去掉了Reactor的线程池。

> 当系统在运行过程中，如果频繁的进行线程上下文切换，会带来额外的性能损耗。
> 
> 多线程并发执行某个业务流程，业务开发者还需要时刻对线程安全保持警惕，哪些数据可能会被并发修改，如何保护？这不仅降低了开发效率，也会带来额外的性能损耗。
> 
> 为了解决上述问题，Netty采用了串行化设计理念
>
> 从消息的读取、编码以及后续Handler的执行，始终都由IO线程EventLoop负责，这就意味着整个流程不会进行线程上下文的切换，数据也不会面临被并发修改的风险。这也解释了为什么Netty线程模型去掉了Reactor主从模型中线程池。

## 参考文章
1. [深入Netty逻辑架构，从Reactor线程模型开始](https://zhuanlan.zhihu.com/p/381483820)
2. [<scalable io in java> 翻译](https://haoxpdp.github.io/categories/sth/scalableIOinJava/)
3. [一步一图，带你走进 Netty 的世界！](https://www.cnblogs.com/shoshana-kong/p/14652527.html)
4. [高性能：《一遍文章带你看懂 Netty世界》](https://zhuanlan.zhihu.com/p/70970558)
5. [高性能跨平台网络IO（Reactor、epoll、iocp）总结](https://www.cnblogs.com/zl1991/p/12098272.html)
6. [Reactor(主从)原理详解与实现](https://blog.csdn.net/weixin_38312719/article/details/108271087)
7. [单Reactor和主从Reactor模式介绍](https://blog.csdn.net/qq_37535749/article/details/115400243)
8. 