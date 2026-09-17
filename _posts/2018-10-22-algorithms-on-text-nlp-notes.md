---
layout: post
title: "文本上的算法"
subtitle: ""
date: 2018-10-22
author: "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
  - NLP

---

## 1. 写在前面的话

最近在看路彦雄写的《文本上的算法——深入浅出的自然语言处理》,有些许感悟是之前没有想过的，十分好的想法与思考，写下备注下。全书深入浅出，理论篇主要从
概率论与信息论入手，同时也简单介绍了一下贝叶斯法则。然后从最优求解的角度引出最大似然估计与最大后验，以及通用的梯度下降法。最后从机器学习的角度，介绍传统的
机器学习算法包括逻辑回归，最大熵/条件随机场，以及深入介绍了主题模型。之后是 NLP 深度学习中的常用的 RNN,LSTM,GRU,CW-RNN 等，一些 attention 的思想等等

## 2. 一些笔记

1、主题模型的常用两种求解方式。变分-EM 算法与 Gibbs Sampling 的方法。前者通常就是在 EM 算法的 E 步对隐藏变量的后验概率进行近似得到 Q 函数，M 步就是最大化近似函数 Q。后者的 gibbs sampling 比较熟悉，通过边界条件概率采样得到全概率分布。以及用 gibbs sampling 对主题模型的实现。

2、浅层网络训练缺陷。初始值随机，容易收敛到局部最优，导致过拟合。增加隐藏会导致梯度稀疏。改进：无监督，逐层训练和微调等等。

3、结合主题模型的词级别 embedding。word muti-embedding。 解决一次多义的情况。

word muti-embedding 算法：

Input: D 个句子，窗口大小为 n，上下文窗口为 d，多义项词典

Output：词向量$w_i=1...N$

    for t=1..D do

        K=word number of t's sentence

        w=lookup the K words

        for i=1..K words

            for i=1..K do

            $context(w_i)=c^{w_i}={w_{i-d},..w_{i-1},w_{i+1},..w_{i+d}\vert{w_i}}$

            $class(w_i,z)=c^{w_i}={w_{i-d},..w_{i-1},w_{i+1},..w_{i+d}\vert{w_i}}$

            $z=argmax x_z\sum_{a=1..2d}^{}\sum_{b=1..m}^{} S(C_a^{w_i},Z_b^{w_i,z})$

            $\theta(b,d,w^{i,z},U,H,C)=\theta(b,d,w^{i,z},U,H,C)+\varepsilon\frac{\partial{logP(w_{i,z}\vert{w_{i-n+1,..,w_{i-1}}})}}{\partial{}\theta}$

4、计算 fast
系统级别，算法/数据结构级别，代码级别。尽可能用局部变量，近可能少调用函数，用指针防止 copy 的大量时间，处理的数据尽可能紧凑，用内存
