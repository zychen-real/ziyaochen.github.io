---
layout:     post
title:      "End2End Task Completion nueral dialogue systems"
subtitle:   ""
date:       2018-8-7
author:     "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
    - TASK
    - End-End
    
---

### 1. 写在前面的话

这一篇端到端的模型主要是在传统算法框架上面实现，即包括语言理解模块（领域识别，意图识别，槽位识别），对话管理模块（状态追踪。策略学习），自然语言生成模块。文章主要模块包括user simulator， NLG（semantic frame to natural language），LU（natural language to semntic frame），DM。

### 2. 定义

![](/img/Nueral-dialogue/define.png)

定义了用户的状态，用户状态$$s_u$$主要包括两部分，一部分是用户的对反馈的系统反馈$$a_m$$的操作，即Agenda，另外一部分是用户的goals，用户的goal不能发现就是两部分组成，即inform_slot以及request_slot。未来便于理解，下面以一段对话实例说明上述的定义：

![](/img/Nueral-dialogue/sample1.png)

![](/img/Nueral-dialogue/sample2.png)

初始化阶段，用户的状态，用户的goal是满足根据约束条件bar beer central 去填充R的具体值，即找到满足inform_slot的地方。A里面存的就是一系列的action，随着对话不断进行，A不断根据对话采取pop或者push action 的操作。

首先 user1：I'm looking for a nice bar serving beer。 

主要提供了两个slot-value pairs，user说这句话，我们可以理解就是agenda，在栈顶pop掉两个inform。其实就是$$a_u$$造成$$s_u$$的状态变化。即$$P(a_u\vert{s_u})$$

其次sys1：Ok, a wine bar. What price range?

同时更新了goal与agenda，其实就是$$a_m$$造成的影响，即P(s_u^{``}\vert{a_m,s_u^{`}}).

接下来看一下整体的转换

![](/img/Nueral-dialogue/p1.png)


![](/img/Nueral-dialogue/p2.png)

![](/img/Nueral-dialogue/p3.png)

![](/img/Nueral-dialogue/p4.png)

warm start 采用rule policy 的方式，当experience reply pool 达到一定的size的时候停止warm start，开始RL training。

figure

DQN具体实现细节：

DQN的train过程：

![](/img/Nueral-dialogue/DQNtrain.png)

随机从experience reply pool挑选宇哥batch的数据进行训练，包括用costFun计算grads和cost，用singleBatch进行aprameter的参数更新，这样我们就能更新DQN的参数

每个sample的格式如下：

figure







