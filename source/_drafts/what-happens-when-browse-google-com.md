---
title: 访问google.com的时候都发生了什么
summary: 梳理一下网络协议知识
date: 2021-01-04 00:45:25
categories: 
    - internet
tags:
    - internet
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co2.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co2.jpg
---


## 键盘


## 浏览器


## url

## ISO 7层网络模型
### 应用层
网络服务与最终用户的一个接口。
协议有：HTTP FTP TFTP SMTP SNMP DNS TELNET HTTPS POP3 DHCP
### 表示层
数据的表示、安全、压缩。（在五层模型里面已经合并到了应用层）
格式有，JPEG、ASCll、EBCDIC、加密格式等
### 会话层
建立、管理、终止会话。（在五层模型里面已经合并到了应用层）
对应主机进程，指本地主机与远程主机正在进行的会话
### 传输层
定义传输数据的协议端口号，以及流控和差错校验。
协议有：TCP UDP，数据包一旦离开网卡即进入网络传输层
### 网络层
进行逻辑地址寻址，实现不同网络之间的路径选择。
协议有：ICMP IGMP IP（IPV4 IPV6）
### 数据链路层
建立逻辑连接、进行硬件地址寻址、差错校验 等功能。（由底层网络定义协议）
将比特组合成字节进而组合成帧，用MAC地址访问介质，错误发现但不能纠正。
### 物理层
建立、维护、断开物理连接。（由底层网络定义协议）

## DNS

## UDP

## IP

## TCP



## 参考文章
1. [有了 IP 地址，为什么还要用 MAC 地址？](https://www.zhihu.com/question/21546408)

https://draveness.me/whys-the-design-non-unique-mac-address/

https://draveness.me/whys-the-design-ipv6-replacing-ipv4/