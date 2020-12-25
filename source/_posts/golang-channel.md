---
title: golang channel
date: 2020-10-24 00:22:57
summary: 学习一下golang的channel是怎么实现的
tags:
  - golang
categories:
  - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co108-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co108.jpg
---

## 源码
```golang
type hchan struct {
    qcount   uint           // 循环列表元素个数. chan 中已经接收但还没被取走的元素的个数，函数 len 可以返回这个字段的值
    dataqsiz uint           // 循环队列的大小, cap函数可以返回这个字段的值
    buf      unsafe.Pointer // 循环队列的指针
    elemsize uint16         // chan中元素的大小
    closed   uint32         // 是否已close
    elemtype *_type         // chan中元素类型
    sendx    uint           // send在buffer中的索引
    recvx    uint           // recv在buffer中的索引
    recvq    waitq          // receiver的等待队列
    sendq    waitq          // sender的等待队列 
    
    // 互拆锁
    // lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
    lock mutex
}
```

sendq和recvq的类型是waitq的结构体：
```golang
type waitq struct {
    first *sudog
    last  *sudog
}
```

waitq里面连接的是一个sudog双向链表，保存的是等待的goroutine 。整个chan的图例大概是这样：
![channel内部结构-整体](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/golang-channel/golang-chan-sudog.png)

我们通过汇编结果也可以查看到make(chan int)这句代码会调用到runtime的makechan函数中：
```golang
// maxAlign是8，那么maxAlign-1的二进制就是111，然后和int(unsafe.Sizeof(hchan{}))取与就是取它的低三位，hchanSize就得到的是8的整数倍，做对齐使用。
const (
    maxAlign  = 8
    hchanSize = unsafe.Sizeof(hchan{}) + uintptr(-int(unsafe.Sizeof(hchan{}))&(maxAlign-1)) 
)

func makechan(t *chantype, size int) *hchan {
    elem := t.elem

    // 略去检查代码
    ... 
    //计算需要分配的buf空间
    mem, overflow := math.MulUintptr(elem.size, uintptr(size))
    if overflow || mem > maxAlloc-hchanSize || size < 0 {
        panic(plainError("makechan: size out of range"))
    }

    var c *hchan
    switch {
    case mem == 0: // 缓冲区所需大小为 0，那么在为 hchan 分配内存时，只需要分配 sizeof(hchan) 大小的内存
        // chan的size或者元素的size是0，不必创建buf
        c = (*hchan)(mallocgc(hchanSize, nil, true))
        // Race detector 
        c.buf = c.raceaddr()
    case elem.ptrdata == 0: // 缓冲区所需大小不为 0，而且数据类型不是指针，那么就分配连续的内存
        // 元素不是指针，分配一块连续的内存给hchan数据结构和buf
        c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
        // 表示hchan后面在内存里紧跟着就是buf
        c.buf = add(unsafe.Pointer(c), hchanSize)
    default: // 缓冲区所需大小不为 0，而且数据类型包含指针，那么就不使用add的方式让hchan和buf放在一起了，而是单独的为buf申请一块内存
        // 元素包含指针，那么单独分配buf
        c = new(hchan)
        c.buf = mallocgc(mem, elem, true)
    }

    // 元素大小、类型、容量都记录下来
    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)

    return c
}
```

chansend 函数是在编译器解析到 c <- x 这样的代码的时候插入的，本质上就是把一个用户元素投递到 hchan 的 ringbuffer 中。chansend 调用的时候，一般用户会遇到两种情况：

投递成功，非常顺利，正常返回
投递受阻，该函数阻塞，goroutine 切走

接下来，我们看下 chansend 究竟是做了什么。

```golang
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // channel 的所有操作，都在互斥锁下；
    lock(&c.lock)
    // 如果投递的目标是已经关闭的 channel，那么直接 panic；
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }
    // 场景一：性能最好的场景，我投递的元素刚好有人在等着（那我直接给他就完了）;
    // 调用的是 send 函数，这个函数后面详细阐述，其实非常简单，递增 sendx, recvx 的索引，然后直接把元素给到等他的人，并且唤醒他；
    if sg := c.recvq.dequeue(); sg != nil {
        send(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true
    }
    // 场景二：ringbuffer 还有空间，那么把元素放好，递增索引，就可以返回了；
    if c.qcount < c.dataqsiz {
        // 复制，赋值好元素；
        qp := chanbuf(c, c.sendx)
        typedmemmove(c.elemtype, qp, ep)
        // 递增索引
        c.sendx++
        // 回环空间
        if c.sendx == c.dataqsiz {
            c.sendx = 0
        }
        // 递增元素个数
        c.qcount++
        unlock(&c.lock)
        return true
    }
    // 判断是否需要阻塞？如果是非阻塞的，那么就直接解锁返回了，如果是阻塞的场景，那么就会走到下面的逻辑哦；
    // chan <- 和 <-chan 的场景，都是 true，但是会有其他场景这里是 false，可以提前想下？
    if !block {
        unlock(&c.lock)
        return false
    }
    // 代码走到这里，说明都是因为条件不满足，要阻塞当前 goroutine，所以做的事情本质上就是保留好通知路径，等待条件满足，会在这个地方唤醒；
    gp := getg()
    mysg := acquireSudog()
    mysg.releasetime = 0
    mysg.elem = ep
    mysg.waitlink = nil
    mysg.g = gp
    mysg.isSelect = false
    mysg.c = c
    gp.waiting = mysg
    gp.param = nil
    // 把 goroutine 相关的线索结构入队，等待条件满足的唤醒；
    c.sendq.enqueue(mysg)
    // goroutine 切走，让出 cpu 执行权限；
    gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)

    // 到这就是某些人唤醒该 goroutine 了。
    // 下面就是唤醒之后的逻辑了；
    if mysg != gp.waiting {
        throw("G waiting list is corrupted")
    }
    // 做一些资源的释放和环境的清理。
    gp.waiting = nil
    gp.activeStackChans = false
    if gp.param == nil {
        // 做一些校验
        if c.closed == 0 {
            throw("chansend: spurious wakeup")
        }
        panic(plainError("send on closed channel"))
    }
    gp.param = nil
    mysg.c = nil
    releaseSudog(mysg)
    return true
}
```


编译器在遇到 <-c 和 v, ok := <-c  的语句的时候，会换成对应的 chanrecv1 ，chanrecv2 函数，这两个函数本质上都是一个简单的封装，元素出队的实现函数是 chanrecv ，我们详细分析下这个函数。block 都等于 true（同样的，只有 select 的时候，block 才会是 false ）
```golang
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	// 特殊场景：非阻塞模式，并且没有元素的场景直接就可以返回了，这个分支是快速分支，下面的代码都是在锁内的；
	if !block && (c.dataqsiz == 0 && c.sendq.first == nil ||
		c.dataqsiz > 0 && atomic.Loaduint(&c.qcount) == 0) &&
		atomic.Load(&c.closed) == 0 {
		return
	}

	// 以下所有的逻辑都在锁内；
	lock(&c.lock)

	if c.closed != 0 && c.qcount == 0 {
		if raceenabled {
			raceacquire(c.raceaddr())
		}
		unlock(&c.lock)
		if ep != nil {
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}

	// 场景：如果发现有个人（sender）正在等着别人接收，那么刚刚好，直接把它的元素给到我们这里就好了；
	if sg := c.sendq.dequeue(); sg != nil {
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}

	// 场景：ringbuffer 还有空间存元素，那么下面就可以把元素放到 ringbuffer 放好，递增索引，就可以返回了；
	if c.qcount > 0 {
		// 存元素
		qp := chanbuf(c, c.recvx)
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		// 递增索引
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

	// 代码到这说明 ringbuffer 空间是不够的，后面学会要做两个事情，是否需要阻塞？
	// 如果 block 为 false ，那么直接就退出了，返回对应的返回值；
	if !block {
		unlock(&c.lock)
		return false, false
	}

	// 到这就说明要阻塞等待了，下面唯一要做的就是给阻塞做准备（准备好唤醒的条件）
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	// goroutine 作为一个 waiter 入队列，等待条件满足之后，从这个队列里取出来唤醒；
	c.recvq.enqueue(mysg)
	// goroutine 切走，交出 cpu 执行权限
	goparkunlock(&c.lock, waitReasonChanReceive, traceEvGoBlockRecv, 3)

	// 这里是被唤醒的开始的地方；
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	// 下面做一些资源的清理
	gp.waiting = nil
	closed := gp.param == nil
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, !closed
}
```

## 参考文章
1. 《极客时间-Go并发编程实战》
2. [多图详解Go中的Channel源码](https://www.luozhiyun.com/archives/427)
3. [channel & select 源码分析](https://talkgo.org/t/topic/75)
4. [Go语言的有缓冲channel和无缓冲channel](https://zhuanlan.zhihu.com/p/101063277)
5. https://speakerdeck.com/kavya719/understanding-channels
6. https://codeburst.io/diving-deep-into-the-golang-channels-549fd4ed21a8
7. https://blog.csdn.net/weixin_30257433/article/details/101833959
8. https://segmentfault.com/a/1190000017958702
9. https://juejin.cn/post/6895738899348324359

http://legendtkl.com/2017/08/06/golang-channel-implement/

https://www.pengrl.com/p/21027/

