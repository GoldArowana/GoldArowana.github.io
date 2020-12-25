---
title: 高性能网络模型
date: 2022-01-05 13:03:33
summary: select? poll? epoll? c10k? c100k?
tags:
    - internet
    - design patterns
    - os
categories:
    - design patterns
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co76.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co76.jpg
---
## I/O多路复用
> 我们可以把标准输入、套接字等都看做 I/O 的一路，多路复用的意思，就是在任何一路 I/O 有“事件”发生的情况下，通知应用程序去处理相应的 I/O 事件，这样我们的程序就变成了“多面手”，在同一时刻仿佛可以处理多个 I/O 事件。

## select
> 仅知道有I/O事件发生，却不知是哪几个流，只会无差异轮询所有流，找出能读数据或写数据的流进行操作。同时处理的流越多，无差别轮询时间越长 - O(n)。

> 当socket较多时，每次select都要通过遍历FD_SETSIZE个socket，不管是否活跃，这会浪费很多CPU时间。如果能给 socket 注册某个回调函数，当他们活跃时，自动完成相关操作，即可避免轮询，这就是epoll与kqueue。

## poll
> poll没有最大文件描述符数量的限制

## iocp(Windows)
## epoll(Linux)
> 在调用epoll_create时，内核除了帮我们在epoll文件系统里建了个file结点，在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket外，还会再建立一个list链表，用于存储准备就绪的事件，当epoll_wait调用时，仅仅观察这个list链表里有没有数据即可。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。所以，epoll_wait非常高效。

> epoll是通过内核与用户空间mmap同一块内存实现的。mmap将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址（不管是用户空间还是内核空间都是虚拟地址，最终要通过地址映射映射到物理地址），使得这块物理内存对内核和对用户均可见，减少用户态和内核态之间的数据交换。内核可以直接看到epoll监听的句柄，效率高。

### ET
> 当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。
### LT(默认)
> 当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

## kqueue(FreeBsd)

## 参考文章
1. 《极客时间-网络编程实战》
2. [源码解读poll/select内核机制](http://gityuan.com/2019/01/05/linux-poll-select/)
3. [何兴鹏: select函数源码简析](https://cloud.tencent.com/developer/article/1517892)
4. [Linux select源码分析](https://www.jianshu.com/p/95b50b026895)
5. [Linux内核select源码剖析](http://www.pandademo.com/2016/11/linux-kernel-select-source-dissect/)
6. [select](https://en.wikipedia.org/wiki/Select_(Unix))
7. [Polling](https://en.wikipedia.org/wiki/Polling_(computer_science))
8. [epoll](https://en.wikipedia.org/wiki/Epoll)
9. [kqueue](https://en.wikipedia.org/wiki/Kqueue)
10. [流？I/O 操作？阻塞？epoll?](https://learnku.com/articles/41814)
11. [---慢更-基于 go 的 IM 聊天](https://learnku.com/articles/37534)
12. [深入理解IO复用之epoll](https://zhuanlan.zhihu.com/p/87843750)
13. [如果这篇文章说不清epoll的本质，那就过来掐死我吧！ （1）](https://zhuanlan.zhihu.com/p/63179839)
14. [Scalable IO in Java (Doug Lea)](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)
15. [select、poll、epoll之间的区别(搜狗面试)](https://www.cnblogs.com/aspirant/p/9166944.html)
16. [Linux服务端编程。关于IO复用，在epoll出现多年的情况下，select和poll还有使用场景吗？](https://www.zhihu.com/question/30262027/answer/53206495)
17. [高性能跨平台网络IO（Reactor、epoll、iocp）总结](https://www.cnblogs.com/zl1991/p/12098272.html)
18. [《Scalable IO in Java》译文](https://www.cnblogs.com/dafanjoy/p/11217708.html)
19. [epoll底层红黑树使用部分源码剖析：为什么使用红黑树以及如何使用红黑树](https://blog.csdn.net/Mr_H9527/article/details/99745659)
20. [Linux下的I/O复用与epoll详解(ET与LT)](https://blog.csdn.net/eyucham/article/details/86502117)
21. [Linux内核剖析-----IO复用函数poll内核源码剖析](https://blog.csdn.net/Eunice_fan1207/article/details/99641348)
22. [Linux内核剖析-----IO复用函数epoll内核源码剖析](https://blog.csdn.net/Eunice_fan1207/article/details/99674021)
23. [彻底学会使用epoll(一)——ET模式实现分析](http://blog.chinaunix.net/uid-28541347-id-4273856.html)


https://www.zhihu.com/question/20122137/answer/146866418

https://www.jianshu.com/p/e4d1c485c32a

https://www.cnblogs.com/jukan/p/5272257.html