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

