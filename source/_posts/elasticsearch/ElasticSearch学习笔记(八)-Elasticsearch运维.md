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
  
#### Elasticsearch设置用户名和密码

* 版本号: `7.14.1`

* 需要在配置文件中开启`x-pack`验证,修改`config`目录下面的`elasticsearch.yml`文件,添加以下内容:

  ```yml
  xpack.security.enabled: true
  xpack.license.self_generated.type: basic
  xpack.security.transport.ssl.enabled: true
  ```

* 设置密码,两种模式:`auto`和`interactive`

  * `auto`:自动,使用该模式后,会自动生成密码并打印在控制台上

    ```sh
    ./bin/elasticsearch-setup-passwords auto
    ```

  * `interactive`: 交互模式.

    ```sh
    ./bin/elasticsearch-setup-passwords interactive
    Initiating the setup of passwords for reserved users elastic,kibana,logstash_system,beats_system.
    You will be prompted to enter passwords as the process progresses.
    Please confirm that you would like to continue [y/N]y
    Enter password for [elastic]: 
    Enter password for [elastic]: 
    Reenter password for [elastic]: 
    Enter password for [elastic]: 
    Reenter password for [elastic]: 
    Enter password for [kibana]: 
    Reenter password for [kibana]: 
    Enter password for [logstash_system]: 
    Reenter password for [logstash_system]: 
    Enter password for [beats_system]: 
    Reenter password for [beats_system]: 
    Changed password for user [kibana]
    Changed password for user [logstash_system]
    Changed password for user [beats_system]
    Changed password for user [elastic]
    ```

* 修改密码

  ```sh
  curl -H "Content-Type:application/json" -XPOST -u elastic 'http://127.0.0.1:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "123456" }'
  ```

  

​    

  