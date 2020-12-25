---
title: golang栈空间管理
date: 2021-3-2 00:33:49
summary: 学习一下golang的栈空间管理机制
tags:
  - golang
categories:
  - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co109-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co109.jpg
---

## 线程栈
不同架构下, 默认线程栈的大小不同。在x86_64架构下, 默认栈大小为2MB

## 逃逸分析
```c
// 悬挂指针示例
int *dangling_pointer() {
    int i = 2;
    return &i;
}
```
> 当 dangling_pointer 函数返回后，它的本地变量会被编译器回收，调用方获取的是危险的悬挂指针，我们不确定当前指针指向的值是否合法时，这种问题在大型项目中是比较难以发现和定位的。

> Go 语言的逃逸分析遵循以下两个不变性：
> 
> 1. 指向栈对象的指针不能存在于堆中 (一旦函数返回后函数栈会被回收，该指针指向的值就不再合法)
> 1. 指向栈对象的指针不能在栈对象回收后存活(栈底部有一个指针指向了栈顶, 那么当栈顶的函数释放后, 栈底的指针不再合法)

## 栈内存空间
> Goroutine 的栈内存空间和栈结构也在早期几个版本中发生过一些变化：
> 1. v1.0 ~ v1.1 — 最小栈内存空间为 4KB
> 1. v1.2 — 将最小栈内存提升到了 8KB(其目的是为了减轻分段栈中的栈分裂对程序的性能影响)
> 1. v1.3 — 使用连续栈替换之前版本的分段栈
> 1. v1.4 — 将最小栈内存降低到了 2KB

## 分段栈
> 当 Goroutine 调用的函数层级或者局部变量需要的越来越多时，运行时会调用 runtime.morestack:go1.2 和 runtime.newstack:go1.2 创建一个新的栈空间，这些栈空间虽然不连续，但是当前 Goroutine 的多个栈空间会以链表的形式串联起来，运行时会通过指针找到连续的栈片段

> 分段栈机制虽然能够按需为当前 Goroutine 分配内存并且及时减少内存的占用，但是它也存在两个比较大的问题：
> 
> 1. 如果当前 Goroutine 的栈几乎充满，那么任意的函数调用都会触发栈扩容，当函数返回后又会触发栈的收缩，如果在一个循环中调用函数，栈的分配和释放就会造成巨大的额外开销，这被称为热分裂问题（Hot split）；
> 1. 一旦 Goroutine 使用的内存越过了分段栈的扩缩容阈值，运行时会触发栈的扩容和缩容，带来额外的工作量；

## 连续栈
> 分段栈的扩张操作是在另一个栈上进行的，这两个栈彼此没有连续。这种设计的缺陷很容易破坏缓存的局部性原理，从而降低程序的运行时性能

> 使用连续栈而不是分段栈的目的是，利用局部性优势提升执行速度，原理是CPU读取地址时会将相邻的内存读取到访问速度比内存快的多级cache中，地址连续性越好，L1、L2、L3 cache命中率越高，速度也就越快

> 栈的收缩是垃圾回收的过程中实现的．当检测到栈只使用了不到1/4时，栈缩小为原来的1/2

## 文章列表
1. [Go语言是如何处理栈的](https://tonybai.com/2014/11/05/how-stacks-are-handled-in-go/)
2. [深入研究goroutine栈](http://www.huamo.online/2019/06/25/%E6%B7%B1%E5%85%A5%E7%A0%94%E7%A9%B6goroutine%E6%A0%88/)
3. [函数——go世界中的一等公民](https://segmentfault.com/a/1190000023340324)
4. [深入理解golang 的栈](https://www.jianshu.com/p/7ec9acca6480)

https://kirk91.github.io/posts/2d571d09/

https://www.luozhiyun.com/archives/513

https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-stack-management/

https://www.bookstack.cn/read/GoExpertProgramming/chapter04-4.3-escape_analysis.md

1. [深入解析go-连续栈](https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.5.html)