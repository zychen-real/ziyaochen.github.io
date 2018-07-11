---
layout:     post
title:      "ACL_2016_Incorporating Copying Mechanism in Sequence-to-Sequence Learning"
subtitle:   ""
date:       2018-7-11
author:     "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
    - COPY
    
---

### 1. 写在前面的话

Copying Mechanism的出现缓解了NLP的一个难题，即OOV问题。什么是OOV就是通常情况下我们的词表大小是确定的，但是测试集中有可能会出现词表中没有陌生词，之前的处理方式就是直接用UNK的符号表示这一类OOV，因为通常情况下OOV不会非常多，但是你要想再提高比如1%的准确率，那么这个Copying Mechanism你不可或缺。Copying Mechanism 能解决例如旅行者背包的问题。整体来看，现在这个Copying Mechanism是十分常用的一种手段，例如在对话系统中，sequence-to-sequence的这样一个encode-decode框架下，decode预测的词是除了词表的分布外结合对话上文出现词的这样一个概率和。借用manning的这篇 Key-Value Retrieval Networks for Task-Oriented Dialogue表示一下这个分布结果：

figure

可以看到同时又词表的分布与上文的分布，对于两者重叠的词将会有更大的概率。

### 2. 简单的介绍一下attention机制

Attention机制你可以理解成一个query在memory中查找跟它相似的地方并给予加强。而一个memory你可以理解成是key与value构成的，这样想的话，attention其实就是在寻找query与key的匹配程度并且加强这部分的value权重。通式可以表示成这样:

$$att=softmax(F(Query,Key))*Value$$

上述公式中的F可以是常用的比如一个DNN等。我们看一下encode-decode中的context vector是怎么计算的：

假设$$s_{t-1}$$代表上一轮decode的隐藏层，$$h_{\tau}$$ 代表encode的第$$\tau$$此时的隐藏那么可以得到对于每个encode隐藏层的这样一个权重计算公式：

$$\alpha_{t\tau}=\frac{e^{\eta(s_{t-1},h_{\tau})}}{\sum_{\tau^'=1}^{T_s}e^{\eta(s_{t-1},h_{\tau^{'}})}}$$

这样我们就可以得到当前的一个context vector 用$$c_t$$表示如下：

$$c_t=\sum_{\tau=1}^{T_s}\alpha_{t\tau}h_{\tau}$$


话不多说，直接进入模型。

### 2. Model

figure













