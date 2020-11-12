---
layout: post
title: 这就是TLS系列之一(简介)
categories: [系统技术, 网络技术]
description: some word here
keywords: keyword1, keyword2
---

TLS， 为Transport Layer Security的缩写， 用于在两个通信应用程序之间提供保密性和数据完整性。  这里面涉及到几点事情， 一是数据的传输格式， 这个是通过 TLS Record记录层来提供的， 另外的是数据传输之前的身份确认， 这个是TLS Handshake层来提供的， 大体的， 传输层通过 X.509协议认证， 利用非对称加密深处来对通信方做身份认证； 然后是数据的传输， 先是通过交换对称密钥作为session key， 再用这个密钥来加密数据， 从而保证 两个 应用通信的保密性和可靠性， 使得通信过程不被攻击者窃听。 

TLS及SSL(安全套接字Secure Sockets Layer)的关系为SSL为TLS的前身， 都是用于保障通信的安全及数据完整性。

