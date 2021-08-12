---
title: ElasticSearch(八)-运维
mermaid: true
date: 2021-08-12 14:56:17
cover: /img/cover/Elasticsearch.jpg
tags:
- 运维
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

#### cat命令

* 健康检查

  * `/_cat/health?v`

    ```
    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
    1628751491 06:58:11  local-cluster yellow          1         1      3   3    0    0        2             0                  -                 60.0%
    ```

* 节点分片数量查询

  * `/_cat/allocation?v`

  * 查询每个节点上分配的分片（shard）的数量和每个分片（shard）所使用的硬盘容量
  
    ```
     shards disk.indices disk.used disk.avail disk.total disk.percent host      ip        node
          3       40.6mb   108.8gb    817.4gb    926.3gb           11 127.0.0.1 127.0.0.1 local-es
          2                                                                               UNASSIGNED
    ```
  
* 节点统计
  
  * `/_cat/nodes?v`
  * 显示当前es集群节点的状况
  
* 文档数量查询
  
  * `/_cat/count?v`
  
* 查询master节点
  
  * `/_cat/master?v`
  * 用于显示master节点ID，绑定IP地址，节点名称
  
* 查询节点自定义属性
  
  * `/_cat/nodeattrs?v`
  
* 执行任务查询
  
  * `/_cat/pending_tasks?v`
  
* 索引分片恢复查询
  
  * `/_cat/recovery?v`
  
* 节点线程查询
  
  * `/_cat/thread_pool`
  
* 索引所在分片查询
  
  * `/_cat/shards`
  
  
  
  
  
    
  
  