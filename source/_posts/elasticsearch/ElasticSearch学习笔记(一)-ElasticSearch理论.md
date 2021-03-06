---
title: ElasticSearch(一)-理论
mermaid: true
date: 2021-06-12 00:21:17
cover: /img/cover/Elasticsearch.jpg
tags:
- 理论
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

### ElasticSerch使用场景

* **存储**

  > ElasticSearch天然支持分布式，具备存储海量数据的能力，其搜索和数据分析的功能都建立在ElasticSearch存储的海量的数据之上；ElasticSearch很方便的作为海量数据的存储工具，特别是在数据量急剧增长的当下，ElasticSearch结合爬虫等数据收集工具可以发挥很大用处

* **搜索**

  > ElasticSearch使用倒排索引，每个字段都被索引且可用于搜索，更是提供了丰富的搜索api，在海量数据下近实时实现近秒级的响应,基于Lucene的开源搜索引擎，为搜索引擎（全文检索，高亮，搜索推荐等）提供了检索的能力。 具体场景:
  >
  > * Stack Overflow（国外的程序异常讨论论坛），IT问题，程序的报错，提交上去，有人会跟你讨论和回答，全文检索，搜索相关问题和答案，程序报错了，就会将报错信息粘贴到里面去，搜索有没有对应的答案；
  > * GitHub（开源代码管理），搜索上千亿行代码；
  > * 电商网站，检索商品；
  > * 日志数据分析，logstash采集日志，ElasticSearch进行复杂的数据分析（ELK技术，elasticsearch+logstash+kibana）

* **数据分析**

  > ElasticSearch也提供了大量数据分析的api和丰富的聚合能力，支持在海量数据的基础上进行数据的分析和处理。具体场景：
  >
  > * 爬虫爬取不同电商平台的某个商品的数据，通过ElasticSearch进行数据分析（各个平台的历史价格、购买力等等）；

### ElasticSerch核心概念

![img](https://www.holelin.cn/img/elasticsearch/es%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

##### 近实时

> Near Realtime (NRT)近实时：数据提交索引后，立马就可以搜索到。

##### Cluster集群

> 一个集群由一个唯一的名字标识，默认为“elasticsearch”。集群名称非常重要，具体相同集群名的节点才会组成一个集群。集群名称可以在配置文件中指定。集群中有多个节点，其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。ElasticSearch的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看ElasticSearch集群，在逻辑上是个整体，你与任何一个节点的通信和与整个ElasticSearch集群通信是等价的。

##### Node 节点

> 存储集群的数据，参与集群的索引和搜索功能。像集群有名字，节点也有自己的名称，默认在启动时会以一个随机的UUID的前七个字符作为节点的名字，你可以为其指定任意的名字。通过集群名在网络中发现同伴组成集群。一个节点也可是集群。每一个运行实例称为一个节点,每一个运行实例既可以在同一机器上,也可以在不同的机器上。所谓运行实例,就是一个服务器进程，在测试环境中可以在一台服务器上运行多个服务器进程，在生产环境中建议每台服务器运行一个服务器进程。

##### Index 索引

> 一个索引是一个文档的集合（等同于solr中的集合）。每个索引有唯一的名字，通过这个名字来操作它。一个集群中可以有任意多个索引。索引作动词时，指索引数据、或对数据进行索引。Type 类型：指在一个索引中，可以索引不同类型的文档，如用户数据、博客数据。从6.0.0 版本起已废弃，一个索引中只存放一类数据。Elasticsearch里的索引概念是名词而不是动词，在Elasticsearch里它支持多个索引。 一个索引就是一个拥有相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来 标识（必须全部是小写字母的），并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，你能够创建任意多个索引。

##### Document 文档

> 被索引的一条数据，索引的基本信息单元，以JSON格式来表示。一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档、某一个产品的一个文档、某个订单的一个文档。文档以JSON格式来表示，而JSON是一个到处存在的互联网数据交互格式。在一个index/type里面，你可以存储任意多的文档。注意，一个文档物理上存在于一个索引之中，但文档必须被索引/赋予一个索引的type。

##### Shard 分片

> 在创建一个索引时可以指定分成多少个分片来存储。每个分片本身也是一个功能完善且独立的“索引”，可以被放置在集群的任意节点上（分片数创建索引时指定，创建后不可改了。备份数可以随时改）。索引分片，ElasticSearch可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。分片的好处：
> **-** 允许我们水平切分/扩展容量
> **-** 可在多个分片上进行分布式的、并行的操作，提高系统的性能和吞吐量。

##### Replication 备份

>  一个分片可以有多个备份（副本）。备份的好处：
> **-** 高可用扩展搜索的并发能力、吞吐量。
> **-** 搜索可以在所有的副本上并行运行。

##### Primary Shard 主分片

> 每个文档都存储在一个分片中，当你存储一个文档的时候，系统会首先存储在主分片中，然后会复制到不同的副本中。默认情况下，一个索引有5个主分片。你可以在事先制定分片的数量，当分片一旦建立，分片的数量则不能修改。

##### Replica Shard 副本分片

> 每一个分片有零个或多个副本。副本主要是主分片的复制，其中有两个目的：
> **-** 增加高可用性：当主分片失败的时候，可以从副本分片中选择一个作为主分片。
> **-** 提高性能：当查询的时候可以到主分片或者副本分片中进行查询。默认情况下，一个主分配有一个副本，但副本的数量可以在后面动态的配置增加。副本必须部署在不同的节点上，不能部署在和主分片相同的节点上。

##### term索引词

> 在elasticsearch中索引词(term)是一个能够被索引的精确值。foo，Foo几个单词是不相同的索引词。索引词(term)是可以通过term查询进行准确搜索。

##### text文本

> 是一段普通的非结构化文字，通常，文本会被分析称一个个的索引词，存储在elasticsearch的索引库中，为了让文本能够进行搜索，文本字段需要事先进行分析；当对文本中的关键词进行查询的时候，搜索引擎应该根据搜索条件搜索出原文本

##### analysis

> 分析是将文本转换为索引词的过程，分析的结果依赖于分词器，比如： FOO BAR, Foo-Bar, foo bar这几个单词有可能会被分析成相同的索引词foo和bar，这些索引词存储在elasticsearch的索引库中。当用 FoO:bAR进行全文搜索的时候，搜索引擎根据匹配计算也能在索引库中搜索出之前的内容。这就是elasticsearch的搜索分析。

##### routing路由

> 当存储一个文档的时候，他会存储在一个唯一的主分片中，具体哪个分片是通过散列值的进行选择。默认情况下，这个值是由文档的id生成。如果文档有一个指定的父文档，从父文档ID中生成，该值可以在存储文档的时候进行修改。

##### type类型

> 在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组相同字段的文档定义一个类型。比如说，我们假设你运营一个博客平台 并且将你所有的数据存储到一个索引中。在这个索引中，你可以为用户数据定义一个类型，为博客数据定义另一个类型，当然，也可以为评论数据定义另一个类型。

##### template

> 索引可使用预定义的模板进行创建,这个模板称作Index templatElasticSearch。模板设置包括settings和mappings。

##### mapping

> 映射像关系数据库中的表结构，每一个索引都有一个映射，它定义了索引中的每一个字段类型，以及一个索引范围内的设置。一个映射可以事先被定义，或者在第一次存储文档的时候自动识别。

##### field

> 一个文档中包含零个或者多个字段，字段可以是一个简单的值（例如字符串、整数、日期），也可以是一个数组或对象的嵌套结构。字段类似于关系数据库中的表中的列。每个字段都对应一个字段类型，例如整数、字符串、对象等。字段还可以指定如何分析该字段的值。

##### source field

> 默认情况下，你的原文档将被存储在_source这个字段中，当你查询的时候也是返回这个字段。这允许您可以从搜索结果中访问原始的对象，这个对象返回一个精确的json字符串，这个对象不显示索引分析后的其他任何数据。

##### id

> 一个文件的唯一标识，如果在存库的时候没有提供id，系统会自动生成一个id，文档的index/type/id必须是唯一的。

##### recovery

> 代表数据恢复或叫数据重新分布，ElasticSearch在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

##### River

> 代表ElasticSearch的一个数据源，也是其它存储方式（如：数据库）同步数据到ElasticSearch的一个方法。它是以插件方式存在的一个ElasticSearch服务，通过读取river中的数据并把它索引到ElasticSearch中，官方的river有couchDB的，RabbitMQ的，Twitter的，Wikipedia的，river这个功能将会在后面的文件中重点说到。

##### gateway

> 代表ElasticSearch索引的持久化存储方式，ElasticSearch默认是先把索引存放到内存中，当内存满了时再持久化到硬盘。当这个ElasticSearch集群关闭再重新启动时就会从gateway中读取索引数据。ElasticSearch支持多种类型的gateway，有本地文件系统(默认), 分布式文件系统，Hadoop的HDFS和amazon的s3云存储服务。

##### discovery.zen

> 代表ElasticSearch的自动发现节点机制，ElasticSearch是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互。

##### Transport

> 代表ElasticSearch内部节点或集群与客户端的交互方式，默认内部是使用tcp协议进行交互，同时它支持http协议（json格式）、thrift、servlet、memcached、zeroMQ等的传输协议（通过插件方式集成）。

### Elasticsearch常用插件

#### [elasticsearch-head](https://github.com/mobz/elasticsearch-head)

#### [bigdesk](https://github.com/hlstudio/bigdesk)

* Elasticsearch的一个集群监控工具，可以通过它来查看es集群的各种状态，如：cpu、内存使用情况，索引数据、搜索情况，http连接数等。

[**Kopf**](https://github.com/lmenezes/elasticsearch-kopf) 

* 一个ElasticSearch的管理工具，它也提供了对ES集群操作的API。

### Elasticsearch 的fielddata内存控制、预加载以及circuit breaker断路器

#### **fielddata核心原理**

> fielddata加载到内存的过程是lazy加载的，对一个analzyed field执行聚合时，才会加载，而且是field-level加载的一个index的一个field，所有doc都会被加载，而不是少数doc不是index-time创建，是query-time创建

#### **fielddata内存限制**

> elasticsearch.yml： indices.fielddata.cache.size: 20%，超出限制，清除内存已有fielddata数据fielddata占用的内存超出了这个比例的限制，那么就清除掉内存中已有的fielddata数据默认无限制，限制内存使用，但是会导致频繁evict和reload，大量IO性能损耗，以及内存碎片和gc

#### **监控fielddata内存使用**

```sh
#各个分片、索引的fielddata在内存中的占用情况
[root@elk-node03 ~]# curl -X GET 'http://10.0.8.47:9200/_stats/fielddata?fields=*'    
 
#每个node的fielddata在内存中的占用情况
[root@elk-node03 ~]# curl -X GET 'http://10.0.8.47:9200/_nodes/stats/indices/fielddata?fields=*'
 
#每个node中的每个索引的fielddata在内存中的占用情况
[root@elk-node03 ~]# curl -X GET 'http://10.0.8.47:9200/_nodes/stats/indices/fielddata?level=indices&fields=*'
```

#### **circuit breaker断路由**

> 如果一次query load的feilddata超过总内存，就会oom --> 内存溢出;
> circuit breaker会估算query要加载的fielddata大小，如果超出总内存，就短路，query直接失败;
> 在elasticsearch.yml文件中配置如下内容:
> indices.breaker.fielddata.limit： fielddata的内存限制，默认60%
> indices.breaker.request.limit： 执行聚合的内存限制，默认40%
> indices.breaker.total.limit：    综合上面两个，限制在70%以内

##### **限制内存使用 (Elasticsearch聚合限制内存使用)**

> 通常为了让聚合(或者任何需要访问字段值的请求)能够快点，访问fielddata一定会快点， 这就是为什么加载到内存的原因。但是加载太多的数据到内存会导致垃圾回收(gc)缓慢， 因为JVM试着发现堆里面的额外空间，甚至导致OutOfMemory (即OOM)异常。
>
> 然而让人吃惊的发现, Elaticsearch不是只把符合你的查询的值加载到fielddata. 而是把index里的所document都加载到内存，甚至是不同的 _type 的document。逻辑是这样的，如果你在这个查询需要访问documents X，Y和Z， 你可能在下一次查询就需要访问别documents。而一次把所有的值都加载并保存在内存 ， 比每次查询都去扫描倒排索引要更方便。
>
> JVM堆是一个有限制的资源需要聪明的使用。有许多现成的机制去限制fielddata对堆内存使用的影响。这些限制非常重要，因为滥用堆将会导致节点的不稳定（多亏缓慢的垃圾回收）或者甚至节点死亡（因为OutOfMemory异常）；但是垃圾回收时间过长，在垃圾回收期间，ES节点的性能就会大打折扣，查询就会非常缓慢，直到最后超时。

##### **如何设置堆大小**

> 对于环境变量 $ES_HEAP_SIZE 在设置Elasticsearch堆大小的时候有2个法则可以运用:
>
> 1) 不超过RAM的50%
> Lucene很好的利用了文件系统cache，文件系统cache是由内核管理的。如果没有足够的文件系统cache空间，性能就会变差;
>
> 2) 不超过32G
> 如果堆小于32GB，JVM能够使用压缩的指针，这会节省许多内存：每个指针就会使用4字节而不是8字节。把对内存从32GB增加到34GB将意味着你将有更少的内存可用，因为所有的指针占用了双倍的空间。同样，更大的堆，垃圾回收变得代价更大并且可能导致节点不稳定；这个限制主要是大内存对fielddata影响比较大。



