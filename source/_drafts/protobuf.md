---
title: protobuf基础知识
date: 2021-01-08 18:39:17
summary: 学习一下Google设计的二进制序列化方案
tags:
    - protobuf
    - rpc
    - grpc
    - distribute
categories:
    - distribute
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co83.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co83.jpg
---

## 基本原理
> Protocol Buffer是Google提供的跨语言、跨平台的二进制序列化方案，可用于数据存储、数据交换等场景，扩展性非常高。

## 压缩

### varint

### zigzag
> `zigzag`编码的出现是为了解决`varint`对负数编码效率低的问题。`zigzag`编码的原理非常简单，就是将有符号整数映射为无符号整数。在实现上，映射通过移位即可实现，而不需要使用映射表来存储。

## 参考文章
1. 《深入理解序列化与反序列化》
1. [详解varint编码原理](https://segmentfault.com/a/1190000020500985?utm_source=tag-newest)
