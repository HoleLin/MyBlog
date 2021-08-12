---
title: ElasticSearch(三)-常见配置以及配置文件详解
mermaid: true
date: 2021-08-12 11:14:17
cover: /img/cover/Elasticsearch.jpg
tags:
- 配置文件
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

#### Cluser配置

| 配置                                  | 含义                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `cluster.name: holelin_elasticsearch` | 所属的集群名，默认为 elasticsearch ，可自定义（最好给生产环境的ES集群改个名字，改名字的目的其实就是防止某台服务器加入了集群这种意外） |

#### Node配置

| 配置                                      | 含义                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `node.name: holelin_elasticsearch_node01` | 节点名，默认为 UUID前7个字符，可自定义                       |
| `node.master: true`                       | 指定该节点是否可以作为master节点，默认是true。ES集群默认是以第一个节点为master，如果该节点出故障就会重新选举master。 |
| `node.data: true`                         | 该节点是否存索引数据，允许该节点存储数据。默认true。         |
| `node.client: false`                      | 当该值设置为true时，node.master值自动设置为false，不参加master选举。 |
| `node.rack`                               | 无默认值，为节点添加自定义属性。                             |
| `node.max_local_storage_nodes: 1`         | （默认值：1） 设置能运行的节点数目，一般采用默认的1即可，因为我们一般也只在一台机子上部署一个节点。 |

#### Indices部分

| 配置                          | 含义                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `index.number_of_shards: 5`   | (默认值为5)  设置默认索引分片个数。<br />**需要注意:** "number_of_shards" 是索引创建后一次生成的,后续不可更改设置 "number_of_replicas" 是可以通过update-index-settings API实时修改设置。 |
| `index.number_of_replicas: 1` | (默认值为1）设置索引的副本个数                               |

* **Indices Circuit Breaker**

  * elasticsearch包含多个circuit breaker来避免操作的内存溢出。每个breaker都指定可以使用内存的限制。另外有一个父级breaker指定所有的breaker可以使用的总内存

  | 配置                          | 含义                                                         |
  | ----------------------------- | ------------------------------------------------------------ |
  | `indices.breaker.total.limit` | 所有breaker使用的内存值，默认值为 JVM 堆内存的70%，当内存达到最高值时会触发内存回收。 |

*  **Field Data Circuit Breaker**  

  * 允许elasticsearch预算待加载field的内存，防止field数据加载引发异常

  | 配置                               | 含义                                                         |
  | ---------------------------------- | ------------------------------------------------------------ |
  | indices.breaker.fielddata.limit    | field数据使用内存限制，默认为JVM 堆的60%。                   |
  | indices.breaker.fielddata.overhead | elasticsearch使用这个常数乘以所有fielddata的实际值作field的估算值。默认为 1.03。 |

*  **Request Circuit Breaker**

  * 请求断路器,允许elasticsearch防止每个请求的数据结构超过了一定量的内存

  | 配置                               | 含义                                                         |
  | ---------------------------------- | ------------------------------------------------------------ |
  | `indices.breaker.request.limit`    | request数量使用内存限制，默认为JVM堆的40%。                  |
  | `indices.breaker.request.overhead` | elasticsearch使用这个常数乘以所有request占用内存的实际值作为最后的估算值。默认为 1。 |

*  **Indices Fielddata Cache**

  * 字段数据缓存主要用于排序字段和计算聚合。将所有的字段值加载到内存中，以便提供基于文档快速访问这些值

  | 配置                                      | 含义                                                         |
  | ----------------------------------------- | ------------------------------------------------------------ |
  | `indices.fielddata.cache.size：unbounded` | 设置字段数据缓存的最大值，值可以设置为节点堆空间的百分比，例：30%，可以值绝对值，例：12g。默认为无限。 该设置是静态设置，必须配置到集群的每个节点。 |

*  **Indices Node Query Cache**

  * query cache负责缓存查询结果，每个节点都有一个查询缓存共享给所有的分片。缓存实现一个LRU驱逐策略：当缓存使用已满，最近最少使用的数据将被删除，来缓存新的数据。query cache只缓存过滤过的上下文

  | 配置                         | 含义                                                         |
  | ---------------------------- | ------------------------------------------------------------ |
  | `indices.queries.cache.size` | 查询请求缓存大小，默认为10%。也可以写为绝对值，例：512m。 该设置是静态设置，必须配置到集群的每个数据节点。 |

* **Indexing Buffer**

  * 索引缓冲区用于存储新索引的文档。缓冲区写满，缓冲区的文件才会写到硬盘。缓冲区划分给节点上的所有分片。
  * Indexing Buffer的配置是静态配置，必须配置都集群中的所有数据节点

  | 配置                                         | 含义                                                         |
  | -------------------------------------------- | ------------------------------------------------------------ |
  | `indices.memory.index_buffer_size`           | 允许配置百分比和字节大小的值。默认10%，节点总内存堆的10%用作索引缓冲区大小。 |
  | `indices.memory.min_index_buffer_size`       | 如果index_buffer_size被设置为一个百分比，这个设置可以指定一个最小值。默认为 48mb。 |
  | `indices.memory.max_index_buffer_size`       | 如果index_buffer_size被设置为一个百分比，这个设置可以指定一个最小值。默认为无限。 |
  | `indices.memory.min_shard_index_buffer_size` | 设置每个分片的最小索引缓冲区大小。默认为4mb。                |

* **Indices Shard Request Cache** 

  * 当一个搜索请求是对一个索引或者多个索引的时候，每一个分片都是进行它自己内容的搜索然后把结果返回到协调节点，然后把这些结果合并到一起统一对外提供。分片缓存模块缓存了这个分片的搜索结果。这使得搜索频率高的请求会立即返回。

  * **注意：**请求缓存只缓存查询条件 size=0的搜索，缓存的内容有hits.total, aggregations, suggestions，不缓存原始的hits。通过now查询的结果将不缓存。 缓存失效：只有在分片的数据实际上发生了变化的时候刷新分片缓存才会失效。刷新的时间间隔越长，缓存的数据越多，当缓存不够的时候，最少使用的数据将被删除

  * 缓存过期可以手工设置，例如:

    ```sh
    curl -XPOST 'localhost:9200/kimchy,elasticsearch/_cache/clear?request_cache=true'
    ```

  * 默认情况下缓存未启用，但在创建新的索引时可启用，例如:

    ```sh
    curl -XPUT localhost:9200/my_index -d'
    {
    　　"settings": {
    　　　　"index.requests.cache.enable": true
    　　}
    }'
    ```

  * 当然也可以通过动态参数配置来进行设置:

    ```sh
    curl -XPUT localhost:9200/my_index/_settings -d'
    { "index.requests.cache.enable": true }'
    ```

  * 每请求启用缓存，查询字符串参数request_cache可用于启用或禁用每个请求的缓存。例如:

    ```sh
    curl 'localhost:9200/my_index/_search?request_cache=true' -d'
    {
    　　"size": 0,
    　　"aggs": {
    　　　　"popular_colors": {
    　　　　　　"terms": {
    　　　　　　　　"field": "colors"
    　　　　　　}
    　　　　}
    　　}
    }'
    ```

  * 分片请求缓存是在节点级别进行管理的，并有一个默认的值是JVM堆内存大小的1%，可以通过配置文件进行修改。 例如： indices.requests.cache.size: 2%

    * 分片缓存大小的查看方式:`curl 'localhost:9200/_stats/request_cache?pretty&human'` 或者`curl 'localhost:9200/_nodes/stats/indices/request_cache?pretty&human'`

* **Indices Recovery**

* | 配置                                           | 含义                                                         |
  | ---------------------------------------------- | ------------------------------------------------------------ |
  | indices.recovery.concurrent_streams            | 限制从其它分片恢复数据时最大同时打开并发流的个数。默认为 3。 |
  | indices.recovery.concurrent_small_file_streams | 从其他的分片恢复时打开每个节点的小文件(小于5M)流的数目。默认为 2。 |
  | indices.recovery.file_chunk_size               | 默认为 512kb。                                               |
  | indices.recovery.translog_ops                  | 默认为 1000。                                                |
  | indices.recovery.translog_size                 | 默认为 512kb。                                               |
  | indices.recovery.compress                      | 恢复分片时，是否启用压缩。默认为 ``true``。                  |
  | indices.recovery.max_bytes_per_sec             | 限制从其它分片恢复数据时每秒的最大传输速度。默认为 40mb。    |

  **Indices TTL Interval**

  | 配置                  | 含义                                              |
  | --------------------- | ------------------------------------------------- |
  | indices.ttl.interval  | 允许设置多久过期的文件会被自动删除。默认值是60s。 |
  | indices.ttl.bulk_size | 设置批量删除请求的数量。默认值为1000。            |

#### Paths配置

| 配置                           | 含义                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| path.data: /path/to/data       | 数据目录 #注意：数据目录可以有多个，可以通过逗号分隔指定多个目录。一个索引数据只会放入一个目录中！！ |
| path.logs: /path/to/log        | 日志目录                                                     |
| path.plugins: /path/to/plugins | 插件目录                                                     |
| path.conf: /path/to/conf       | 配置文件存储位置。                                           |
| path.work: /path/to/work       | 临时文件的路径 。                                            |

#### Memory配置

| 配置                     | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Bootstrap.mlockall: true | （默认为false）锁住内存，当JVM进行内存转换的时候，es的性能会降低，所以可以使用这个属性锁住内存。同时也要允许elasticsearch的进程可以锁住内存，linux下可以通过`ulimit -l unlimited`命令，或者在/etc/sysconfig/elasticsearch文件中取消 MAX_LOCKED_MEMORY=unlimited 的注释即可。如果使用该配置则ES_HEAP_SIZE必须设置，设置为当前可用内存的50%，最大不能超过31G，默认配置最小为256M，最大为1G。 |

* 可以通过请求查看mlockall的值是否设定:`curl http://localhost:9200/_nodes/process?pretty`
* 如果mlockall的值是false，则设置失败。可能是由于elasticsearch的临时目录（/tmp）挂载的时候没有可执行权限。
  * 可以使用下面的命令来更改临时目录:`./bin/elasticsearch -Djna.tmpdir=/path/to/new/dir`

#### Network,Transport,HTTP配置

| 配置                            | 含义                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| `network.bind_host`             | 设置绑定的ip地址,可以是ipv4或ipv6的。                        |
| `network.publish_host`          | 设置其它节点和该节点交互的ip地址,如果不设置它会自动设置,值必须是个真实的ip地址。 |
| `network.host`                  | 同时设置bind_host和publish_host两个参数，值可以为网卡接口、127.0.0.1、私有地址以及公有地址。 |
| `http_port`                     | 接收http请求的绑定端口。可以为一个值或端口范围，如果是一个端口范围，节点将绑定到第一个可用端口。默认为：9200-9300。 |
| `transport.tcp.port`            | 节点通信的绑定端口。可以为一个值或端口范围，如果是一个端口范围，节点将绑定到第一个可用端口。默认为：9300-9400。 |
| `transport.tcp.connect_timeout` | 套接字连接超时设置，默认为 30s。                             |
| `transport.tcp.compress`        | 设置为true启用节点之间传输的压缩（LZF），默认为false。       |
| `transport.ping_schedule`       | 定时发送ping消息保持连接，默认transport客户端为5s，其他为-1（禁用）。 |
| `httpd.enabled`                 | 是否使用http协议提供服务。默认为：true（开启）。             |
| `http.max_content_length`       | 最大http请求内容。默认为100MB。如果设置超过100MB，将会被MAX_VALUE重置为100MB。 |
| `http.max_initial_line_length`  | http的url的最大长度。默认为：4kb。                           |
| `http.max_header_size`          | http中header的最大值。默认为8kb。                            |
| `http.compression`              | 支持压缩（Accept-Encoding）。默认为：false。                 |
| `http.compression_level`        | 定义压缩等级。默认为：6。                                    |
| `http.cors.enabled`             | 启用或禁用跨域资源共享。默认为：false。                      |
| `http.cors.allow-origin`        | 启用跨域资源共享后，默认没有源站被允许。在//中填写域名支持正则，例如 /https?:\/\/localhost(:[0-9]+)?/。 * 是有效的值，但是开放任何域名的跨域请求被认为是有安全风险的elasticsearch实例。 |
| `http.cors.max-age`             | 浏览器发送‘preflight’OPTIONS-request 来确定CORS设置。max-age 定义缓存的时间。默认为：1728000 （20天）。 |
| `http.cors.allow-methods`       | 允许的http方法。默认为OPTIONS、HEAD、GET、POST、PUT、DELETE。 |
| `http.cors.allow-headers`       | 允许的header。默认 X-Requested-With, Content-Type, Content-Length。 |
| `http.cors.allow-credentials`   | 是否允许返回Access-Control-Allow-Credentials头部。默认为：false。 |
| `http.detailed_errors.enabled`  | 启用或禁用输出详细的错误信息和堆栈跟踪响应输出。默认为：true。 |
| `http.pipelining`               | 启用或禁用http管线化。默认为：true。                         |
| `http.pipelining.max_events`    | 一个http连接关闭之前最大内存中的时间队列。默认为：10000。    |

#### Discovery配置

| 配置                                                | 含义                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| `discovery.zen.minimum_master_nodes: 3`             | 预防脑裂（split brain）通过配置大多数节点（总节点数/2+1）。默认为3。 |
| `discovery.zen.ping.multicast.enabled: false`       | 设置是否打开组播发现节点。默认false。                        |
| `discovery.zen.ping.unicast.hosts`                  | 单播发现所使用的主机列表，可以设置一个属组，或者以逗号分隔。每个值格式为 host:port 或 host（端口默认为：9300）。默认为 127.0.0.1，[::1]。 |
| `discovery.zen.ping.timeout: 3s`                    | 设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。 |
| `discovery.zen.join_timeout`                        | 节点加入到集群中后，发送请求到master的超时时间，默认值为ping.timeout的20倍。 |
| `discovery.zen.master_election.filter_client：true` | 当值为true时，所有客户端节点（node.client：true或node.date，node.master值都为false）将不参加master选举。默认值为：true。 |
| `discovery.zen.master_election.filter_data：false`  | 当值为true时，不合格的master节点（node.data：true和node.master：false）将不参加选举。默认值为：false。 |
| `discovery.zen.fd.ping_interval`                    | 发送ping监测的时间间隔。默认为：1s。                         |
| `discovery.zen.fd.ping_timeout`                     | ping的响应超时时间。默认为30s。                              |
| `discovery.zen.fd.ping_retries`                     | ping监测失败、超时的次数后，节点连接失败。默认为3。          |
| `discovery.zen.publish_timeout`                     | 通过集群api动态更新设置的超时时间，默认为30s。               |
| `discovery.zen.no_master_block`                     | 设置无master时，哪些操作将被拒绝。all 所有节点的读、写操作都将被拒绝。write 写操作将被拒绝，可以读取最后已知的集群配置。默认为：write。 |

#### Gateway配置

| 配置                                 | 含义                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `gateway.expected_nodes: 0`          | 设置这个集群中节点的数量，默认为0，一旦这N个节点启动，就会立即进行数据恢复。 |
| `gateway.expected_master_nodes`      | 设置这个集群中主节点的数量，默认为0，一旦这N个节点启动，就会立即进行数据恢复。 |
| `gateway.expected_data_nodes`        | 设置这个集群中数据节点的数量，默认为0，一旦这N个节点启动，就会立即进行数据恢复。 |
| `gateway.recover_after_time: 5m`     | 设置初始化数据恢复进程的超时时间，默认是5分钟。              |
| `gateway.recover_after_nodes`        | 设置集群中N个节点启动时进行数据恢复。                        |
| `gateway.recover_after_master_nodes` | 设置集群中N个主节点启动时进行数据恢复。                      |
| `gateway.recover_after_data_nodes`   | 设置集群中N个数据节点启动时进行数据恢复。                    |

