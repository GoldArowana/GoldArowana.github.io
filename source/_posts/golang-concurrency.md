---
title: golang 并发 
date: 2020-08-01 13:15:50 
summary: golang并发学习笔记
tags:
    - golang 
categories:
    - golang 
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co62.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co62.jpg
---

## 互斥锁Mutex

### 示例: 用mutex解决并发计数问题
```go
import (
	"fmt"
	"sync"
)

type Counter struct {
	sync.Mutex
	Count int64
}

func (c *Counter) Incr() {
	c.Lock()
	c.Count++
	c.Unlock()
}

var Incr10k = func(counter *Counter, wg *sync.WaitGroup) {
	defer wg.Done()

	for j := 0; j < 100000; j++ {
		counter.Incr()
	}
}

func main() {
	var counter Counter
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go Incr10k(&counter, &wg)
	}

	wg.Wait()
	fmt.Println(counter.Count)
}

```

### Mutex实现
#### go最早期的Mutex实现: 最简单的实现
key: 0表示锁未被持有。1表示锁被持有, 没有等待者。值为n表示锁被持有, 还有n-1个等待着。
sema: 等待着队列使用的信号量。当锁被占用的时候, 抢锁失败的线程会调用semacquire(), 使用信号量将自己休眠。等锁释放的时候, 信号量将会唤醒队列的头部等待着。

1. 公平: 排队等待
2. 不可重入
3. 其他协程可以释放锁
#### go 0.x版本: 新增乐观抢锁
非公平: 新协程可能会抢, 而不是每次都在FIFO的队列头部获取

```golang
type Mutex struct {
    state int32
    sema uint32
}
```

state这一个字段包含多个意义:
- 最高30位表示阻塞等待的waiter数量
- 接下来的1位表示是否有唤醒的goroutine. (这个标识有什么作用? 因为被唤醒的goroutine没抢到锁后, 不需要使waiter++, 所以需要区分是否是被唤醒的。)
- 最低1位表示这个锁是否被持有


获取锁的过程:
```golang
func (m *Mutex) Lock() {
    // Fast path: 幸运case，能够直接获取到锁
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        return
    }

    awoke := false
    for {
        old := m.state
        new := old | mutexLocked // 新状态加锁
        if old&mutexLocked != 0 {
            new = old + 1<<mutexWaiterShift //等待者数量加一
        }
        if awoke {
            // goroutine是被唤醒的，
            // 新状态清除唤醒标志
            new &^= mutexWoken
        }
        if atomic.CompareAndSwapInt32(&m.state, old, new) {//设置新状态
            if old&mutexLocked == 0 { // 锁原状态未加锁
                break
            }
            runtime.Semacquire(&m.sema) // 请求信号量
            awoke = true
        }
    }
}
```


释放锁的过程:
```golang
func (m *Mutex) Unlock() {
    // Fast path: drop lock bit.
    new := atomic.AddInt32(&m.state, -mutexLocked) //去掉锁标志
    if (new+mutexLocked)&mutexLocked == 0 { //本来就没有加锁
        panic("sync: unlock of unlocked mutex")
    }

    old := new
    for {
        if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0 { // 没有等待者，或者有唤醒的waiter，或者锁原来已加锁
            return
        }
        new = (old - 1<<mutexWaiterShift) | mutexWoken // 新状态，准备唤醒goroutine，并设置唤醒标志
        if atomic.CompareAndSwapInt32(&m.state, old, new) {
            runtime.Semrelease(&m.sema)
            return
        }
        old = m.state
    }
}
```
#### go 1.5版本: 新增自旋抢锁
非公平: 新协程可能会抢, 而不是每次都在FIFO的队列头部获取。而且相比于之前的版本, 对比新来请求锁的goroutine和被唤醒的goroutine, 都会自旋地多尝试几次

```golang
func (m *Mutex) Lock() {
    // Fast path: 幸运之路，正好获取到锁
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        return
    }

    awoke := false
    iter := 0
    for { // 不管是新来的请求锁的goroutine, 还是被唤醒的goroutine，都不断尝试请求锁
        old := m.state // 先保存当前锁的状态
        new := old | mutexLocked // 新状态设置加锁标志
        if old&mutexLocked != 0 { // 锁还没被释放
            if runtime_canSpin(iter) { // 还可以自旋
                if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                    atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                    // 将锁的唤醒标记置为1，表示已经有醒着的线程正在获取锁，Unlock的时候便无需唤醒新的线程                    
                    awoke = true
                }
                runtime_doSpin()
                iter++
                continue // 自旋，再次尝试请求锁
            }
            new = old + 1<<mutexWaiterShift
        }
        if awoke { // 唤醒状态
            if new&mutexWoken == 0 {
                panic("sync: inconsistent mutex state")
            }
            new &^= mutexWoken // 新状态清除唤醒标记
        }
        if atomic.CompareAndSwapInt32(&m.state, old, new) {
            if old&mutexLocked == 0 { // 旧状态锁已释放，新状态成功持有了锁，直接返回
                break
            }
            runtime_Semacquire(&m.sema) // 阻塞等待
            awoke = true // 被唤醒
            iter = 0
        }
    }
}
```

#### go 1.9版本: 新增饥饿模式
```golang
type Mutex struct {
    state int32
    sema uint32
}
```

state这一个字段包含多个意义:
- 最高29位表示阻塞等待的waiter数量
- 接下来的1位表示饥饿标记(此版本新增)
- 再接下来的1位表示是否有唤醒的goroutine
- 最低1位表示这个锁是否被持有

```golang
func (m *Mutex) Lock() {
    // Fast path: 幸运之路，一下就获取到了锁
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        return
    }
    // Slow path：缓慢之路，尝试自旋竞争或饥饿状态下饥饿goroutine竞争
    m.lockSlow()
}

func (m *Mutex) lockSlow() {
    var waitStartTime int64
    starving := false // 此goroutine的饥饿标记
    awoke := false // 唤醒标记
    iter := 0 // 自旋次数
    old := m.state // 当前的锁的状态
    for {
        // 锁是非饥饿状态，锁还没被释放，尝试自旋
        if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
            if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                awoke = true
            }
            runtime_doSpin()
            iter++
            old = m.state // 再次获取锁的状态，之后会检查是否锁被释放了
            continue
        }
        new := old
        if old&mutexStarving == 0 {
            new |= mutexLocked // 非饥饿状态，加锁
        }
        if old&(mutexLocked|mutexStarving) != 0 {
            new += 1 << mutexWaiterShift // waiter数量加1
        }
        if starving && old&mutexLocked != 0 {
            new |= mutexStarving // 设置饥饿状态
        }
        // 不管是获得了锁还是进入休眠，我们都需要清除 mutexWoken 标记
        if awoke {
            if new&mutexWoken == 0 {
                throw("sync: inconsistent mutex state")
            }
            new &^= mutexWoken // 新状态清除唤醒标记
        }
        // 成功设置新状态
        if atomic.CompareAndSwapInt32(&m.state, old, new) {
            // 原来锁的状态已释放，并且不是饥饿状态，正常请求到了锁，返回
            if old&(mutexLocked|mutexStarving) == 0 {
                break // locked the mutex with CAS
            }
            // 处理饥饿状态

            // 如果以前就在队列里面，加入到队列头
            queueLifo := waitStartTime != 0
            if waitStartTime == 0 {
                waitStartTime = runtime_nanotime()
            }
            // 阻塞等待
            runtime_SemacquireMutex(&m.sema, queueLifo, 1)
            // 唤醒之后检查锁是否应该处于饥饿状态
            starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
            old = m.state
            // 如果锁已经处于饥饿状态，直接抢到锁，返回
            if old&mutexStarving != 0 {
                if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
                    throw("sync: inconsistent mutex state")
                }
                // 加锁并且将waiter数减1（加mutexLocked是加锁，减1<<mutexWaiterShift是减少waiter）
                delta := int32(mutexLocked - 1<<mutexWaiterShift)
                if !starving || old>>mutexWaiterShift == 1 { // 当前goroutine等待还没超过1毫秒, 且没有其他的waiter 
                    delta -= mutexStarving // 最后一个waiter或者已经不饥饿了，清除饥饿标记
                    // （同理，减mutexStarving是清楚饥饿标记）
                }
                atomic.AddInt32(&m.state, delta)
                break
            }
            awoke = true
            iter = 0
        } else {
            old = m.state
        }
    }
}

func (m *Mutex) Unlock() {
    // Fast path: drop lock bit.
    new := atomic.AddInt32(&m.state, -mutexLocked)
    if new != 0 {
        m.unlockSlow(new)
    }
}

func (m *Mutex) unlockSlow(new int32) {
    if (new+mutexLocked)&mutexLocked == 0 {
        throw("sync: unlock of unlocked mutex")
    }
    if new&mutexStarving == 0 {
        old := new
        for {
            if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
                return
            }
            new = (old - 1<<mutexWaiterShift) | mutexWoken
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                runtime_Semrelease(&m.sema, false, 1)
                return
            }
            old = m.state
        }
    } else {
        runtime_Semrelease(&m.sema, true, 1)
    }
}
```

## 读写锁RWMutex

```golang
type RWMutex struct {
  w           Mutex   // 互斥锁解决多个writer的竞争。为 writer 的竞争锁而设计。
  writerSem   uint32  // writer信号量
  readerSem   uint32  // reader信号量
  readerCount int32   // reader的数量。记录当前 reader 的数量（以及是否有 writer 竞争锁）, rw.readerCount是负值的时候，意味着此时有writer等待请求锁
  readerWait  int32   // writer等待完成的reader的数量。记录 writer 请求锁时需要等待 read 完成的 reader 的数量；
}

const rwmutexMaxReaders = 1 << 30 // 最大的 reader 数量
```

读读相关操作
```golang
func (rw *RWMutex) RLock() {
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
            // rw.readerCount是负值的时候，意味着此时有writer等待请求锁，因为writer优先级高，所以把后来的reader阻塞休眠
        runtime_SemacquireMutex(&rw.readerSem, false, 0)
    }
}
func (rw *RWMutex) RUnlock() {
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
        rw.rUnlockSlow(r) // 有等待的writer
    }
}
func (rw *RWMutex) rUnlockSlow(r int32) {
    if atomic.AddInt32(&rw.readerWait, -1) == 0 {
        // 最后一个reader了，writer终于有机会获得锁了
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```


写锁相关操作
```golang
func (rw *RWMutex) Lock() {
    // 首先解决其他writer竞争问题
    rw.w.Lock()
    // 反转readerCount，告诉reader有writer竞争锁
    r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
    // 如果当前有reader持有锁，那么需要等待
    if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
        runtime_SemacquireMutex(&rw.writerSem, false, 0)
    }
}

func (rw *RWMutex) Unlock() {
    // 告诉reader没有活跃的writer了
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
    
    // 唤醒阻塞的reader们
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem, false, 0)
    }
    // 释放内部的互斥锁
    rw.w.Unlock()
}
```

## WaitGroup

结构体定义:
```golang
type WaitGroup struct {
    // 避免复制使用的一个技巧，可以告诉vet工具违反了复制使用的规则
    noCopy noCopy
    // 64bit(8bytes)的值分成两段，高32bit是计数值，低32bit是waiter的计数
    // 另外32bit是用作信号量的
    // 因为64bit值的原子操作需要64bit对齐，但是32bit编译器不支持，所以数组中的元素在不同的架构中不一样，具体处理看下面的方法
    // 总之，会找到对齐的那64bit作为state，其余的32bit做信号量
    state1 [3]uint32
}


// 得到state的地址和信号量的地址
func (wg *WaitGroup) state() (statep *uint64, semap *uint32) {
    if uintptr(unsafe.Pointer(&wg.state1))%8 == 0 {
        // 如果地址是64bit对齐的，数组前两个元素做state，后一个元素做信号量
        return (*uint64)(unsafe.Pointer(&wg.state1)), &wg.state1[2]
    } else {
        // 如果地址是32bit对齐的，数组后两个元素用来做state，它可以用来做64bit的原子操作，第一个元素32bit用来做信号量
        return (*uint64)(unsafe.Pointer(&wg.state1[1])), &wg.state1[0]
    }
}    
```

Add操作:
```golang
func (wg *WaitGroup) Add(delta int) {
  // statep表示wait数和计数值
  // 低32位表示wait数，高32位表示计数值
   statep, semap := wg.state()
   // uint64(delta)<<32 将delta左移32位
    // 因为高32位表示计数值，所以将delta左移32，增加到技术上
   state := atomic.AddUint64(statep, uint64(delta)<<32)
   // 当前计数值
   v := int32(state >> 32)
   // 阻塞在检查点的wait数
   w := uint32(state)
   if v > 0 || w == 0 {
      return
   }
   
   // 如果计数值v为0并且waiter的数量w不为0，那么state的值就是waiter的数量
    // 将waiter的数量设置为0，因为计数值v也是0,所以它们俩的组合*statep直接设置为0即可。此时需要并唤醒所有的waiter
   *statep = 0
   for ; w != 0; w-- {
      runtime_Semrelease(semap, false, 0)
   }
}
```

Done操作:
```golang
// Done方法实际就是计数器减1
func (wg *WaitGroup) Done() { 
  wg.Add(-1)
}
```

Wait操作:
```golang
func (wg *WaitGroup) Wait() {
   // statep表示wait数和计数值
   // 低32位表示wait数，高32位表示计数值
   statep, semap := wg.state()
   for {
      state := atomic.LoadUint64(statep)
      // 将state右移32位，表示当前计数值
      v := int32(state >> 32)
      // w表示waiter等待值
      w := uint32(state)
      if v == 0 {
         // 如果当前计数值为零，表示当前子goroutine已全部执行完毕，则直接返回
         return
      }
      // 否则使用原子操作将state值加一。
      if atomic.CompareAndSwapUint64(statep, state, state+1) {
         // 阻塞休眠等待
         runtime_Semacquire(semap)
         // 被唤醒，不再阻塞，返回
         return
      }
   }
}
```

## 参考资料

1. 《极客时间-Go并发编程实战课》
2. [sync.mutex 源代码分析](https://colobu.com/2018/12/18/dive-into-sync-mutex/)
3. [draveness-同步原语与锁](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/#mutex)
4. [多图详解Go的互斥锁Mutex](https://www.cnblogs.com/luozhiyun/p/14157542.html)
5. [Go中由WaitGroup引发对内存对齐思考](https://www.wangt.cc/2021/01/go%e4%b8%ad%e7%94%b1waitgroup%e5%bc%95%e5%8f%91%e5%af%b9%e5%86%85%e5%ad%98%e5%af%b9%e9%bd%90%e6%80%9d%e8%80%83/)

https://golang.design/under-the-hood/zh-cn/part1basic/ch05sync/

https://draveness.me/whys-the-design-communication-shared-memory/

https://blog.csdn.net/fwhezfwhez/article/details/82900498

https://studygolang.com/articles/29935?fr=sidebar
