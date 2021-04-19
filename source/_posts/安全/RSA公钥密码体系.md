---
title: RSA公钥密码体系
date: 2020-09-15 14:48:22
tags: [密码学]
---

RSA公钥密码体制是1978年由麻省理工学院的三位密码学家Rivest、Shamir和Adleman提出的一种用数论构造的，也是迄今为止最为成熟完善的公钥密码体制。<!--more-->

为了进行RSA系统的初始化，即生成RSA的公私钥密码对，需要进行以下几个步骤：

1. 选取两个不同的大素数$p$和$q$。
2. 计算$n=pq$，$\varphi(n)=(p-1)(q-1)$，其中$\varphi(n)$是$n$的欧拉函数。
3. 随机选取整数$e<Z,1<e<\varphi(n)$作为公钥，要求满足$(e,\varphi(n))=1$。
4. 采用欧几里得算法计算私钥$d$，使$ed=1(mod \varphi(n))$，即$d=e^{-1}(mod \varphi(n))$，则$e$和$n$是公钥，$d$是私钥。

注：$e$和$n$是公开的，在系统初始化成功以后两个素数$p$、$q$和$\varphi(n)$可以销毁，但不能泄露。

**1.加密过程：**

RSA的加密函数为$E(m)=m^e(mod n)$，$\forall m  \in M$，$M$为明文空间，计算$c=m^e(mod n)$，$c$就是密文。

**2.解密过程：**

接收方收到密文$c$之后，利用RSA的解密函数$D(c)=c^d(mod n)$，即$m=c^d(mod n)$。

<img src="http://img.wanghaojun.cn//img/20200915151346.png" style="zoom:50%;" />