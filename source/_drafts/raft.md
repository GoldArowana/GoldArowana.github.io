---
title: raft协议
date: 2021-02-02 12:09:44
summary: raft协议学习与论文解读
tags:
    - raft
    - consistency
    - distribute
categories:
    - distribute
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co15-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co15.jpg
---

## Raft简介
复制日志问题: 是复制状态机的一种抽象, 把具有一定顺序的一系列action抽象成一条日志(log), 每个action都是日志中的一个条目(entry)。
如果想让每个节点的服务状态相同, 则要把日志中的所有entry按照记录顺序执行一遍。
所以复制状态机的核心问题就变成了让每个节点都具有相同的日志的问题, 也就是把日志复制到每个节点上的问题。

### 组成成员
- leader
- follower

## Raft三大部分
### leader选举
#### 任期
任期主要解决一下两个问题

问题1: 所有follower同事开始争leader。
问题2: 脑裂。


### 日志复制
1. leader收到客户端请求后, 将entry记录追加到末尾(append操作, 即插入到当前index处)。每追加一个entry, index就会加1。
2. leader之后会向所有follower发送AppendEntries请求。
3. leader收到大多数follower的成功请求后, 这个entry就被leader认为达到提交状态了。leader将这个entry应用到状态机中。



### 安全性

#### 一致性检查

#### 不提交旧的leader的entry

### 角色改变

## 参考文章
1. [寻找一种易于理解的一致性算法（扩展版）](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)
2. [go夜读-etcd raft 源码阅读](https://talkgo.org/t/topic/51)
3. [go夜读-通过 hashicorp/raft 库手把手调试 raft 算法](https://talkgo.org/t/topic/882)
4. [对Raft的理解](https://zhuanlan.zhihu.com/p/55070003)
5. [线性一致性：什么是线性一致性？](https://zhuanlan.zhihu.com/p/42239873)

https://www.luozhiyun.com/archives/287

https://raft.github.io/raft.pdf

https://disksing.com/even-node-raft/

https://www.youtube.com/watch?v=UzzcUS2OHqo

https://www.youtube.com/watch?v=64Zp3tzNbpE

https://www.youtube.com/watch?v=4r8Mz3MMivY

https://www.cnblogs.com/linbingdong/p/6442673.html

https://www.youtube.com/watch?v=fcFqFfsAlSQ

https://zhuanlan.zhihu.com/p/262192992

https://www.youtube.com/watch?v=g4epAvtzDYA

https://github.com/wenweihu86/raft-java

https://mendylee.gitbooks.io/geeker-study-courses/content/fen-bu-shi-jia-gou-jing-dian-tu-shu-he-lun-wen-ff08-fen-bu-shi-jia-gou-ff09.html

https://draveness.me/consensus/

http://thesecretlivesofdata.com/raft/