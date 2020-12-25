---
title: paxos协议
date: 2021-02-01 12:10:55
summary: 据说是最难懂的一致性协议?
tags:
    - paxos
    - consistency
    - distribute
categories:
    - distribute
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co18-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co18.jpg
---

## Basic Paxos
### 基本角色
- proposer
- acceptor
- learner

### 提议编号
每个提议都包含一个提议编号, 并且这个提议编号是唯一且递增的。

这里提供一种参考思路: 每个进行都会分配一个唯一的进程标识(processid, 32位), 每个进程都维护一个计数器(counter, 32位)。那么提议编号可以设计为 counter<<32 + processid


### 选择提案

### 活锁

### 学习值

### distinguished learner

## Multi Paxos

### 独立实例运行的完整paxos算法

#### 脑裂处理

#### 空洞处理

### 只运行一次prepare消息的完整paxos算法

#### 脑裂处理

## 复制状态机

### 空洞处理

## 原子广播

## 参考文章
1. [拜占庭将军问题](http://duanple.com/?p=174)
1. [The Part-Time Parliament](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf)
1. [Paxos算法《The Part-Time Parliament》译文](https://blog.csdn.net/zhaoshouyue/article/details/92787920)
1. [Paxos Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)
1. [Paxos Made Simple 译文](https://www.jianshu.com/p/67dd80555ba2)
1. [Fast Paxos](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2005-112.pdf)

https://zhuanlan.zhihu.com/p/343225569

https://disksing.com/paxos/

https://lamport.azurewebsites.net/pubs/paxos-simple.pdf

https://zhuanlan.zhihu.com/p/343225836

http://www.read.seas.harvard.edu/~kohler/class/08w-dsi/chandra07paxos.pdf

https://zhuanlan.zhihu.com/p/343226028

https://zhuanlan.zhihu.com/p/343225670

https://zhuanlan.zhihu.com/p/343225248

http://duanple.com/?p=164

http://linbingdong.com/2017/11/21/PhxPaxos%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94%E2%80%94Paxos%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/

https://www.cnblogs.com/linbingdong/p/6253479.html

https://www.zhihu.com/question/19787937

https://www.youtube.com/watch?v=JEpsBg0AO6o

https://www.youtube.com/watch?v=d7nAGI_NZPk

https://www.youtube.com/watch?v=fcFqFfsAlSQ

https://www.youtube.com/watch?v=-Bl5GleEN5s

https://www.youtube.com/watch?v=uBQSE4MMWhY

https://www.youtube.com/watch?v=JEpsBg0AO6o

https://www.youtube.com/playlist?list=PLkLazW-A4v3KEuYn-rW_19OEnPmn0Gq3v

https://www.youtube.com/watch?v=bkWL4mtiVbs

https://www.youtube.com/watch?v=vmwnhZmEZjc

https://blog.openacid.com/algo/paxos/

https://drmingdrmer.github.io/algo/2020/10/18/quorum.html

https://github.com/kr/paxos

https://github.com/openacid/paxoskv

https://lrita.github.io/2018/10/23/safety-and-liveness-in-distributed/

https://zhuanlan.zhihu.com/p/21438357?refer=lynncui

https://zhuanlan.zhihu.com/p/21466932

http://oceanbase.org.cn/?p=90

http://oceanbase.org.cn/?p=111

http://oceanbase.org.cn/?p=160

http://oceanbase.org.cn/?p=136

https://en.wikipedia.org/wiki/Paxos_(computer_science)#Multi-Paxos

https://mp.weixin.qq.com/s?__biz=MjM5MDg2NjIyMA==&mid=203607654&idx=1&sn=bfe71374fbca7ec5adf31bd3500ab95a&key=8ea74966bf01cfb6684dc066454e04bb5194d780db67f87b55480b52800238c2dfae323218ee8645f0c094e607ea7e6f&ascene=1&uin=MjA1MDk3Njk1&devicetype=webwx&version=70000001&pass_ticket=2ivcW%2FcENyzkz%2FGjIaPDdMzzf%2Bberd36%2FR3FYecikmo%3D

https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=403582309&idx=1&sn=80c006f4e84a8af35dc8e9654f018ace&scene=1&srcid=0119gtt2MOru0Jz4DHA3Rzqy&key=710a5d99946419d927f6d5cd845dc9a72ff3d652a8e66f0ddf87d91262fd262f61f63660690d2d5da76a44a29e155610&ascene=0&uin=MjA1MDk3Njk1&devicetype=iMac+MacBookPro11%2C4+OSX+OSX+10.11.1+build(15B42)&version=11020201&pass_ticket=bhstP11nRHvorVXvQ4pt9fzB9Vdzj5sSRBe84783gsg%3D

https://github.com/hedengcheng/tech/blob/master/distributed/PaxosRaft%20%E5%88%86%E5%B8%83%E5%BC%8F%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90%E5%8F%8A%E5%85%B6%E5%9C%A8%E5%AE%9E%E6%88%98%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8.pdf

https://github.com/oldratlee/translations/tree/master/paxoslease

https://github.com/oldratlee/translations/tree/master/paxos-made-simple