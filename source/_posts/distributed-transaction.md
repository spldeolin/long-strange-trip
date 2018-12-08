---
title: 分布式事务的流程与思路

date: 2018-07-10 09:47:00

tags:
- 微服务

categories: Java

permalink: distributed-transaction
---



- 采用**可靠消息最终一致性**这样的方案

![](/images/distributed-transaction-01.jpg)



- 通过确保消息入库来保证消息的”可靠”
- 通过两个“僚机”集群来做正向流程的异常处理
- 特点是最终一致，所以不太适合强实时性的场合