---
title: web开发安全
date: 2021-10-25 11:05:59
summary: web开发安全, sql注入、xss攻击、水平越权、垂直越权、
tags:
categories:
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co191-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co191.jpg
---

## 水平越权

## 垂直越权

## XSS攻击

### 存储型
攻击者将恶意代码提交到目标网站的数据库中

### 反射性
攻击者构造出特殊的 URL，其中包含恶意代码, 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器

### DOM型
攻击者构造出特殊的 URL，其中包含恶意代码。用户打开带有恶意代码的 URL。 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行

## CSRF攻击(跨站请求伪造)

## SQL注入
### 基于布尔的盲注
### 基于时间的盲注
### 基于报错的注入
### 联合查询注入
### 堆查询注入


## 参考文章

1. [前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)
2. [反射型XSS漏洞详解](http://www.ttlsa.com/safe/xss-description/)
3. [前端安全系列（二）：如何防止CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)
4. [HMAC Timing Attacks](http://www.eggie5.com/45-hmac-timing-attacks)
5. [HMAC Timing Attacks](https://coolshell.cn/articles/21003.html)
6. [SQL注入漏洞详解](https://www.cnblogs.com/jazzka702/p/11875728.html)
7. [Mysql分页负数sql攻击](https://www.kancloud.cn/digest/mysqlsummary/132837)