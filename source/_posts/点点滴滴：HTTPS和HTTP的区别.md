---
title: 点点滴滴：HTTPS和HTTP的区别
date: 2016-08-27 21:17:04
tags: 点点滴滴
---

从iOS9开始，苹果官方开始要求，上线的APP需要对HTTPs协议进行支持，虽然我们可以通过在项目的plist文件中设置属性的方式，暂时绕开这条限制， 但是我个人认为还是有必要了解下，苹果为什么要这么做？
<!--more-->
先简单科普下HTTP与HTTPS分别代表什么，以及它们的区别到底是什么？
### [HTPPS和HTTP的概念](#1)

HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。这个系统的最初研发由网景公司进行，提供了身份验证与加密通讯方法，现在它被广泛用于万维网上安全敏感的通讯，例如交易支付方面。

超文本传输协议 (HTTP-Hypertext transfer protocol) 是一种详细规定了浏览器和万维网服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

### [HTTPS和HTTP的区别：](#1)

1.HTTPS议需要到CA申请证书，一般免费证书很少，需要交费。HTTP是超文本传输协议，不需要证书
2.端口号不同，HTTPS端口号是443，HTTP端口号是80
3.HTTPS要进行多次的身份验证，6/7的握手以达到数据安全，性能消耗较大，也即通信之前需要先确认身份，只有身份确认才会发送信息。

看完上面HTTPS和HTTP的不同之后，我们不难发现其实二者最根本的区别就是在HTTPS比HTTP多使用的SSL层，那SSL层又是什么东东呢？且看下面这张图
![](http://ock9zbzms.bkt.clouddn.com/643.png)
公式：HTTP+SSL/TLS+TCP = HTTPS

[那么SSL、TLS又是什么鬼呢？](#2)
简单的立即SSL就是对传输的内容通过某种算法进行加密的一种协议，那TLS又是什么呢？TLS实际上是SSL的升级版，看下下面的换算公式：
SSL 2.0
SSL 3.0
TLS 1.0 (SSL 3.1)
TLS 1.1 (SSL 3.1)
TLS 1.2 (SSL 3.1)

目前，应用最广泛的是TLS 1.0，接下来是SSL 3.0。但是，主流浏览器都已经实现了TLS 1.2的支持。

那么身为移动端开发工程师，面对如此大势所趋，我们能做什么呢？
方法一：要求公司的服务端进行升级，至少支持到苹果要求的TLS1.2
方法二：在info.plist中进行配置，使APP能够访问不支持HTTPS的服务器。
如果让你的APP能够访问任意的Host地址那么可以这样配置（这是最省事也是最不安全的）

```mm
NSAppTransportSecurity    
    NSAllowsArbitraryLoads

```