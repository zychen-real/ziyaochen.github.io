---
layout: post
title: "End2End Task Completion nueral dialogue systems"
subtitle: ""
date: 2018-08-07
author: "ziyaochen"
header-img: "img/2017-bg.jpg"
catalog: true
mathjax: true
tags:
  - TASK
  - End-End

---

## 1. 写在前面的话

这一篇端到端的模型主要是在传统算法框架上面实现，即包括语言理解模块（领域识别，意图识别，槽位识别），对话管理模块（状态追踪。策略学习），自然语言生成模块。文章主要模块包括 user simulator， NLG（semantic frame to natural language），LU（natural language to semntic frame），DM。

## 2. 定义

![define](/img/Nueral-dialogue/define.png)

定义了用户的状态，用户状态$s_u$主要包括两部分，一部分是用户的对反馈的系统反馈$a_m$的操作，即 Agenda，另外一部分是用户的 goals，用户的 goal 不能发现就是两部分组成，即 inform_slot 以及 request_slot。未来便于理解，下面以一段对话实例说明上述的定义：

![sample1](/img/Nueral-dialogue/sample1.png)

![sample2](/img/Nueral-dialogue/sample2.png)

初始化阶段，用户的状态，用户的 goal 是满足根据约束条件 bar beer central 去填充 R 的具体值，即找到满足 inform_slot 的地方。A 里面存的就是一系列的 action，随着对话不断进行，A 不断根据对话采取 pop 或者 push action 的操作。

首先 user1：I'm looking for a nice bar serving beer。

主要提供了两个 slot-value pairs，user 说这句话，我们可以理解就是 agenda，在栈顶 pop 掉两个 inform。其实就是$a_u$造成$s_u$的状态变化。即$P(a_u\vert{s_u})$

其次 sys1：Ok, a wine bar. What price range?

同时更新了 goal 与 agenda，其实就是$a_m$造成的影响，即 P(s_u^{``}\vert{a_m,s_u^{`}}).

接下来看一下整体的转换

![p1](/img/Nueral-dialogue/p1.png)

![p2](/img/Nueral-dialogue/p2.png)

![p3](/img/Nueral-dialogue/p3.png)

![p4](/img/Nueral-dialogue/p4.png)

warm start 采用 rule policy 的方式，当 experience reply pool 达到一定的 size 的时候停止 warm start，开始 RL training。

![warmStart](/img/Nueral-dialogue/warmStart.png)

DQN 具体实现细节：

DQN 的 train 过程：

![DQNtrain](/img/Nueral-dialogue/DQNtrain.png)

随机从 experience reply pool 挑选宇哥 batch 的数据进行训练，包括用 costFun 计算 grads 和 cost，用 singleBatch 进行 aprameter 的参数更新，这样我们就能更新 DQN 的参数

每个 sample 的格式如下：

![form](/img/Nueral-dialogue/form.png)

## 3 Result

![res](/img/Nueral-dialogue/res.png)
