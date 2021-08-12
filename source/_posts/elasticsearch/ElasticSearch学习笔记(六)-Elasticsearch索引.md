---
title: ElasticSearch(六)-索引
mermaid: true
date: 2021-08-12 14:01:17
cover: /img/cover/Elasticsearch.jpg
tags:
- 索引
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

#### 索引收缩

* 索引的分片数是不可更改的，如要减少分片数可以通过收缩方式收缩为一个新的索引。新索引分片数必须是原分片数的因子值，如原分片数是8，则新索引分片数可以为4、2、1 。

##### 收缩流程

* 先把所有主分片都转移到一台主机上；
*  在这台主机上创建一个新索引，分片数较小，其他设置和原索引一致；
* 把原索引的所有分片，复制（或硬链接）到新索引的目录下；
* 对新索引进行打开操作恢复分片数据；(可选)重新把新索引的分片均衡到其他节点上。

######  收缩前准备工作

* 将原索引设置为只读；将原索引各分片的一个副本重分配到同一个节点上，并且要是健康绿色状态。

```sh
PUT /my_source_index/_settings
{
"settings": {
    "index.routing.allocation.require._name": "shrink_node_name",
    "index.blocks.write": true
  }
}
```

###### 进行收缩

```sh
POST my_source_index/_shrink/my_target_index
{
"settings": {
    "index.number_of_replicas": 1,
    "index.number_of_shards": 1,
    "index.codec": "best_compression"
}}
```

######  监控收缩过程

```sh
GET _cat/recovery?v
GET _cluster/health
```

#### 索引拆分

* 当索引的分片容量过大时，可以通过拆分操作将索引拆分为一个倍数分片数的新索引。能拆分为几倍由创建索引时指定的`index.number_of_routing_shards `路由分片数决定。这个路由分片数决定了根据一致性hash路由文档到分片的散列空间。如`index.number_of_routing_shards = 30` ，指定的分片数是5，则可按如下倍数方式进行拆分：

  ```
  5 → 10 → 30 (split by 2, then by 3)
  5 → 15 → 30 (split by 3, then by 2)
  5 → 30 (split by 6)
  ```

  * **注意：**只有在创建时指定了index.number_of_routing_shards 的索引才可以进行拆分，ES7开始将不再有这个限制。

* 准备一个索引来做拆分

  ```
  PUT my_source_index
  {
      "settings": {
          "index.number_of_shards" : 1,
          "index.number_of_routing_shards": 2 
      }
  }
  ```

* 设置索引只读

  ```
  PUT /my_source_index/_settings
  {
  "settings": {
  "index.blocks.write": true
    }
  }
  ```

* 监控收缩过程

  ```sh
  GET _cat/recovery?v
  GET _cluster/health
  ```


#### 别名滚动

* 对于有时效性的索引数据，如日志，过一定时间后，老的索引数据就没有用了。我们可以像数据库中根据时间创建表来存放不同时段的数据一样，在ES中也可用建多个索引的方式来分开存放不同时段的数据。比数据库中更方便的是ES中可以通过别名滚动指向最新的索引的方式，让你通过别名来操作时总是操作的最新的索引。

* ES的rollover index API 让我们可以根据满足指定的条件（时间、文档数量、索引大小）创建新的索引，并把别名滚动指向新的索引。

*  Rollover Index 示例

  * 创建一个名字为logs-0000001 、别名为logs_write 的索引

    ```
    PUT /logs-000001
    {
        "aliases": {
            "logs_write": {}
        }
    }
    ```

  * 如果别名logs_write指向的索引是7天前（含）创建的或索引的文档数>=1000或索引的大小>= 5gb，则会创建一个新索引 logs-000002，并把别名logs_writer指向新创建的logs-000002索引

    ```
    POST /logs_write/_rollover
    {
        "conditions": {
            "max_age":  "7d",
            "max_docs":  1000,
            "max_size": "5gb"
        }
    }
    ```

* Rollover Index 新建索引的命名规则

  * 如果索引的名称是-数字结尾，如logs-000001，则新建索引的名称也会是这个模式，数值增1。

  * 如果索引的名称不是-数值结尾，则在请求rollover api时需指定新索引的名称:

    ```
    POST /my_alias/_rollover/my_new_index_name
    {
        "conditions": {
            "max_age":  "7d",
            "max_docs":  1000,
            "max_size": "5gb"
        }
    }
    ```



