---
title: ElGamal公钥密码体系
date: 2020-09-15 15:19:21
tags: 密码学
---

1985年ElGamal提出ElGamal公钥密码体系，它基于离散对数问题（Discrete logarithm problem，DLP）且主要为数字签名的目的而设计，是继RSA之后最著名的数字签名方案。<!--more-->

ElGamal的公私钥对生成过程如下：

参数生成：设$G$为有限域$Z_p^{\*}$上的乘法群，$p$是一个素数，$g$是$Z_p^\*$上的一个生成元。

密钥生成：选取$a\in(1,p-1)$，计算$\beta=g^a mod p$,那么得到的私钥为a，公钥为$(p,g,\beta)$。

加密过程：

对于加密消息$m$可以任意的选取随机数$k\in(1,p-1)$，计算$\gamma=g^k mod p$和$\delta=m\beta^k mod p$，可以得到密文$c=(\gamma,\delta)$。

解密过程：

接收者收到密文$c=(\gamma,\delta)$后，使用私钥$a$，计算$\gamma^{-a}\delta=(g^k)^{-a}\delta=m \ mod \ p = m$，可以得到相对应的明文$m$。

![](http://img.wanghaojun.cn//img/20200915153209.png)