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
