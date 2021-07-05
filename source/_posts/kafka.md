---
title: kafka
date: 2021-11-02 13:54:45
summary: kafka原理学习
tags:
    - kafka
    - MQ
categories:
    - MQ
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co129-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co129.jpg
---

##  消息分区机制
- 轮循策略(未指定key的默认策略)
- 随机策略(老版本)
- 根据key计算hash(指定key时的默认策略)

## 参考资料
1. 《极客时间-Kafka核心技术与实战》
1. 《极客时间-Kafka核心源码解读》