---
title: golang map实现
date: 2021-05-01 18:49:49
summary: 每门语言必看的一个数据结构-map
tags:
    - golang
    - map
categories:
    - golang
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co51-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co51.jpg
---

## map内部数据结构
![map内部结构-整体](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/golang-map/map-view.jpg)

### hmap
```golang
// A header for a Go map.
type hmap struct {
	// 元素个数，调用 len(map) 时，直接返回此值
	count int
	flags uint8
	
	// buckets 的对数 log_2
	B uint8
	
	// overflow 的 bucket 近似数
	noverflow uint16
	
	// 计算 key 的哈希的时候会传入哈希函数
	hash0 uint32
	
	// 指向 buckets 数组，大小为 2^B
	// 如果元素个数为0，就为 nil
	buckets unsafe.Pointer
	
	// 扩容的时候，buckets 长度会是 oldbuckets 的两倍
	oldbuckets unsafe.Pointer
	
	// 指示扩容进度，小于此地址的 buckets 迁移完成
	nevacuate uintptr
	extra     *mapextra
	
	// optional fields
}
```

### buckets
```golang
type bmap struct {
    tophash [bucketCnt]uint8
}
```

### bmap
```golang
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow  uintptr
}
```

## get操作

## 扩容过程
### 增量扩容
当负载过多时, 会触发增量扩容
### 原地整理
当overflow过多时, 会触发原地整理迁移
### 渐进式迁移
每次最多前一两个bucket

## 遍历
### 普通遍历
### 渐进式迁移过程中的map遍历

## 其他问题
### key 可以是 float 型吗

## 参考文章
1. [Go map实现原理](https://my.oschina.net/renhc/blog/2208417)
2. [深度解密Go语言之map](https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA)
3. [draveness-golang-map](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/)
4. [你不知道的Golang map](https://www.cnblogs.com/sunsky303/p/11815172.html)

https://juejin.cn/post/6954707500151078919

https://segmentfault.com/a/1190000023879178

https://www.cnblogs.com/33debug/p/11851585.html

https://github.com/cch123/golang-notes/blob/master/map.md

https://lukechampine.com/hackmap.html

https://juejin.cn/post/6844903848587296781

https://en.wikipedia.org/wiki/Associative_array

https://cloud.tencent.com/developer/article/1422446