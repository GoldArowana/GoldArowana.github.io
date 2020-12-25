---
title: b树
date: 2021-09-15 13:26:40
summary: b树数据结构及应用
tags:
    - algorithm
    - data structure
    - database
categories:
    - data structure
img:  https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co21-m.jpg
tinyImg:  https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co21.jpg
---

## 实现
### MongoDB
> 1. 由于 B 树的非叶结点也可以存储数据，所以查询一条数据所需要的平均随机 IO 次数会比 B+ 树少，使用 B 树的 MongoDB 在类似场景中的查询速度就会比 MySQL 快。
> 1. 读多写少的场景下, B树 比 LSM-tree 更合适。

## 参考文章
1. 《Modern B-Tree Techniques》
2. [为什么 MongoDB 使用 B 树](https://blog.csdn.net/weixin_32666923/article/details/113452107)
3. [为什么MongoDB使用B树](https://blog.csdn.net/weixin_41987908/article/details/105255119)
4. [WiredTiger存储引擎之一：基础数据结构分析](https://mongoing.com/topic/archives-35143)
5. [B树](https://blog.csdn.net/u010916338/article/details/86134334)
6. [为什么MongoDB使用B-Tree,Mysql使用B+Tree ?](https://juejin.cn/post/6844904136576598029)