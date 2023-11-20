---
title: GC算法
date: 2021-04-07 11:05:59
summary: GC算法
tags: gc
categories: gc
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co148-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co148.jpg
---

评价GC算法的性能时，我们采用以下4个标准。
- 吞吐量
- 最大暂停时间
- 堆使用效率
- 访问的局部性


<p class="note note-primary">标签</p>


### mutator
mutator是Dijkstra琢磨出来的词，有“改变某物”的意思。说到要改变什么，那就是GC对象间的引用关系。不过光这么说可能大家还是不能理解，其实用一句话概括的话，它的实体就是“应用程序”。这样说就容易理解了吧。GC就是在这个mutator内部精神饱满地工作着。
mutator实际进行的操作有以下2种：
- 生成对象
- 更新指针

mutator在进行这些操作时，会同时为应用程序的用户进行一些处理（数值计算、浏览网页、编辑文章等）。随着这些处理的逐步推进，对象间的引用关系也会“改变”。伴随这些变化会产生垃圾，而负责回收这些垃圾的机制就是GC。



## 朴素的标记清除
### 标记
```java
01   mark_phase(){
02     for(r : $roots)
03        mark(*r)
04   }
05
06
07   mark(obj){
08     if(obj.mark == FALSE)
09        obj.mark = TRUE
10        for(child : children(obj))
11          mark(*child)
12   }
```

深搜？广搜？
两者性能差不多, 但是深搜比广搜更节省内存。

### 清除
```java
1   sweep_phase(){
2     sweeping = $heap_start
3     while(sweeping < $heap_end)
4        if(sweeping.mark == TRUE)
5          sweeping.mark = FALSE
6        else
7          sweeping.next = $free_list
8          $free_list = sweeping
9        sweeping += sweeping.size
 10   }
```
### 分配
```java
1   new_obj(size){
2     chunk = pickup_chunk(size, $free_list)
3     if(chunk ! = NULL)
4        return chunk
5     else
6        allocation_fail()
7   }
```
#### First-fit、Best-fit、Worst-fit的不同
之前我们讲的分配策略叫作First-fit。因为在pickup_chunk()函数中，最初发现大于等于size的分块时就会立即返回该分块。然而，分配策略不止这些。还有遍历空闲链表，返回大于等于size的最小分块，这种策略叫作Best-fit。还有一种策略叫作Worst-fit，即找出空闲链表中最大的分块，将其分割成mutator申请的大小和分割后剩余的大小，目的是将分割后剩余的分块最大化。但因为Worst-fit很容易生成大量小的分块，所以不推荐大家使用此方法。除去Worst-fit，剩下的还有Best-fit和First-fit这两种。当我们使用单纯的空闲链表时，考虑到分配所需的时间，选择使用First-fit更为明智。

### 合并
改造sweep_phase, 在清楚阶段进行合并
```java
01   sweep_phase(){
02     sweeping = $heap_start
03     while(sweeping < $heap_end)
04        if(sweeping.mark == TRUE)
05          sweeping.mark = FALSE
06        else
07          if(sweeping == $free_list + $free_list.size)
08             $free_list.size += sweeping.size
09          else
10             sweeping.next = $free_list
11             $free_list = sweeping
12        sweeping += sweeping.size
13   }
```

### 总结

#### 优点 
- 实现简单: 说到GC标记-清除算法的优点，那当然要数算法简单，实现容易了。打个比方，接下来我们将在第3章中提到引用计数法，在引用计数法中就很难切实管理计数器的增减，实现也很困难。另外，如果算法实现简单，那么它与其他算法的组合也就相应地简单。在第3章和第4章中，我们会为大家介绍把GC标记-清除算法与其他GC算法相结合的方法。
- 与保守式GC算法兼容：在第6章中介绍的保守式GC算法中，对象是不能被移动的。因此保守式GC算法跟把对象从现在的场所复制到其他场所的GC复制算法（第4章）与标记-压缩算法（第5章）不兼容。而GC标记-清除算法因为不会移动对象，所以非常适合搭配保守式GC算法。事实上，在很多采用保守式GC算法的处理程序中也用到了GC标记-清除算法。

#### 缺点
- 碎片化
- 分配速度慢
- 与写时复制技术不兼容


### 可以考虑使用多个空闲链表优化

### BiBOP法

### 位图标记

### 延迟清除法

### 三色标记法



## 引用计数法

### 普通实现

### 优点

### 缺点

### 延迟引用计数法

### Sticky引用计数法

### 1位引用计数法

### 部分标记-清除法

#### 四色

## 参考文章
1. 《垃圾回收的算法与实现-中村成洋》
1. [Tracing garbage collection](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Tri-color_marking)

