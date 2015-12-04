title: HBase线上集群硬件升级纪要
date: 2011-11-28 15:19:09
categories: dev
tags: HBase
---

## 1. 现状

15台region server，4000多个region。

## 2. 升级目标

将原有15台配置较差的region server，升级为新的配置好的服务器。这是一次硬件升级。即用新的15台机器，替换旧的15台机器。

## 3. 升级过程

为了保证升级平稳过渡，采用以下策略
将新的15台 server加入到region server集群中；
观察region server的负载分布，等待这30个region server均匀；
待均匀后，让原有的region server下线；
等待region的恢复
以上过程，大约耗时5分钟可以完成，期间数据可以访问。在d步骤中，前3分钟没有什么反应（从基于hdfs的hlog中读取恢复region），后续两分钟，2000多个region恢复上线。

------END-------

伍涛@成都 重整
2013-07-28
