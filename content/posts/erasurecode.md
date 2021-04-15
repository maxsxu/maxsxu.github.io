+++ 
draft = true
date = 2021-02-19T12:10:09+08:00
title = "纠删码"
description = ""
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++

在编码理论中，擦除码是在假定比特擦除（而不是比特错误）的情况下的前向纠错（FEC）码，它将前k个符号的消息转换为具有n个符号的较长消息（代码字），从而使可以从n个符号的子集中恢复原始消息。分数r = k / n称为编码率。分数k’/ k，其中k’表示恢复所需的符号数，称为接收效率。


# 参考
- Erasure code. https://en.wikipedia.org/wiki/Erasure_code
- 纠删码 Erasure Code, https://zhuanlan.zhihu.com/p/69374970
- Erasure-Code (纠删码) 最佳实践, https://zhuanlan.zhihu.com/p/106096265s
- Ceph 的正确玩法之 Ceph 纠删码理论与实践, https://www.huaweicloud.com/articles/8c0cfc8c91acf4a4a1f07066189430c1.html