---
title: java原子类cas的实现原理 
date: 2021-01-19 13:26:15
summary: java并发必学-原子操作cas
categories: 
    - java
tags:
    - 并发
    - atomic
    - cas
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co5.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co5.jpg
---

## cas简介
### cas是什么
> CAS(compare and swap)，比较和交换，是原子操作的一种，
可用于在多线程编程中实现不被打断的数据交换操作，
从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。
该操作通过将内存中的值与指定数据进行比较，当数值一样时将内存中的数据替换为新的值。
现代的大多数CPU都实现了CAS,它是一种无锁(lock-free),且非阻塞的一种算法，保持数据的一致性。


### cas能干什么
它可以用于解决多线程并发安全的问题。比如多线程修改一个变量, 进行自增操作时, 由于线程的工作线程写回主存时会有覆盖问题, 就像是'丢失了几次写操作'一样。

(线程工作内存和主存等相关问题请查阅JMM, 这里不再过多描述)

这里就是一个多线程同时修改counter的**错误**使用方式:
```java
public class WrongCounter {

    private static Integer counter = 0;

    public static void main(String[] args) throws InterruptedException {

        Runnable function = () -> {
            for (var i = 0; i < 100; i++) {
                System.out.println(counter++);
            }
        };

        for (var i = 0; i < 40; i++) {
            new Thread(function).start();
        }


        Thread.sleep(2000);
        System.out.printf("final counter: %d \n", counter);
    }
}
/*  output:
    ...
    ...
	3990
	3991
	3992
	3993
	3994
	final counter: 3995 
 */
```

最终会看到counter只加到了3995, 而没有加到预期的4000

### 使用cas完成计数案例
我们把上面的错误案例修改一下, 借助atomic类的cas的来完成多线程计数功能:
```java
import java.util.concurrent.atomic.AtomicInteger;

public class CounterCAS {
    public static void main(String[] args) throws InterruptedException {
        var counter = new AtomicInteger();

        Runnable function = () -> {
            for (var i = 0; i < 100; i++) {
                var current = counter.getAndIncrement();
                System.out.println(current);
            }
        };

        for (var i = 0; i < 40; i++) {
            new Thread(function).start();
        }


        Thread.sleep(2000);
        System.out.printf("final counter: %d \n", counter.get());
    }
}

/* output:
    ...
    ...
    3996
    3997
    3998
    3999
    final counter: 4000 
 */
```
可以看到最终输出的是4000, 复合我们的预期。

### cas的优点
1. 线程安全
1. 乐观锁思想, 不用抢锁。没有竞争到的线程也不用挂起/切换上下文。低并发的时候有很好的表现

### cas的缺点
1. 不适用于高并发场景, 会造成过多的自旋, 使cpu空转
1. 会产生ABA问题

## Atomic类的getAndIncrement()的实现原理
在 `java.util.concurrent.atomic.AtomicInteger` 里getAndIncrement()方法实现如下:
```java
    private static final Unsafe U = Unsafe.getUnsafe();

    private static final long VALUE
        = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;

    /**
     * Atomically increments the current value,
     * with memory effects as specified by {@link VarHandle#getAndAdd}.
     *
     * <p>Equivalent to {@code getAndAdd(1)}.
     *
     * @return the previous value
     */
    public final int getAndIncrement() {
        return U.getAndAddInt(this, VALUE, 1);
    }
```
发现是调用了jdk.internal.misc.Unsafe.getAndAddInt()方法实现的。参数是this, VALUE, 还有1。

其中1很好理解, 就是让计数器counter的值每次自增1的意思。

VALUE可以理解为value变量在AtomicInteger对象里的偏移量, 可以通过this+VALUE来定位到value这个变量, 以便直接对内存进行操作。

getAndAddInt()方法内部实现如下:
```java
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }

    public native int getIntVolatile(Object o, long offset);

    public final boolean weakCompareAndSetInt(Object o, long offset,
                                              int expected,
                                              int x) {
        return compareAndSetInt(o, offset, expected, x);
    }

    public final native boolean compareAndSetInt(Object o, long offset,
                                                 int expected,
                                                 int x);
```
其中的`o`就是刚才传入的 AtomicInteger对象的`this`。
其中的`weakCompareAndSetInt`最终是调用的native方法`compareAndSetInt`。

发现里面的逻辑很简单, 就是一个do-while循环:
通过`this和offset`从内存里拿到了当前的value值, 记为`v`。
然后通过while循环不断地调用`compareAndSetInt`, 尝试进行**比较再替换**。


## cas 的实现原理

## cas的源码探索之旅(java -> c++ -> 汇编)



## 参考文章
1. [【并发编程】CAS与FAA](https://blog.csdn.net/zhoufanyang_china/article/details/89488959)
2. [Double compare-and-swap](https://en.wikipedia.org/wiki/Double_compare-and-swap)
3. [x86系统cache locking的原理](https://cloud.tencent.com/developer/article/1634000)

https://coolshell.cn/articles/11454.html

https://blog.csdn.net/qq_35642036/article/details/82801708

https://albk.tech/%E8%81%8A%E8%81%8ACPU%E7%9A%84LOCK%E6%8C%87%E4%BB%A4.html

https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf

https://www.zhihu.com/question/65372648

https://hjlarry.github.io/docs/go/lock/