---
title: golang atomic cas
date: 2021-12-22 13:26:15
summary: golang并发必学-原子操作cas
categories:
    - golang
tags:
    - 并发
    - atomic
    - cas
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co193-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co193.jpg
---

## 使用

## 原理
Go / src / runtime / internal / atomic / asm_amd.s文件中
```nasm
TEXT runtime∕internal∕atomic·Cas64(SB), NOSPLIT, $0-25
 MOVQ ptr+0(FP), BX
 MOVQ old+8(FP), AX
 MOVQ new+16(FP), CX
 LOCK
 CMPXCHGQ CX, 0(BX)
 SETEQ ret+24(FP)
 RET

```


## 参考文章