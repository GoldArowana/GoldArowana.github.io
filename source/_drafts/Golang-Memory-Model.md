---
title: Golang Memory Model
date: 2021-05-01 13:15:50
summary: golang内存模型
tags:
    - golang
categories:
    - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co199-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co199.jpg
---

## happens-before
1. init函数顺序: a依赖b。那么b包的init函数happens-before于a包的init函数
2. 启动goroutine的go语句的执行, 一定happens-before此goroutine内代码的执行
3. channel
   1. 发送 happens-before 接受
   2. close() happens-before 从关闭的Channel中读取出一个零值
   3. 对于unbuffered的Channel, 读取数据的调用一定happens-before往此Channel发送数据的调用完成
   4. 对于buffered channel, 如果Channel的容量是m(m>0), 那么第n个receive一定happens-before 第n+m个send的完成
4. 互斥锁: 第n次的m.Unlock一定happens-before第n+1次m.Lock方法的返回
5. Once: 对于once.Do(f), 函数f一定会在Do方法返回前执行


## 参考文章
1. https://golang.org/ref/mem
2. 《极客时间-Go并发编程实战》
