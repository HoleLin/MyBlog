---
title: ElasticSearch(七)-路由
mermaid: true
date: 2021-08-12 14:01:17
cover: /img/cover/Elasticsearch.jpg
tags:
- 路由
categories:
- Elasticsearch
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [Elasticsearch 最佳运维实践 - 总结（二）](https://www.cnblogs.com/kevingrace/p/10682264.html)

#### 路由

* 环境

  ```
  集群元信息
  Cluster-name：local-es
  Nodes:
       node1   192.168.0.1   master
       node2   192.168.0.2
       node3   192.168.0.3
  Indics:
      s0:
            shard0:
                   primay: 192.168.0.1
                   rep:192.168.0.3
  ```

* 创建索引的流程

  * 请求node3创建索引
  * node3请求转发给master节点
  * 选择节点存放分片、副本，记录元信息
  * 通知给参与存放索引分片、副本的节点从节点，创建分片、副本
  * 参与节点向主节点反馈结果
  * 等待时间到了，master向node3反馈结果信息，node3响应请求。
  * 主节点将元信息广播给所有从节点。

* 索引文档流程

  * node2计算文档的路由值得到文档存放的分片（假定路由选定的是分片0）。
  * 将文档转发给分片0的主分片节点 node1。
  * node1索引文档，同步给副本节点node3索引文档。
  * node1向node2反馈结果
  * node2作出响应

* 搜索

  * node2解析查询。
  * node2将查询发给索引s1的分片/副本（R1,R2,R0）节点
  * 各节点执行查询，将结果发给Node2
  * node2合并结果，作出响应。

#### **文档如何路由**

* 决定文档存放到哪个分片上就是文档路由。ES中通过下面的计算得到每个文档的存放分片：
  `shard = hash(routing) % number_of_primary_shards`
* routing 是用来进行hash计算的路由值，默认是使用文档id值。我们可以在索引文档时通过routing参数指定别的路由值，在索引、删除、更新、查询中都可以使用routing参数（可多值）指定操作的分片。

