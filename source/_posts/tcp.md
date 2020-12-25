---
title: tcp
date: 2020-03-05 18:40:03
summary: 学习一下tcp网络协议
categories:
    - internet
tags:
    - internet
    - tcp
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co9-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co9.jpg
---

## 连接
### 三次握手
![三次握手](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/tcp/tcp-con.jpg)

### 四次挥手
![四次挥手](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/tcp/tcp-close-con.jpg)

## 重传
### 超时重传
> 重传机制的其中一个方式，就是在发送数据时，设定一个定时器，当超过指定的时间后，没有收到对方的 ACK 确认应答报文，就会重发该数据，也就是我们常说的超时重传。

### 快速重传
> 快速重传的工作方式是当收到三个相同的 ACK 报文时，会在定时器过期之前，重传丢失的报文段。

> 快速重传机制只解决了一个问题，就是超时时间的问题，但是它依然面临着另外一个问题。就是重传的时候，是重传之前的一个，还是重传所有的问题。

### SACK重传机制(Selective Acknowledgment 选择性确认)
> 这种方式需要在 TCP 头部「选项」字段里加一个 SACK 的东西，它可以将缓存的地图发送给发送方，这样发送方就可以知道哪些数据收到了，哪些数据没收到，知道了这些信息，就可以只重传丢失的数据。

### D-SACK
> D-SACK 有这么几个好处： 
> 1. 可以让「发送方」知道，是发出去的包丢了，还是接收方回应的 ACK 包丢了;
> 2. 可以知道是不是「发送方」的数据包被网络延迟了;
> 3. 可以知道网络中是不是把「发送方」的数据包给复制了;

## 滑动窗口

## 拥塞控制
> 拥塞窗口 cwnd 变化的规则：
> 1. 只要网络中没有出现拥塞，cwnd 就会增大；
> 1. 但网络中出现了拥塞，cwnd 就减少；

### 慢启动
> 为什么需要慢启动?
> 
> 发送端和接收端在连接建立之初，谁也不知道可用带宽是多少，因此需要一个估算机制，然后还要根据网络中不断变化的条件而动态改变速度。此时，根据交换数据来估算客户端与服务器之间的可用带宽是唯一的方法，而且这也是慢启动算法的设计思路。


### 拥塞避免算法
当拥塞窗口 cwnd 「超过」慢启动门限 ssthresh 就会进入拥塞避免算法。
一般来说 ssthresh 的大小是 65535 字节。

### 拥塞发生
当发生了「超时重传」，则就会使用拥塞发生算法。

这个时候，ssthresh 和 cwnd 的值会发生变化：tcp
- ssthresh 设为 cwnd/2
- cwnd 重置为 1

![拥塞发生](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/tcp/tcp-hp01.jpg)

### 快速恢复
> 快速重传和快速恢复算法一般同时使用，快速恢复算法是认为，你还能收到 3 个重复 ACK 说明网络也不那么糟糕，所以没有必要像 RTO 超时那么强烈。

进入快速恢复之前，cwnd 和 ssthresh 已被更新了：
- cwnd = cwnd/2 ，也就是设置为原来的一半;
- ssthresh = cwnd;

然后，进入快速恢复算法如下：
- 拥塞窗口 cwnd = ssthresh + 3 （ 3 的意思是确认有 3 个数据包被收到了）；
- 重传丢失的数据包；
- 如果再收到重复的 ACK，那么 cwnd 增加 1；
- 如果收到新数据的 ACK 后，把 cwnd 设置为第一步中的 ssthresh 的值，原因是该 ACK 确认了新的数据，说明从 duplicated ACK 时的数据都已收到，该恢复过程已经结束，可以回到恢复之前的状态了，也即再次进入拥塞避免状态；

![快速恢复](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/tcp/tcp-hp02.jpg)


## 参考文章
1. [全解网络协议](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E5%85%A8%E8%A7%A3%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE)
2. [小林coding-30 张图解： 面试必问的 TCP 重传、滑动窗口、流量控制、拥塞控制](https://blog.csdn.net/qq_34827674/article/details/105606205)
3. [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
4. [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
5. [UDP 协议：UDP 和 TCP 相比快在哪里？](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E9%87%8D%E5%AD%A6%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F-%E5%AE%8C/34%20%20UDP%20%E5%8D%8F%E8%AE%AE%EF%BC%9AUDP%20%E5%92%8C%20TCP%20%E7%9B%B8%E6%AF%94%E5%BF%AB%E5%9C%A8%E5%93%AA%E9%87%8C%EF%BC%9F.md)
6. [TCP 为什么是三次握手，而不是两次或四次？](https://www.zhihu.com/question/24853633/answer/573627478)
7. [Linux系列：TCP慢启动原理（拥塞控制机制）](https://www.huaweicloud.com/articles/8357193.html)
8. [“三次握手，四次挥手”你真的懂吗？](https://zhuanlan.zhihu.com/p/53374516)
9. [传输层(Transport Layer)](https://chenxfeng.github.io/2017/05/10/computer_network/computer_network_3%EF%BC%9A%E4%BC%A0%E8%BE%93%E5%B1%82(Transport%20Layer)/)

http://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html

https://www.ruanyifeng.com/blog/2012/06/internet_protocol_suite_part_ii.html

http://www.ruanyifeng.com/blog/2009/03/tcp-ip_model.html

http://www.ruanyifeng.com/blog/2017/06/tcp-protocol.html

https://datatracker.ietf.org/doc/rfc1644/

http://www.tcpipguide.com/free/t_TCPSlidingWindowAcknowledgmentSystemForDataTranspo-6.htm

https://www.youtube.com/playlist?list=PLoCMsyE1cvdWKsLVyf6cPwCLDIZnOj0NS

https://draveness.me/whys-the-design-tcp-three-way-handshake/

https://draveness.me/whys-the-design-tcp-performance/

https://draveness.me/whys-the-design-tcp-segment-ip-packet/

https://draveness.me/whys-the-design-tcp-message-frame/

https://draveness.me/whys-the-design-tcp-time-wait/

https://draveness.me/whys-the-design-tcp-message-frame/

