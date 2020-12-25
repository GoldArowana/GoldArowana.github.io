---
title: MQTT
date: 2021-09-03 11:05:59
summary: MQTT物联网协议
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co169-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co169.jpg
---

## QoS
### QoS0
> 最多一次送达。也就是发出去就fire掉，没有后面的事情了。

### QoS1
> QoS 1 也好理解，我发一个带messageid的消息出去，对方收到了，给我回一个带messageid的ack，我才认为数据收到了。

### QoS2
![QoS2的交互过程](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/mqtt/qos2-detail.jpg)

#### 故障处理
- publish下发失败了，服务器重发publish。
- pubrec上报失败了，服务器重发publish。这个时候，客户端仍然是重复收到多次publish。
- pubrel下发失败了，服务器重发pubrel。
- pubcomp上报失败了，服务器重发pubrel。

## 参考文章
1. [MQTT协议QoS2 准确一次送达的实现](https://blog.csdn.net/zerooffdate/article/details/78950907)
2. [MQTT协议中文版](https://mcxiaoke.gitbooks.io/mqtt-cn/content/)