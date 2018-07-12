---
layout:     post
title:      "ACL_2018_Sequicity: Simplifying Task-oriented Dialogue Systems with Single Sequence-to-Sequence Architectures"
subtitle:   ""
date:       2018-7-10
author:     "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
    - TASK
    - End-End
    
---

### 1. 写在前面的话

该文章引入了一个定义 belief span简称(bspan)，说实在点，其实就是状态追踪的槽位信息，比如当前轮次对话的inform_slot的槽位与request_slot槽位。目前任务型的数据大概只有两个数据集满足该要求——CamRes676(订餐)与KVRET(车载助手)。为何呢，因为该模型对数据集有极高的要求，对于每个session的每一句话，你都需要标注当前的槽位信息，轮次等各种特征，这需要花费大量的力气，所以这种数据集更加不好弄。

文章提出了一种通用的框架来解决端到端的task-Oriented Dialogue 这个任务，取名sequicity，该框架具有以下两个优点，1. 本来我们的状态追踪需要从$$U_1R_1... U_tR_t$$,现在只要根据上一轮的用户回答$$U_{t-1}$$以及bspan的信息B_{t-1}以及上一轮的机器回复$$R_{t-1}$$——即$$U_{t}B_{t-1}R_{t-1}$$可以表示整个对话的走向。2. 还有一个优点是由于bspan是个固定大小的表示，整体的算法复杂度从$$O(n^2)$$下降到$$O(n)$$。

### 2. Sequicity

figure

接下来详细聊一下sequicity这个框架，先介绍一下基本单元，$$B_t$$表示当前对话的隐藏状态，即当前所涉及到的slot信息，包括request_slot与inform_slot，理解为上文到时间t的一个状态追踪，$$U_t$$为t时刻的用户的回答，$$R_t$$为t时刻的机器回复。训练的时候数据是这样组织的，{$$(B_0R_0U_1;B_1R_1);(B_1R_1U_2;B_2R_2);...;(B_{t-1}R_{t-1}U_t;B_tR_t)$$},$$B_0,R_0$$初始化为空。从图中可以看到，训练的时候采用的是模板的形式，测试的时候会从实际的KB中寻找替换成具体的value，如餐厅的具体名字。由于B的分布与R的分布不一样，作者采用分步decode的方式，先用第一步decode生成$$B_t$$,再利用第二步decode生成$$R_t$$，并且第一步产生的$$B_t$$也是生成$$R_t$$的条件之一。表达式可以写成如下：

$$B_t=seq2seq(B_{t-1}R_{t-1}U_t\vert{0,0})$$

$$R_t=seq2seq(B_{t-1}R_{t-1}U_t\vert{B_t,k_t})$$

### 3. Two-Stage CopyNet

figure

文章中在这个通用框架上实例化了一个例子。使用基于copyNet的decode，copy mechanical 这里不再叙述，有需要的同学可以去我的另外一篇专门讲到过：https://ziyaochen.github.io/2018/07/11/Copy-Mechanism/

这里提几个要点，主要是copyNet的copy attention的问题。第一个copyNet的attention是用户回复上X进行，第二个copyNet的attention是在$$B_t$$上进行。

数据库的交互主要涉及第一步decode后产生的$$B_t$$用于在KB中产生$$k_t$$的过程。$$k_t$$实际上为一个三维的vector，每个维度分别代表1个，0个或者多个实体。同时$$k_t$$作为额外的特征加入$$y_j$$中一起作为生成。(这里对$$k_t$$的具体形态表示疑惑，文章并没有讲清楚如何实现生成后template的替换？？)

### 4. Training

1. 采用交叉熵进行监督学习

$$\sum_{j=1}^{m}y_jlogP_j(y_j)$$

2.采用强化学习微调上述训练好的模型

由于decode一些templet比decode一些其他词更重要，文章中采用了RL进行训练，将decode的网络理解为policy网络，表示为$$\pi_{\theta}(y_j)$$，用于decode $$y_j$$,其中$$m^i+1<=j<=m$$，每个选择的$$y_j$$可以认为action，被GRU产生的隐藏表示，可以认为state，训练时候的policy gradient 如下：

##\sum_{j=1}^{m}y_jlogP_j(y_j)\frac{\partial{log\pi_{\theta}(y_j)}}{\theta}##

其中 $$r^{(j)}=r^{(j)}+\lambdar^{(j)}+\lambdar^{(j+1)}...\lambda^{m-j+1}r^{(m)}$$

reward采用这么设置，当request_slot被decode的时候，reward为1，否则其他的为-0.1，$$\lambda$$设置为0.8

### 5.评价指标与结果

数据集介绍如图：

figure


#### 4.1 评价指标

1. BLEU 

2. Entity match rate

3. Success F1

4. Training time

#### 4.2 Result

NDM 

NDM+Att+SS

LIDM

KVRN

这几种模型与本文比较如下：

figure

OOV的test结果如下：

figure

















