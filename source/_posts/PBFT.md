---
title: PBFT
date: 2021-07-07 11:05:59
summary: 实用拜占庭容错算法(PBFT, Practical Byzantine Fault Tolerance)
tags:
  - consistency
  - distribute
categories:
  - distribute
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co190-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co190.jpg
---


## 两将军问题
> 根据FLP不可能原理，两将军问题无通用解

## 拜占庭将军问题
> 拜占庭系统普遍采用的假设条件包括：
> 1. 拜占庭节点的行为可以是任意的，拜占庭节点之间可以共谋；
> 2. 节点之间的错误是不相关的；
> 3. 节点之间通过异步网络连接，网络中的消息可能丢失、乱序并延时到达，但大部分协议假设消息在有限的时间里能传达到目的地；
> 4. 服务器之间传递的信息，第三方可以嗅探到，但是不能篡改、伪造信息的内容和验证信息的完整性。

## BFT(拜占庭容错)简介

## PBFT简介
> PBFT算法可以在失效节点不超过总数1/3的情况下同时保证Safety和Liveness

> PBFT 算法采用密码学相关技术(RSA 签名算法、消息验证编码和摘要)确保消息传递过程无法被篡改和破坏

> PBFT是一种状态机副本复制算法
## SBFT简介
Hyperledger fabric v1.0对PBFT的改进

## 优点
> PBFT算法具有高交易通量和吞吐量，高可用性，易于理解。

## 缺点
> 1. 计算效率依赖于参与协议的节点数量，由于每个副本节点都需要和其它节点进行P2P的共识同步，因此随着节点的增多，性能会下降的很快，但在较少节点的情况下可以有不错的性能，并且分叉的几率很低，不适用于节点数量过大的区块链，扩展性差。
> 2. 系统节点是固定的，无法应对公有链的开放环境，只适用于联盟链或私
有链环境。
> 3. PBFT算法要求总节点数n>=3f+1(其中，f代表作恶节点数)。系统的失效节点数量不得超过全网节点的1/3，容错率相对较低。

## 总结
> PBFT算法由于每个副本节点都需要和其他节点进行P2P的共识同步，因此随着节点的增多，性能会下降的很快，但是在较少节点的情况下可以有不错的性能，并且分叉的几率很低。PBFT主要用于联盟链，但是如果能够结合类似DPOS这样的节点代表选举规则的话也可以应用于公联，并且可以在一个不可信的网络里解决拜占庭容错问题，TPS应该是远大于POW的。

## 参考文章


https://blog.csdn.net/jfkidear/article/details/81275974

https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98

https://zhuanlan.zhihu.com/p/43067427

https://lrita.github.io/2018/10/28/time-sequence-in-blockchain-by-bft/

http://pmg.csail.mit.edu/papers/osdi99.pdf

https://www.jianshu.com/p/78e2b3d3af62

《极客时间-分布式协议与算法实战》