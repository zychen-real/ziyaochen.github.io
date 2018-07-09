---
layout:     post
title:      "ACL_2018_Mem2Seq: Effectively Incorporating Knowledge Bases into End-to-End Task-Oriented Dialog Systems"
subtitle:   ""
date:       2018-7-9
author:     "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
    - TASK
    - End-End
---

### 1. 写在前面的话

2018 ACL上面有两篇端到端的任务型文章，Mem2Seq: Effectively Incorporating Knowledge Bases into End-to-EndTask-Oriented Dialog Systems就是其中一篇，它的框架就是encode-decode的模式，它将相关的KB信息作为memory跟历史对话放在一起作为训练数据，encode端的输出实际是历史对话几个hop增强后的attention，也就是对历史对话的一个attention.这也是沿用了MemNNs的特点，MemNNs能对一个很大的外部数据实现得到一个循环的attention。下面详细介绍一下这个模型。

### 2. Mem2Seq

模型的整体框架图如下

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Model.png)

接下来逐个模块介绍。

#### 2.1.Encode

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Encode.png)

使用MemNNs作为历史对话追踪的的原因

1. MemNNs实际操作中是query对若干个embedding矩阵进行addressing与reading的操作，具体如下：

2. 文章中的每个memory都是词表级别的embedding矩阵，第k个memory记为$$C_k$$,   假设有K+1个memory unit，记为$$C_1,C_2,...C_{k+1}$$，query的输入为one-hot的表示；
3. Addressing的操作就是在query在词表中的embedding，进行softmax得到概率分布p,下面是对第i个memory unit的reading操作

$$p_i^k=Softmax((q^k)^TC_i^k)$$

$$p^k$$ 可以认为是memory选择器，就是对query中的相关的memory进行选择，注意这里的query=KB+history dialogue。
4. 对第i+1的memory unit 进行reading，实际是就是得到attention的向量输出：

$$o^k=\sum_{i}{ }p_i^kC_i^{k+1}$$

到这一步没问题，文章中

$$q^{k+1}=q^k+o^k$$

的表示有歧义，其实这里不能认为是q，作者的代码中也体现了这一点，其实是随机初始化的u作为更新对象：

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/uattention.png)

也就是说，encode的输出其实是muti-hop加强后的attention，作为decode的初始化隐藏层初始值。

#### 2.1.Decode

Decode的模型作者选用了GRU模型，每个词生成的时候会进行与memory的一个交互，包括两个loss，一个是在词表分布中选出最大概率的词语，或者是用point loss从历史对话信息与KB中直接获得生成的词。作者源码中encode的memory与decode竟然不共享？？

具体细节如下：

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Decode.png)

1. decode 每个隐藏层的求解：

$$h_t=GRU(C^1(y_{t-1}^{-}),h_{t-1})$$

2. 生成的词语的分布采用$$o_1$$与$$h_t$$的concat经过一个全连接层后的softmax得到的概率分布，其实就是在词表上的一个概率分布。

$$P_{vocab}(y_t^{-})=Softmax(W_1[h_t:o^1])$$

3.生成的词来自历史信息与KB中，这时候需要point network进行计算。

ptr可以理解为经过整个memory最后输出的一个attention,就是对memory中（query与历史对话）每个词的概率的一个分布。

4. 两者分布计算的源码如下：

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Distribution.png)

loss计算源码片段如下：

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Loss.png)

### 3. Result

#### 3.1. 数据集

采用多轮任务型对话的数据集：包括bAbI（订餐部分Task1-5）,DSTC(对话状态追踪的数据集)，In-Car Assistant(车载对话)

#### 3.2. 评价标准

1. BLEU，就是看n-gram的重叠程度

2. Entity F1 response中实体的准确率

3. Per-response 模型生成的回复与gold response(就是测试集中)的一模一样。

4. Dialog Accuracy 该对话中任何一个Per-response都正确才算准确，是一个任务完成率的一个指标


#### 3.3. 评价结果

三种数据集跑的结果如下

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/Result.png)

模型训练相比其他模型的优势：

![](/img/Mem2Seq-in-dialogue-oriented-dialogue/responseRate.png)

同时作者深入探究了不同hop的影响以及同时可视化了最后一个hop输出的memory的attention，就是对历史对话消息与KB的一个概率分布。





















