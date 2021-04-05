---
title: logistic回归
date: 2019-04-05 11:29:21
tags: deeplearning
---
logistic回归是一种用于监督式学习的算法，它的输出y总是0或者是1，logistic回归的目标是尽可能减少预测值和训练数据之间的差别。 <!--more-->
举例：Cat vs No-cat
给出一张由特征向量x表示的图片，该算法会预测图片中有一只猫的可能性。
$$Given   x,\hat{y}=P(y=1|x),where \quad 0 \le \hat{y} \le 1$$
logistic回归中的参数如下：
* 输入特征向量：$x \in R^{n_x}$ ，$n_x$是指特征的数量
* 训练标签：$y \in 0,1$
* 权重：$w \in  R^{n_x} $，$n_x$是指特征的数量
* 偏移量：$b \in R$
* 输出：$\hat{y}=\sigma(w^T x+b)$
* sigmoid函数：$s=\sigma(w^T x+b)=\sigma(z)=\frac{1}{1+e^{-z}}$
![](http://img.wanghaojun.cn/img/20190401210938.png)
$(w^T x+b)$是一个线性函数$(ax+b)$，但是我们需要的是一个介于[0,1]之间的值，所以使用了sigmoid函数，该函数的函数值范围是[0,1]。
从图中可以发现：
* 当z是一个很大的正数时，$\sigma(z)=1$
* 当z是一个绝对值很大的负数时，$\sigma(z)=0$
* 如果z=0，$\sigma(z)=0.5$
### logistic回归的损失函数
为了训练参数w和b，我们需要一个损失函数。
$$\hat{y}^{(i)}=\sigma(w^T x^{(i)}+b),where\quad\sigma(z^{(i)})=\frac{1}{1+e^{-z^{(i)}}}$$
$Given[(x^{(1)},y^{(1)}),(x^{(2)},y^{(2)}),...,(x^{(i)},y^{(i)})]$ 我们希望$\hat{y}^{(i)}\approx{y}^{(i)}$
损失函数(Loss function)是为了计算预测值$\hat{y}^{(i)}$和期望值${y}^{(i)}$之间的差别，对于单个的训练样本，损失函数有如下形式：
$$L(\hat{y}^{(i)},{y}^{(i)})=-[{y}^{(i)}log(\hat{y}^{(i)})+(1-{y}^{(i)})log(1-\hat{y}^{(i)})]$$
* 当${y}^{(i)}=1:L(\hat{y}^{(i)},{y}^{(i)})=-log(\hat{y}^{(i)})$,当$\hat{y}^{(i)}$接近1时，L接近0。
* 当${y}^{(i)}=0:L(\hat{y}^{(i)},{y}^{(i)})=-log(1-\hat{y}^{(i)})$,当$\hat{y}^{(i)}$接近0时，L接近0。

成本函数（Cost function）是损失函数在整个训练集的体现，我们希望获得最好的w和b来最小化成本函数。
$$J(w,b)=\frac{1}{m}\sum_{i=1}^{m}L(\hat{y}^{(i)},{y}^{(i)})=-\frac{1}{m}\sum_{i=1}^{m}[{y}^{(i)}log(\hat{y}^{(i)})+(1-{y}^{(i)})log(1-\hat{y}^{(i)})]$$
### logistic回归的梯度下降
假设输入x是两维的，所以w也是两维的那么此时的正向计算单个样本的损失函数如下：
![](http://img.wanghaojun.cn/img/20190402154324.png)
反向过程如下：
![](http://img.wanghaojun.cn/img/20190402154556.png)
计算导数时：
* $da=\frac{dL(a,y)}{da}=-\frac{a}{y}+\frac{1-y}{1-a}$
* $ dz=\frac{dL(a,y)}{dz}=a-y$

以下是计算整个样本集代码：
![](http://img.wanghaojun.cn/img/20190403105227.png)
### logistic回归的代码实现
使用向量化去代替循环，可以极大的提高程序的运行速度，因为向量化的运算使用了CPU或者GPU的SIMD（全称Single Instruction Multiple Data，单指令多数据流），SIMD在GPU上的表现也要优于CPU。所以尽量在程序中避免使用循环。
使用向量化简化以上正向传播和反向传播的过程如下：

    import numpy as np
    s = 1 # 学习率
    m = 10 # 样本集大小
    w = np.zeros(10) # 权值
    w = w.reshape(10,1)
    X = np.random.randn(10,10) # 输入
    b = 0   # 截距
    Y = np.array([1,0,1,0,1,1,1,0,1,0]) # 输出
    for item in range(10):
            Z = np.dot(w.T,X) + b
            A = 1.0/(1.0 + np.exp(-Z)) # 预测值
            L = -(np.dot(Y,np.log(A).T))+np.dot((1-Y),np.log(1-A).T) #Loss function
            dZ = A - Y        
            dw = np.dot(X,dZ.T) / m 
            db = np.sum(dZ) / m
            w -= s * dw   
            b -= s * db
            print(str(item) + ":" + str(L))
    print(w)
    print(b)

![](http://img.wanghaojun.cn/img/20190403170048.png)