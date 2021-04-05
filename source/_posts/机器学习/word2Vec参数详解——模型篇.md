---
title: word2Vec参数详解——模型篇
date: 2020-03-11 09:04:52
tags: NLP
---

这篇文章是学习Xin Rong的《word2vec Parameter Learning Explained》记录的学习笔记，主要介绍了word2Vec的CBOW模型和Skip-gram模型，以及其模型涉及的推导过程。

<!--more-->

## 1.Continuous Bag-Of-Word Model

### 1.1 one-word context

首先介绍最简单的CBOW模型，假设每个上下文只考虑一个单词，即模型在一个给定的上下文单词的基础上预测一个单词，类似于一个双连词模型。

图1给出了简化上下文环境下的网络模型，在我们的环境中，词汇表的大小为V，隐藏层的大小为N，邻近的两个层是全连接的。输入是one-hot向量，即对于一个给定的上下文单词，对于长度为V的向量$\{x_1,...x_v\}$，只有该单词对应的$x_k$是1，其余都是0。

![图一 上下文只有一个单词的简单的CBOW模型](http://img.wanghaojun.cn//img/20200306094756.png)

<center>图1 上下文只有一个单词的简单的CBOW模型</center>
输入层和输出层之间的权重向量可以表示为一个$V \times N$的向量$W$，每一行$W$都是一个N维的表示向量$v_w$，正式的来说，W的第i行就是$v_w^T$，给定一个上下文（一个单词），假设$x_k=1,\quad x_{k'}=0 \quad for \quad k' \neq k$,那么有
$$
\mathbf{h}=\mathbf{W}^{T} \mathbf{x}=\mathbf{W}_{(k,)}^{T}:=\mathbf{v}_{w_{I}}^{T}
\tag{1}
$$
本质上来说，是将W的第K行复制给h，$v_{w_I}$就是输入单词$w_I$的向量表示。这意味着隐含层单元的激活函数是简单的线性函数(即直接将其加权的输入和传递给下一层)。

对于隐藏层到输出层，有一个不同的权重矩阵$W'=\{w'_{ij}\}$，这是一个$N \times V$的矩阵，使用这些权重，可以计算词汇表中每个单词的分数$u_j$，
$$
u_j={v'_{w_j}}^T h
\tag{2}
$$
$v'_{w_j}$是矩阵$W'$的第j列，然后使用softmax，一个对数线性分类模型，得到单词的后验分布，这是一个多项分布。
$$
p\left(w_{j} | w_{I}\right)=y_{j}=\frac{\exp \left(u_{j}\right)}{\sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right)}
\tag{3}
$$
$y_j$是输出层中第j个单元的输出。
$$
p\left(w_{j} | w_{I}\right)=\frac{\exp \left({\mathbf{v}'_{w_j}}^T\mathbf{v}_{w_{I}}\right)}{\sum_{j^{\prime}=1}^{V} \exp \left({\mathbf{v}'_{w_{j'}}}^T \mathbf{v}_{w_{I}}\right)}
\tag{4}
$$
注意，$v_w$和$v_w'$都是单词w的表示，$v_w$来自于W的行，这是从输入层到隐藏层的权重矩阵；$v_w'$来自于$W'$的列，这是从隐藏层到输出层的权重矩阵，在接下来的分析中，我们把$v_w$叫做w的“输入向量”，$v_w'$叫做w的“输出向量”。

**从隐藏层到输出层的权重更新公式**

虽然实际的计算是不切实际的(下面会解释)，但进行推导是为了在不使用任何技巧的情况下深入了解这个原始模型。

训练的目的是使实际的输出单词$w_O$的条件概率最大化，在给定输入的上下文单词$w_I$的情况下。
$$
\begin{aligned} \max p\left(w_{O} | w_{I}\right)  &= \max y_{j^{*}} \\ &=\max \log y_{j^{*}} \\ &=u_{j^{*}}-\log \sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right):=-E \end{aligned}
\tag{5}
$$
$E=-logp(w_O|w_I)$是损失函数（希望最小化E）,j*是输出层实际输出单词的索引，注意，这个损失函数可以理解为两个概率分布之间交叉熵的一个特例。

现在，让我们推导隐藏层和输出层之间的权值的更新方程。对E对第j个单元的输入$u_j$求导，得到
$$
\frac{\partial E}{\partial u_{j}}=y_{j}-t_{j}:=e_{j}
\tag{6}
$$
$t_j=1(j=j^*)$，当第j个单元是真实的输出单词时，$t_j=1$,否则$t_j=0$，注意这只是输出层的预测误差$e_j$。接下来，对$w_{ij}'$求导来得到从隐藏层到输出层的权重。
$$
\frac{\partial E}{\partial w_{i j}^{\prime}}=\frac{\partial E}{\partial u_{j}} \cdot \frac{\partial u_{j}}{\partial w_{i j}^{\prime}}=e_{j} \cdot h_{i}
\tag{7}
$$
因此，利用随机梯度下降法，我们得到了隐式的权值更新方程。
$$
w_{i j}^{\prime}(\mathrm{new})=w_{i j}^{\prime}(\mathrm{old})-\eta \cdot e_{j} \cdot h_{i}\tag{8}
$$
或者
$$
\mathbf{v}_{w_{j}}^{\prime}(\text { new })=\mathbf{v}_{w_{j}}^{\prime}(\text { old })-\eta \cdot e_{j} \cdot \mathbf{h} \quad \text { for } j=1,2, \cdots, V\tag{9}
$$
$\eta>0$是学习率，$e_j=y_j-t_j$,$h_i$是第i个隐藏层，$v_{w_j}'$是$w_j$的输出向量。注意，这个更新方程意味着我们必须遍历词汇表中所有可能的单词，检查它的输出可能性$y_j$，然后比较$y_j$与其渴望的输出$t_j（0 || 1）$。如果$y_j>t_j$（高估）,那么需要按比例的从$v_{w_j}'$中减去h，可以使$v_{w_j}'$远离$v_{w_I}$;如果$y_j>t_j$（低估，只有当$t_j=1,i.e.,w_j=w_o$的时候是正确的），将h按比例加到$v_{w_o}'$上，使其接近$v_{w_I}$。如果$y_j$接近$t_j$，那么就导致权重更新变得非常小。

**从输入层到隐藏层的权重更新公式**

把E对隐含层的输出求导，得到
$$
\frac{\partial E}{\partial h_{i}}=\sum_{j=1}^{V} \frac{\partial E}{\partial u_{j}} \cdot \frac{\partial u_{j}}{\partial h_{i}}=\sum_{j=1}^{V} e_{j} \cdot w_{i j}^{\prime}:=\mathrm{EH}_{i}
\tag{10}
$$
$h_i$是隐藏层第i个单元的输出。$u_j$在公式（2）中定义，是输出层第j个单元的输入，$e_j=y_j-t_j$是输出层第j个单词的预测误差。$EH$，一个N维的向量，是词汇表中所有单词的输出向量之和，根据预测误差加权。

接下来，求E关于W的导数，回想一下，隐含层对输入层的值执行线性计算，展开公式（1）的向量符号，可以得到：
$$
h_{i}=\sum_{k=1}^{V} x_{k} \cdot w_{k i}
\tag{11}
$$
然后我们可以计算E关于W中某个元素的导数：
$$
\frac{\partial E}{\partial w_{k i}}=\frac{\partial E}{\partial h_{i}} \cdot \frac{\partial h_{i}}{\partial w_{k i}}=\mathrm{EH}_{i} \cdot x_{k}
\tag{12}
$$
这等价于向量x和EH的张量积：
$$
\frac{\partial E}{\partial \mathbf{W}}=\mathbf{x} \otimes \mathrm{EH}=\mathbf{x} \mathrm{EH}^{T}
\tag{13}
$$
由于向量x中只有一个元素非0，所以这个式子只有一行不为0，其值为$EH^T$，一个N维的向量。于是，W的更新公式为:
$$
\mathbf{v}_{w_{I}}^{(\text {new })}=\mathbf{v}_{w_{I}}^{(\text {old })}-\eta \mathrm{EH}^{T}\tag{14}
$$
$v_{w_I}$是W中的一行，是唯一的一个上下文单词的“输入向量”，也是W中唯一导数不为0的一行，W中的其余行的值都保持不变，因为导数为0。

直观的来看，因为向量$EH$是词汇表中所有单词的输出向量的和，由预测误差$e_j=y_j-t_j$来加权，我们可以把公式（14）理解为将词汇表中每个单词的输出向量的一部分加到上下文向量的输入向量上去。如果，在输出层，一个单词$w_j$是输出单词的概率被高估了（$y_j > t_j $），那么上下文单词$w_I$的输入向量就会远离$w_j$的输出向量；相反的，如果$w_j$是输出单词的概率被低估了（$y_j < t_j $），那么输入向量$w_I$就会更加接近输出向量$w_j$，如果$w_j$的概率被准确的预测了，那么它只会对输入向量$w_I$产生细微的影响。$w_I$的输入向量的移动是由词汇表中所有向量的预测误差决定的，预测误差越大，单词对上下文单词输入向量的影响越明显。

当我们通过训练语料库生成的上下文-目标词对迭代地更新模型参数时，对向量的影响将会累积。我们可以想象一个单词w的输出向量被与w由共同成分的邻居的输入向量来来回回的“拖动”，就好像w的向量和它的邻居的向量之间有物理弦一样。类似地，一个输入向量也可以被认为是被许多输出向量拖拽的。这种解释可以让我们想起重力，或者力指向的图形布局。每个虚拟字符串的平衡长度与相关单词对之间的相似程度以及学习速度有关。经过多次迭代，输入和输出向量的相对位置将最终稳定下来。

### 1.2 Multi-word context

接下来介绍使用多个单词作为上下文的CBOW模型。当计算隐藏层的输出的时候，不是直接复制输入上下文单词的输入向量，CBOW模型计算岭输入上下文单词的向量平均值，然后使用输入层到隐藏层的权重向量和平均向量的产物作为输出。
$$
\begin{aligned}
\mathbf{h} &=\frac{1}{C} \mathbf{W}^{T}\left(\mathbf{x}_{1}+\mathbf{x}_{2}+\cdots+\mathbf{x}_{C}\right) \\
&=\frac{1}{C}\left(\mathbf{v}_{w_{1}}+\mathbf{v}_{w_{2}}+\cdots+\mathbf{v}_{w_{C}}\right)^{T}
\end{aligned}
\tag{15}
$$
C是上下文单词的数目，$w_1,...,w_2$是上下文单词，$v_w$是单词$w$的输入向量。损失函数是：
$$
\begin{aligned}
E&= =-\log p\left(w_{O} | w_{I, 1}, \cdots, w_{I, C}\right)\\
&=-u_{j^{*}}+\log \sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right)\\
&=-{\mathbf{v}_{w_{O}}^{'}}^T \cdot \mathbf{h} + \log \sum_{j'=1}^{V} \exp \left({\mathbf{v}_{w_{j}}^{\prime}}^{T} \cdot \mathbf{h}\right)
\end{aligned}
\tag{16}
$$
如下图所示

![](http://img.wanghaojun.cn//img/20200310095823.png)

<center>图2 连续的词袋模型</center>
从隐藏层到输出层的权重更新公式与单个上下文单词的模型是一样的，把公式(8)复制下来：
$$
w_{i j}^{\prime}(\mathrm{new})=w_{i j}^{\prime}(\mathrm{old})-\eta \cdot e_{j} \cdot h_{i}\tag{17}
$$
注意，对于每一次训练，都要把这个公式应用到从隐藏层到输出层的向量矩阵的每个元素上。

从输入层到隐藏层的权重更新公式类似于公式(14)，除了现在需要把下面这个公式应用到上下文中的每个$w_{I,c}$上。
$$
\mathbf{v}_{w_{I,c}}^{(\text {new })}=\mathbf{v}_{w_{I,c}}^{(\text {old })}-\frac{1}{C} \eta \mathrm{EH}^{T}\tag{18}
$$
$v_{w_{I,c}}$是上下文单词的第c个输入向量，$\eta$是正向学习率，$EH$可以从公式(10)得出。这个更新公式的直接理解是和公式(14)一样的。

## 2 skip-Gram Model

图3展示了skip-gram模型的过程，这是CBOW模型相反的一种方式，目标词是输入层，而上下文单词在输出层。

![](http://img.wanghaojun.cn//img/20200310154042.png)

<center>图3 skip-gram模型</center>
这里仍然使用$v_{w_I}$作为输入层唯一的一个单词向量，因此隐藏层的计算过程和公式(1)是一样的，即$\mathbf{h}$只是从输入层到隐藏层的权重矩阵W的与输入单词$w_I$有关的一行的简单的复制。
$$
\mathbf{h}=\mathbf{W}^{T} \mathbf{x}=\mathbf{W}_{(k,)}^{T}:=\mathbf{v}_{w_{I}}^{T}
\tag{19}
$$
在输出层，这里不是输出一个多项分布，而是输出C个多项分布，每个输出的计算过程使用相同的隐藏层到输出层的矩阵：

$$
p\left(w_{c, j}=w_{O, c} | w_{I}\right)=y_{c, j}=\frac{\exp \left(u_{c, j}\right)}{\sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right)}
\tag{20}
$$
$w_{c,j}$是输出层第c组的第j个单词，$w_{O,c}$是上下文中第c个实际输出的单词，$w_I$是唯一的输入向量；$y_{c,j}$是第c组第j个单元的输出；$u_{c,j}$是输出层第c组第j个单元的输入。因为输出层共享相同的权重，因此

$$
u_{c, j}=u_{j}={\mathbf{v}_{w_{j}}^{\prime}}^{T} \cdot \mathbf{h}, \text { for } c=1,2, \cdots, C\tag{21}
$$
$\mathbf{v}_{w_{j}}^{\prime}$是词汇表中第j个单词$w_j $的输出向量，也是隐藏层到输出层的向量矩阵$W'$的一列。

参数更新方程的推导与单词上下文模型的推导区别不是很大。损失函数变成了：
$$
\begin{aligned}
E &=-\log p\left(w_{O, 1}, w_{O, 2}, \cdots, w_{O, C} | w_{I}\right) \\
&=-\log \prod_{c=1}^{C} \frac{\exp \left(u_{c, j_{c}^{*}}\right)}{\sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right)} \\
&=-\sum_{c=1}^{C} u_{j_{c}^{*}}+C \cdot \log \sum_{j^{\prime}=1}^{V} \exp \left(u_{j^{\prime}}\right)
\end{aligned}
\tag{22}
$$
$j_c^*$是上下文单词的第c个单词在词汇表中的索引。

计算E关于每组输出层的每个单元的输入的导数，可以得到：
$$
\frac{\partial E}{\partial u_{c, j}}=y_{c, j}-t_{c, j}:=e_{c, j}\tag{23}
$$
这是每个单元的预测误差，类似于公式(6)，这里定义了一个维度为V的向量$EI={EI_1,...EI_V}$作为所有上下文单词的预测误差之和。
$$
EI_j=\sum_{c=1}^{C}e_{c,j}\tag{24}
$$
接下来，计算E关于隐藏层到输出层的矩阵$\mathbf{W'}$的导数，得到：
$$
\frac{\partial E}{\partial w_{i j}^{\prime}}=\sum_{c=1}^{C} \frac{\partial E}{\partial u_{c, j}} \cdot \frac{\partial u_{c, j}}{\partial w_{i j}^{\prime}}=\mathrm{EI}_{j} \cdot h_{i}\tag{25}
$$
因此，得到了从隐藏层到输出层的矩阵$\mathbf{W'}$的更新公式：
$$
\boldsymbol{w}_{i j}^{\prime}(\text { new })=w_{i j}^{\prime}(\text { old })-\eta \cdot \mathrm{EI}_{j} \cdot h_{i}\tag{26}
$$
或者：
$$
\mathbf{v}_{w_{j}}^{\prime}(\text { new })=\mathbf{v}_{w_{j}}^{\prime}(\text { old })-\eta \cdot EI_{j} \cdot \mathbf{h} \quad \text { for } j=1,2, \cdots, V\tag{27}
$$
这个公示的直观理解类似于公式(9)，唯一不同的是，预测误差是对输出层中所有上下文单词的总和。注意，对于每一次训练，都要把这个公式应用到从隐藏层到输出层的向量矩阵的每个元素上。

从输入层到隐藏层的矩阵的更新公式的推导类似于公式(10)和(14)，不同的是，需要把预测误差$e_j$替换为$EI_j$,可以直接给出更新公式：
$$
\mathbf{v}_{w_{I}}^{(\text {new })}=\mathbf{v}_{w_{I}}^{(\text {old })}-\eta \mathrm{EH}^{T}\tag{28}
$$
EH是一个N维的向量，每个元素定义为：
$$
EH_i = \sum_{j=1}^{V}EI_j \cdot w'_{ij}\tag{29}
$$
公式(28)的理解与公式(14)是一样的。























