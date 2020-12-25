---
title: go sysmon
date: 2020-12-30 16:52:26
summary: golang的系统监控器
tags:
  - golang
categories:
  - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co110-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co110.jpg
---


## 基本原理
> 系统监控是 Go 语言运行时的重要组成部分，它会每隔一段时间检查 Go 语言运行时，确保程序没有进入异常状态

> 系统监控的触发时间就会稳定在 10ms

## 检查死锁

## 运行计时器
获取下一个需要被触发的计时器

> 当前调度器需要执行垃圾回收或者所有处理器都处于闲置状态时，如果没有需要触发的计时器，那么系统监控可以暂时陷入休眠

## 轮循网络
获取需要处理的到期文件描述符

> 如果上一次轮询网络已经过去了 10ms，那么系统监控还会在循环中轮询网络，检查是否有待执行的文件描述符

## 抢占处理器
抢占运行时间较长的或者处于系统调用的 Goroutine

> 1. 当处理器处于 _Prunning 或者 _Psyscall 状态时，如果上一次触发调度的时间已经过去了 10ms，我们会通过 runtime.preemptone 抢占当前处理器；
> 1. 当处理器处于 _Psyscall 状态时，在满足以下两种情况下会调用 runtime.handoffp 让出处理器的使用权：
>   1. 当处理器的运行队列不为空或者不存在空闲处理器时2；
>   2. 当系统调用时间超过了 10ms 时3；


## 垃圾回收
在满足条件时触发垃圾收集回收内存

> 如果需要触发垃圾回收，我们会将用于垃圾回收的 Goroutine 加入全局队列，让调度器选择合适的处理器去执行。

## 参考文章
https://golang.design/under-the-hood/zh-cn/part2runtime/ch06sched/sysmon/

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sysmon/

https://gocn.vip/topics/9884