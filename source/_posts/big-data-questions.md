---
title: 大数据面试问题
date: 2020-01-10 21:48:36
summary: 收集了一些大数据面试问题
tags:
    - big data
categories:
    - big data
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co73.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co73.jpg
---

## 一、黑名单网页过滤
### 题目
实通过判断网页url是否在黑名单里, 现一个黑名单网页过滤系统。
每个网页的url最多占用64B。

### 要求 
1. 允许有万分之一以下的判断失误率。
1. 使用的额外空间不要超过30GB

### 解析 
#### 思路1: 哈希表(不可行)
如果通过哈希表保存, 那么64B * 100亿 > 640GB, 显然不符合要求。
#### 思路2: 布隆过滤器
可以使用布隆过滤器。

## 二、在20亿个整数中找到出现次数最多的数
### 题目
有一个包含20亿个全是32位整数的大文件, 在其中找到出现次数最多的数。

### 要求
1. 内存限制为2GB

### 解析
#### 思路1: 直接使用哈希表(不可行)
如果用哈希表做词频统计, key需要占用4B, value需要占用4B,  那么可能最多需要的空间是: (4B+4B) * 20亿 > 16GB, 显然这种方法不满足要求
#### 思路2: 分片后使用哈希表
可以通过哈希函数, 将20亿个数分成16个小文件。每个小文件对应的数不会大于2亿种, 用哈希表计算2亿种数的词频大约需要1.6GB内存。
接下来只需要选出这16个小文件各自的最高词频即可。

## 三、40亿个非负整数中找到未出现的数
### 题目
32位无符号整数的范围是 0 ~ 4294967295, 请在一个有40亿个无符号整数的文件中, 找出未出现过的数。
### 要求 
1. 最多使用1GB内存

### 解析
#### 思路1: 直接使用哈希表(不可行)
如果直接使用哈希表, 那么需要使用内存: 4B * 40亿 ≈ 16GB, 显然是不满足条件的。
#### 思路2: bitmap
4294967295个bit数组占用500MB空间, 可以解决这个问题。

## 四、在40亿个非负整数中找出1个未出现过的数
### 题目
32位无符号整数的范围是 0 ~ 4294967295, 请在一个有40亿个无符号整数的文件中, 找出1个未出现过的数
### 要求 
1. 最多使用10MB内存

### 解析
#### 思路: 分片统计, 然后用bitmap
将0 ~ 4294967295划分为64个区间, 统计每个区间的数字种类。如果哪个区间的总数不到 (4294967296/64)个, 那么说明在这个区间可以找出未出现过的数。
在对应的区间内使用bitset寻找未出现的数, 4294967296/64 = 67108864 bit ≈ 8MB

## 五、40亿个非负整数中找到出现两次的数
### 题目
32位无符号整数的范围是 0 ~ 4294967295, 现在有40亿个无符号整数, 找出所有出现了2次的数。 
### 要求 
1. 最多使用1GB的内存

### 解析
#### 思路: bitmap变种
普通的bitmap值只有0和1, 在这道题下, 我们让bitmap的值支持0~3。所以申请的空间是4294967295 * 2bit = 1GB。
遍历所有文件, bitmap值为0表示未出现过, 1表示出现过1次, 2表示出现过2次, 3表示出现次数多于2次。

## 六、40亿个非负整数中找到中位数
### 题目
32位无符号整数的范围是 0 ~ 4294967295, 现在有40亿个无符号整数, 找出中位数。
### 要求
1. 最多使用10MB的内存

### 解析
#### 思路: 文件分流
(2 * 1000 * 1000) * 32bit ≈ 8MB , 所以我们将每 2000000 个数字划为一个区间。
4294967296/ 2000000 向上取整为2148个区间。
遍历一遍文件, 统计每个区间的总词频。
之后可以定位到中位数在具体的一个区间内, 再次遍历文件, 不在这个区间的数字忽略不计, 统计这个区间的每个数的词频, 找出中位数。

[大数据求职必看：经典的大数据面试问题 )](https://www.sohu.com/a/138204769_236714)
[程序员代码面试指南 第6章]()

http://duanple.com/?p=289