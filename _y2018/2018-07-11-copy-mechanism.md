---
layout: post
title: "ACL_2016_Incorporating Copying Mechanism in Sequence-to-Sequence Learning"
subtitle: ""
date: 2018-07-11
author: "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
  - COPY

---

## 1. 写在前面的话

Copying Mechanism 的出现缓解了 NLP 的一个难题，即 OOV 问题。什么是 OOV 就是通常情况下我们的词表大小是确定的，但是测试集中有可能会出现词表中没有陌生词，之前的处理方式就是直接用 UNK 的符号表示这一类 OOV，因为通常情况下 OOV 不会非常多，但是你要想再提高比如 1%的准确率，那么这个 Copying Mechanism 你不可或缺。Copying Mechanism 能解决例如旅行者背包的问题。整体来看，现在这个 Copying Mechanism 是十分常用的一种手段，例如在对话系统中，sequence-to-sequence 的这样一个 encode-decode 框架下，decode 预测的词是除了词表的分布外结合对话上文出现词的这样一个概率和。借用 manning 的这篇 Key-Value Retrieval Networks for Task-Oriented Dialogue 表示一下这个分布结果：

![Distribution](/img/Copy-mechanism/Distribution.png)

可以看到同时又词表的分布与上文的分布，对于两者重叠的词将会有更大的概率。

## 2. 简单的介绍一下 attention 机制

Attention 机制你可以理解成一个 query 在 memory 中查找跟它相似的地方并给予加强。而一个 memory 你可以理解成是 key 与 value 构成的，这样想的话，attention 其实就是在寻找 query 与 key 的匹配程度并且加强这部分的 value 权重。通式可以表示成这样:

$$att=softmax(F(Query,Key))*Value$$

上述公式中的 F 可以是常用的比如一个 DNN 等。我们看一下 encode-decode 中的 context vector 是怎么计算的：

假设$s_{t-1}$代表上一轮 decode 的隐藏层，$h_{\tau}$ 代表 encode 的第$\tau$此时的隐藏那么可以得到对于每个 encode 隐藏层的这样一个权重计算公式：

$$\alpha_{t\tau}=\frac{e^{\eta(s_{t-1},h_{\tau})}}{\sum_{\tau^i}^{ }e^{\eta(s_{t-1},h_{\tau^{i}})}}$$

这样我们就可以得到当前的一个 context vector 用$c_t$表示如下：

$$c_t=\sum_{\tau=1}^{T_s}\alpha_{t\tau}h_{\tau}$$

话不多说，直接进入模型。

## 2. Model

![Model](/img/Copy-mechanism/Model.PNG)

### 2.1 Encoder

Encode 部分比较简单就是一个 Bi-RNN 的模型

### 2.2 Attentive Read

Attentive Read 就是使用上面的 Attention 机制，利用 decode 的隐藏层与 encode 的隐藏层做一个 Attention，得到当前的一个 context vector $c_t$作为 decode 输入的一部分。

### 2.3 Decoder

#### 2.3.1 State update

Decoder 部分也就是重点所在,decode 相比于原来通用的 sequence2sequence 的模板有了一些改变，其中原来 decoder 的输入是基于$c_t$,$s_{t-1}$，以及$y_{t-1}$来更新 state($s_t$)的状态，现在的 state update（也就是 decode 的输出$s_t$）是把简单的$y_{t-1}$的这个输入变为

$$(e{y_{t-1}};\zeta(y_{t-1}))$$

接一个 DNN 网络这样的形式,见 state update 的子图,其中重点是

$$\zeta(y_{t-1})$$

其实是类似于 attention，文章中叫做 select read，是为了突出第 i-1 个词的位置信息,具体计算是根据 t-1 时刻的 decode 的输出与 M 计算的一个 attention，公式如下：

$$\zeta(y_{t-1})=\sum_{\tau=1}^{T_s}\rho_{t\tau}h_{\tau}$$

$$\rho_{t\tau}=\frac{1}{K}p(x_r,c\vert{s_{t-1},M})$$

再补充一点，如果这个词不在上文中出现我们有$\zeta(y_{t-1})=0$，这不就正好退化到原来普通的$y_{t-1}$！这样看应该十分直观了~

#### 2.3.2 Decoder 的输入组成

decoder 的输入有四部分组成，$c_t$为当前的 context vector，$s_{t}$ 为 decode 的隐藏层输出，$M$实际上是由每个词的隐藏层输出与位置 encode 特征组成的序列，见图，这里指 state update 模块。

decoder 的输出包括两部分，一种是在词表上的分布，另外一种是在历史信息中的词分布的概率，这样得到最终的一个词表+历史对话信息的分布。

#### 2.3.3 Prediction 模块

Prediction 模块包括两部分，即 copying 与 generation，最后的一个混合概率即可以用下面的公式表示，具体参数含义上面已介绍过：

$$p(y_t\vert{s_t,y_{t-1},c_{t},M})=p(y_t,g\vert{s_t,y_{t-1},c_t,M})+p(y_t,c\vert{s_t,y_{t-1},c_t,M})$$

g 表示 generation mode，c 表示 copying mode。

![Set](/img/Copy-mechanism/Set.png)

上述集合的图表示的很清晰，当 target word 来自不同的区间，对应不同的这样一个概率计算方法。当词语来自上文与词表的这样一个分布时，概率会有比如叠加的效果，当 target word 仅仅来自词表同时不在上文中，那么此时就是单一的概率计算分布，后面同样时这个道理。

上面的公式具体涉及两个 mode 的具体计算

generation 采用如下的方式对每个词打分，

$$\psi_g(y_t=v_i)=v_i^TW_0s_t$$

其实就是每个词的 embedding 与 decode 隐藏层点乘作为该词的评分，同时并以

$$\frac{1}{Z}e^{\psi_g(y_t)}$$

作为该词的概率。

copying 采用如下的方式对每个词打分，

$$\psi(y_t=x_j)=\sigma(h_j^TW_c)s_t$$

作为上文中每个字的评分函数，同时采用相同的如下

$$\frac{1}{Z}e^{\psi_g(y_t)}$$

作为概率的计算，只不过这时候的$y_t$区间来自上文的字.

#### 2.3.4

因为生成的$y_t$具有整个词表+历史信息的一个分布，所以 Loss 函数实际上是在包含 OOV 的词表上计算极大似然估计:

$$L=-\frac{1}{N}\sum_{k=1}^{N}\sum_{t=1}^{T}log(p(y_t^{(k)}\vert{y_{<t}^{(k)},X^{(k)}}))$$

## 3 模型的结果

模型在三种数据集上面做了实验：

1. 简单的模板

即设计规则生成的数据

![R1](/img/Copy-mechanism/R1.PNG)

2. 文本摘要

基于 LCSTC 数据集

![R2](/img/Copy-mechanism/R2.PNG)

3. 单论对话

作者自己设计的单轮对话数据集

![R3](/img/Copy-mechanism/R3.PNG)
