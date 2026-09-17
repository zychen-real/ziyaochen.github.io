---
layout: post
title: "ACL_2018_Mem2Seq: Effectively Incorporating Knowledge Bases into End-to-End Task-Oriented Dialog Systems"
subtitle: ""
date: 2018-07-09
author: "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
  - TASK
  - End-End
---

## 1. 写在前面的话

2018 ACL 上面有两篇端到端的任务型文章，Mem2Seq: Effectively Incorporating Knowledge Bases into End-to-EndTask-Oriented Dialog Systems 就是其中一篇，它的框架就是 encode-decode 的模式，它将相关的 KB 信息作为 memory 跟历史对话放在一起作为训练数据，encode 端的输出实际是历史对话几个 hop 增强后的 attention，也就是对历史对话的一个 attention.这也是沿用了 MemNNs 的特点，MemNNs 能对一个很大的外部数据实现得到一个循环的 attention。下面详细介绍一下这个模型。

## 2. Mem2Seq

模型的整体框架图如下

![Model](/img/Mem2Seq-in-dialogue-oriented-dialogue/Model.png)

接下来逐个模块介绍。

### 2.1. Encode

![Encode](/img/Mem2Seq-in-dialogue-oriented-dialogue/Encode.png)

使用 MemNNs 作为历史对话追踪的的原因

1. MemNNs 实际操作中是 query(文章中是用 zero_vector 来表示 query)对若干个 embedding 矩阵进行 addressing 与 reading 的操作，具体如下：

2. 文章中的每个 memory 都是词表级别的 embedding 矩阵，第 k 个 memory 记为$C_k$,   假设有 K+1 个 memory unit，记为$C_1,C_2,...C_{k+1}$；
3. Addressing 的操作就是在历史对话信息(story)在词表中的 embedding 与 query 的乘积进行 softmax 得到概率分布 p,下面是对第 i 个 memory unit 的 reading 操作

$$p_i^k=Softmax((q^k)^TC_i^k)$$

$p^k$ 可以认为是 memory 选择器，就是对 query 中的相关的 memory 进行选择。
4. 对第 i+1 的 memory unit 进行 reading，实际是就是得到 attention 的向量输出：

$$o^k=\sum_{i}{ }p_i^kC_i^{k+1}$$

到这一步没问题，文章中

$$q^{k+1}=q^k+o^k$$

这里也就是源码中的 u 其实就是 zero vector 也就是 query：

![uattention](/img/Mem2Seq-in-dialogue-oriented-dialogue/uattention.png)

也就是说，encode 的输出其实是 muti-hop 加强后的 attention，作为 decode 的初始化隐藏层初始值。

### 2.1. Decode

Decode 的模型作者选用了 GRU 模型，每个词生成的时候会进行与 memory 的一个交互，包括两个 loss，一个是在词表分布中选出最大概率的词语，或者是用 point loss 从历史对话信息与 KB 中直接获得生成的词。作者源码中 encode 的 memory 与 decode 竟然不共享？？

具体细节如下：

![Decode](/img/Mem2Seq-in-dialogue-oriented-dialogue/Decode.png)

1. decode 每个隐藏层的求解：

$$h_t=GRU(C^1(y_{t-1}^{-}),h_{t-1})$$

2. 生成的词语的分布采用$o_1$与$h_t$的 concat 经过一个全连接层后的 softmax 得到的概率分布，其实就是在词表上的一个概率分布。

$$P_{vocab}(y_t^{-})=Softmax(W_1[h_t:o^1])$$

3.生成的词来自历史信息与 KB 中，这时候需要 point network 进行计算。

ptr 可以理解为经过整个 memory 最后输出的一个 attention,就是对 memory 中（query 与历史对话）每个词的概率的一个分布。

4. 两者分布计算的源码如下：

![Distribution](/img/Mem2Seq-in-dialogue-oriented-dialogue/Distribution.png)

loss 计算源码片段如下：

![Loss](/img/Mem2Seq-in-dialogue-oriented-dialogue/Loss.png)

## 3. Result

### 3.1. 数据集

采用多轮任务型对话的数据集：包括 bAbI（订餐部分 Task1-5）,DSTC(对话状态追踪的数据集)，In-Car Assistant(车载对话)

### 3.2. 评价标准

1. BLEU，就是看 n-gram 的重叠程度

2. Entity F1 response 中实体的准确率

3. Per-response 模型生成的回复与 gold response(就是测试集中)的一模一样。

4. Dialog Accuracy 该对话中任何一个 Per-response 都正确才算准确，是一个任务完成率的一个指标

### 3.3. 评价结果

三种数据集跑的结果如下

![Result](/img/Mem2Seq-in-dialogue-oriented-dialogue/Result.png)

模型训练相比其他模型的优势：

![responseRate](/img/Mem2Seq-in-dialogue-oriented-dialogue/responseRate.png)

同时作者深入探究了不同 hop 的影响以及同时可视化了最后一个 hop 输出的 memory 的 attention，就是对历史对话消息与 KB 的一个概率分布。
