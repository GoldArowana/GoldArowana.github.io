---
title: thrift
date: 2022-01-02 18:39:45
summary: 学习一下Facebook开源的thrift
tags:
    - thrift
    - rpc
    - distribute
categories:
    - distribute
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co8-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co8.jpg
---


## 基本原理

## TProtocol

### TBinaryProtocol

### TCompactProtocol

### TJSONProtocol

### TSimpleJSONProtocol

### TTupleProtocol

## TTransport
> Thrift使用了Transport来封装传输层，但Transport不仅仅是底层网络传输，它还是上层流的封装。
### TNonblockingSocket

### TFramedTransport

### TFastFramedTransport

### TServerTransport

## TServer
> Thrift采用了TServer来作为服务器的抽象，提供了多种类型的服务器实现。用TServerTransport作为服务器的Acceptor抽象，来监听端口，创建客户端Socket连接

### TNonblockingServer

### THsHaServer

### TThreadSelectorServer

## 压缩

### varint

### zigzag
> `zigzag`编码的出现是为了解决`varint`对负数编码效率低的问题。`zigzag`编码的原理非常简单，就是将有符号整数映射为无符号整数。在实现上，映射通过移位即可实现，而不需要使用映射表来存储。

## 参考文章
1. [zigzag算法详解](https://blog.csdn.net/weixin_43708622/article/details/111397290)
2. [iter_zc-Thrift源码分析](https://www.kancloud.cn/digest/thrift/118984)
3. [详解varint编码原理](https://segmentfault.com/a/1190000020500985?utm_source=tag-newest)
4. 《深入理解序列化与反序列化》
5. [Apache Thrift系列详解(二) - 网络服务模型](https://juejin.cn/post/6844903622384287751)
6. [Thrift源码分析](https://www.kancloud.cn/digest/thrift/118983)