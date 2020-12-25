---
title: http
date: 2020-07-11 18:40:09
summary: 学习一下http协议家族
categories: 
    - internet
tags:
    - internet
    - http
    - ssl/tls
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co10.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co10.jpg
---

## OSI网络分层模型
第一层：物理层，网络的物理形式，例如电缆、光纤、网卡、集线器等等；
第二层：数据链路层，它基本相当于 TCP/IP 的链接层；
第三层：网络层，相当于 TCP/IP 里的网际层；
第四层：传输层，相当于 TCP/IP 里的传输层；
第五层：会话层，维护网络中的连接状态，即保持会话和同步；
第六层：表示层，把数据转换为合适、可理解的语法和语义；
第七层：应用层，面向具体的应用传输数据。

## 状态码
1××：提示信息，表示目前是协议处理的中间状态，还需要后续的操作；
2××：成功，报文已经收到并被正确处理；
3××：重定向，资源位置发生变动，需要客户端重新发送请求；
4××：客户端错误，请求报文有误，服务器无法处理；
5××：服务器错误，服务器在处理请求时内部发生了错误。

## http0.9
> HTTP 是基于 TCP/IP 协议的应用层协议。它不涉及数据包（packet）传输，主要规定了客户端和服务器之间的通信格式，默认使用80端口。

### 请求
> 最早版本是1991年发布的0.9版。该版本极其简单，只有一个命令GET。
```http
GET /index.html
```

### 响应
> 协议规定，服务器只能回应HTML格式的字符串，不能回应别的格式。
```html
<html>
  <body>Hello World</body>
</html>
```
> 服务器发送完毕，就关闭TCP连接。

## http1.0
> 引入了POST命令和HEAD命令
### 请求
每次通信都必须包括头信息（HTTP header），用来描述一些元数据。
```http
GET / HTTP/1.0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
Accept: */*
```

### 响应
1. 响应第一行是"协议版本 + 状态码（status code） + 状态描述"。
2. 接下来是"头信息 + 以一个空行结尾表示"头信息部分结束"（\r\n）
3. 最后是"数据"部分

```http
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```
### 缺点
> HTTP/1.0 版的主要缺点是，每个TCP连接只能发送一个请求。发送数据完毕，连接就关闭，如果还要请求其他资源，就必须再新建一个连接。

> TCP连接的新建成本很高，因为需要客户端和服务器三次握手，并且开始时发送速率较慢（slow start）。所以，HTTP 1.0版本的性能比较差。

> 为了解决这个问题，有些浏览器在请求时，用了一个非标准的Connection字段。

## http1.1
### 新增命令
1.1版还新增了许多动词方法：PUT、PATCH、HEAD、 OPTIONS、DELETE。

另外，客户端请求的头信息新增了Host字段，用来指定服务器的域名。

### 持久连接
> 1.1 版的最大变化，就是引入了持久连接（persistent connection），即TCP连接默认不关闭，可以被多个请求复用，不用声明Connection: keep-alive。

> 客户端和服务器发现对方一段时间没有活动，就可以主动关闭连接。不过，规范的做法是，客户端在最后一个请求时，发送Connection: close，明确要求服务器关闭TCP连接。

```http
Connection: close
```

> 目前，对于同一个域名，大多数浏览器允许同时建立6个持久连接。
### 管道机制
> 1.1 版还引入了管道机制（pipelining），即在同一个TCP连接里面，客户端可以同时发送多个请求。这样就进一步改进了HTTP协议的效率。

> 管道机制: 在同一个tcp连接中, 在等待上一个请求响应的同时，发送下一个请求。但是服务器还是按照顺序，先回应A请求，完成后再回应B请求。

### Content-Length字段
> 一个TCP连接现在可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是Content-length字段的作用，声明本次回应的数据长度。

```http
Content-Length: 3495
```
> 上面代码告诉浏览器，本次回应的长度是3495个字节，后面的字节就属于下一个回应了。

> 在1.0版中，Content-Length字段不是必需的，因为浏览器发现服务器关闭了TCP连接，就表明收到的数据包已经全了。

### 分块传输编码
```http
Transfer-Encoding: chunked
```

### 缺点
> 虽然1.1版允许复用TCP连接，但是同一个TCP连接里面，所有的数据通信是按次序进行的。服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。这称为"队头堵塞"（Head-of-line blocking）。

> 为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。这导致了很多的网页优化技巧，比如合并脚本和样式表、将图片嵌入CSS代码、域名分片（domain sharding）等等。如果HTTP协议设计得更好一些，这些额外的工作是可以避免的。

## https
> 不使用SSL/TLS的HTTP通信，就是不加密的通信。所有信息明文传播，带来了三大风险。
> 1. 窃听风险（eavesdropping）：第三方可以获知通信内容。
> 2. 篡改风险（tampering）：第三方可以修改通信内容。
> 3. 冒充风险（pretending）：第三方可以冒充他人身份参与通信。
### SSL/TLS
> SSL/TLS协议的基本思路是采用公钥加密法，也就是说，客户端先向服务器端索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

#### 如何保证公钥不被篡改？
> 解决方法：将公钥放在数字证书中。只要证书是可信的，公钥就是可信的。

#### 公钥加密计算量太大，如何减少耗用的时间？
解决方法: 用非对称加密来传输对称加密的秘钥

#### SSL/TLS基本过程
1. 客户端向服务器端索要并验证公钥。
2. 双方协商生成"对话密钥"。
3. 双方采用"对话密钥"进行加密通信。


## http2

### 二进制化协议
> HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制，并且统称为"帧"（frame）：头信息帧和数据帧。

> 二进制协议的一个好处是，可以定义额外的帧。如果使用文本实现这种功能，解析数据将会变得非常麻烦，二进制解析则方便得多。

### 多工
> HTTP/2 复用TCP连接，在一个连接里，客户端和浏览器都可以同时发送多个请求或回应，而且不用按照顺序一一对应，这样就避免了"队头堵塞"。

> HTTP/2 将每个请求或回应的所有数据包，称为一个数据流（stream）。每个数据流都有一个独一无二的编号。数据包发送的时候，都必须标记数据流ID，用来区分它属于哪个数据流。另外还规定，客户端发出的数据流，ID一律为奇数，服务器发出的，ID为偶数。

### 取消数据流
> 数据流发送到一半的时候，客户端和服务器都可以发送信号（RST_STREAM帧），取消这个数据流。1.1版取消数据流的唯一方法，就是关闭TCP连接。这就是说，HTTP/2 可以取消某一次请求，同时保证TCP连接还打开着，可以被其他请求使用。

### 定义优先级
> 客户端还可以指定数据流的优先级。优先级越高，服务器就会越早回应。

### 头信息压缩
> HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如Cookie和User Agent，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。

> HTTP/2 对这一点做了优化，引入了头信息压缩机制（header compression）。一方面，头信息使用gzip或compress压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就提高速度了。

#### HPACK

### 服务器推送
> HTTP/2 允许服务器未经请求，主动向客户端发送资源，这叫做服务器推送（server push）。

> 常见场景是客户端请求一个网页，这个网页里面包含很多静态资源。正常情况下，客户端必须收到网页后，解析HTML源码，发现有静态资源，再发出静态资源请求。其实，服务器可以预期到客户端请求网页后，很可能会再请求静态资源，所以就主动把这些静态资源随着网页一起发给客户端了。

## http3


## 参考文章
1. 《极客时间-透视HTTP协议》
2. [全解网络协议](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E5%85%A8%E8%A7%A3%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE)
3. [HTTP 协议入门](http://www.ruanyifeng.com/blog/2016/08/http.html)
4. [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
5. [HTTP的前世今生](https://coolshell.cn/articles/19840.html)
6. [为什么 HTTPS 需要 7 次握手以及 9 倍时延](https://draveness.me/whys-the-design-https-latency/)
7. [小林coding-20 张图解，为什么 HTTP3.0 使用 UDP 协议？](https://my.oschina.net/u/4482993/blog/4631144)

https://imququ.com/post/series.html

https://coolshell.cn/articles/19840.html

https://zhuanlan.zhihu.com/p/68024390

https://zhuanlan.zhihu.com/p/89471776

https://imququ.com/post/series.html

https://httpwg.org/specs/rfc7541.html

https://httpwg.org/specs/rfc7540.html

https://zhuanlan.zhihu.com/p/143464334

https://quicwg.org/base-drafts/draft-ietf-quic-http.html

https://www.chromium.org/quic

https://zhuanlan.zhihu.com/p/40595473

https://tools.ietf.org/html/rfc5246

