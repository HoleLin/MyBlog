---
title: Kafka命令实操
date: 2021-05-25 22:46:55
index_img: /img/cover/Kafka.jpg
cover: /img/cover/Kafka.jpg
tags:
- 中间件
- Kafka
categories: 
- Kafka
mermaid: true
---
### Kafka命令实操

#### 1.启动 broker

```shell
./bin/kafka-server-start.sh -daemon <path>/server.properties
```

> `./bin/kafka-server-start.sh <path>/server.properties & `
>
> 有问题,虽然通过添加&符号使该命令在后台运行,但当用户从回话登出(log out)时该进程会被自动kill掉,从而导致broker关闭
>
> 建议使用
>
> `nohup .bin/kafka-server-start.sh  <path>/server.properties  &`

* 验证broker是否正常启动可以通过`server.log`日志文件查看输出项

#### 2.关闭broker

```shell
bin/kafka-server-stop.sh
或者
1. 使用jps命令直接获取Kafka的PID
2. 若没有安装JDK,则运行`ps ax|grep -i 'kafka\.Kafka'|grep java|grep -v grep|awk '{print $1}'`命令获取Kafka的PID
3. 运行`kill -s TTERM $PID`
```

#### 3. 设置JMX端口

* 目的: 用于实时监控集群运行的健康程度

  ```shell
  JMX_PORT=9997 ./bin/kafka-server-start.sh <path>/server.properties
  
  后台运行
  export JMX_PORT=9997 ./bin/kafka-server-start.sh -daemon <path>/server.properties
  ```

#### 4. 升级broker版本

* 更新broker之间通信版本和消息版本

  * 向所有broker的server.properties中增加下面两行:

    ```properties
    inter.broker.protocol.version=<current version>
    log.message.format.version=<current version>
    ```

* 依次更新代码,重启有broker

  * 下载新版本的Kafka二进制包,覆盖已有的目录,然后依次重启所有broker

* 再次更新broker间通信版本和消息版本

  ```properties
  inter.broker.protocol.version=<new version>
  log.message.format.version=<new version>
  ```

* 重启broker

#### 5.Topic管理

* Kafka创建topic的途径当前总共有如下4种:
  * 通过`kafka-topic.sh(bat)`命令行工具创建
  * 通过显示发送`CreateTopicsRequest`请求创建topic
  * 通过发送`MetadataRequest`请求且broker端设置`auto.create.topic.enable`为true
  * 通过向Zookeeper的/broker/topics路径下写入以topic命名的子节点

* kafka-topics脚本命令行参数列表

  ```
  --zookeeper: (必填项)指定连接的zookeeper信息
  --partitions<分区数>: 创建或修改topic时指定分区数
  --replication-factor: 指定副本因子
  --topic: 指定topic的名称
  --alter: 用于修改topic的信息,如分区数,副本因子
  --config <key=value>: 设置topic级别的参数,如clean.policy等
  --create: 创建topic
  --delete: 删除topic
  --delete-config: 删除topic级别的参数
  --describe: 列出topic详情
  --disable-rack-aware: 创建topic时不考虑机架信息
  --force: 无效参数,当前未使用
  --help: 打印帮助信息
  --if--exists: 若设置,脚本只对已存在的topic执行操作
  --list: 列出集群当前所有topic
  --replica-assignment: 手动指定分区分配CSV方案,副本之间使用冒号分割.比如执行双分区方案为0:1:2,3:4:5,表示分区1的3个副本在broker0,1,2上,分区2在broker3,4,5上
  --topic-with-overrides: 展示topic详情时不显示具体的分区信息
  --unavailable-partitions: 只显示topic不可用的分区(即没有leader的分区)
  --under-replicated-partitions: 只显示副本数不足的分区信息
  ```

* 删除topic有3种方式

  * 使用kafka-topics脚本
  * 构造`DeleteTopicsRequest`请求
  * 直接向Zookeeper的/admin/delete_topics下写入子节点(不推荐)

  > 执行kafka-topics.sh --delete命令后,后台会开启一个异步删除的任务,所以用户多等待一段时间后才能观察到topic数据被完全删除.
  >
  > **首先包确保broker端参数delete.topic.enable被设置为true**

* 查看topic列表

  ```shell
  ./bin/kafka-topics.sh --zookeeper localhost:2181 --list
  ```

* 查看topic详情

  ```shell
  ./bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic <topic_name>
  ```

* 修改topic

  ```shell
  // 为topic增加分区,修改topic分区数为10
  ./bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 10 --topic <topic_name>
  ```

  目前只能增加partitions,减少会抛出异常

  `org.apache.kafka.common.errors.InvalidPartitionsException: The number of partitions for a topic can only be increased. `

  * 使用kafka-configs脚本

    ```properties
    --add-config k1=v1,k2=v2: 设置topic级别参数
    --alter: 修改topic或者其他实体(entity)类型
    --delete-config: 删除指定的参数
    --entity-default: 主要与配额(quota)设置有关,用于Kafka配置的默认配额值
    --describe: 列出给定实体类型的参数详情
    --entity-name: 指定实体名称,若是topic类型,则是topic名称
    --entity-type: 指定实体类型,总共有4类实体类型:users,topics,clients,brokers
    --help: 打印帮助信息
    --zookeeper: (必填项)指定连接的Zookeeper信息
    ```

* topic配置

  ```properties
  cleanup.policy: 指定topic的留存策略,可以是compact,delete或者同时指定两者
  compression.type: 指定该topic消息的压缩类型
  max.message.bytes: 指定broker端能够接受该topic消息的最大长度
  min.insync.replicas: 指定ISR中需要接收topic消息的最少broker数,与producer端参数acks=-1配合使用
  preallocate: 是否为该topic的日志文件提前分配存储空间
  retention.ms: 指定持有该topic单个分区消息最长时间
  segment.byetes: 指定该topic日志段文件的大小
  unclean.leader.election.enable: 是否topic启用unclean领导者选举
  ```

* 查看topic配置

  * 通过`kafka-topics.sh`脚本

    ```shell
    ./bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic <topic_name>
    ```

  * 通过`kafka-configs.sh`脚本

    ```shell
    ./bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type topics --entity-name <topic_name>
    ```

* 删除topic配置

  ```shell
  ./bin/kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type topics --entity-name <topic_name> --delete-config <config_name>
  ```

#### 6.consumer相关管理

* 查询消费组`kafka-consumer-groups.sh`

  ```
  --bootstrap-server: 指定kafka集群的broker列表,CSV格式(新版使用)
  --list: 列出集群当前所有消费组
  --describe: 查询消费组详情(包括消费滞后情况等)
  --group: 指定消费组名称
  --zookeeper: 指定Zookeeper连接信息(老版使用)
  --reset-offsets: 重新设置消费组位移
  ```

* 查看消费者组信息

  ```shell
  ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093 --group <group_name> --describe
  Consumer group 'test-holelin' has no active members.
  
  GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
  test-holelin    holelin         0          9990            50008           40018           -               -               -
  test-holelin    holelin         1          0               0               0               -               -               -
  TOPIC: 表示消费者组消费了哪些topic.
  PARTITION: 表示该消费者组消费的是哪个分区.
  CURRENT-OFFSET: 表示消费者组最新消费的位移值.
  LOG-END-OFFSET: 表示topic所有分区当前的日志终端位移值.
  LAG: 表示消费滞后进度.该值等于LOG-END-OFFSET-CURRENT-OFFSET,通常是但与或等于0的整数.若该值接近LOG-END-OFFSET则表明该消费者组消费滞后严重.特别需要注意的是,若该值小于0,则通常表明存在消费数据丢失的情况(即有些消息未被消费就被consumer直接跳过了,若出现这种情况,则需要确认broker端参数`unclean.leader.election.enable`的值是否被设置为了true,并进一步研究是否可能出现因unclean领导者选举而造成的数据丢失).
  CONSUMER-ID: 表示consumer的ID,通常是Kafka自动生成的.如果该列没有值,通常表明此consumer目前尚未处于运行中.
  HOST: 表示consumer所在的broker主机信息.如果该列没有值,通常表明此consumer目前尚未处于运行中.
  CLIENT-ID: 表示用户指定或者系统自动生成的一个标识consumer所在客户端的ID,该ID通常用于定位和调试问题.
  ```

* 重设消费者组位移

  * 流程主要分3步:
    * 确定消费者组待重设topic集合
    * 确定位移重设策略
    * 确定执行方案
  * 确定消费者组下topic的作用域,当前支持3中作用域
    * `--all-topics`: 为消费者组下所有topic的所有分区调整位移;
    * `--topic t1,--topic t2`:  为指定的若干个topic的所有分区调整位移;
    * `--topic t1:0,1,2`:  为topic的指定分区调整位移;
  * 确定topic作用域之后,就是确定位移重设策略.当前支持如下8种设置规则
    * `--to-earliest`:  把位移调整到分区当前最早位移处;
    * `--to-latest`:  把位移调整到分区当前最新位移处;
    * `--to-current`:  把位移调整到分区当前位移处;
    * `--to-offset<offset>`:  把位移调整到指定位移处;
    * `--shift-by N`: 把位移调整到当前位移+N.N可以是负值;
    * `--to-datetime<datetime>`: 把位移调整到大于给定时间的最早位移处.datetime格式是yyyy-MM-ddTHH:mm:ss.xxx 比如2017-08-04T00:00:00.000;
    * `--by-duration<duration>`:  把位移调整到距离当前时间指定间隔的位移处.duration格式是PnDTnHnMnS,比如PT0H5M0S;
    * `--from-file<file>`:  从CSV文件中读取位移调整策略;
  * 确定执行方案,当前支持如下3种方案
    * 不加任何参数: 只是打印位移调整方案,不实际执行;
    * `--execute`:  执行真正的位移调整;
    * `--export`:  把位移调整方案保存成CSV格式并输出到控制,方便用户保存成CSV文件,供后续结合--from-file参数使用;

* 删除消费者组

  ```shell
  ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete -group <group_name>
  Deletion of requested consumer groups ('xxxx') was successful.
  ```

  * 新版消费者组过期数据的移除完全不需要用户操作,Kafka会定期地移除过期消费者组的数据,`broker`端参数有个`offsets.retention.minutes`参数,默认为1440分钟(即1天),该参数控制Kafka何时移除inactive消费者组位移信息.

#### 7. topic分区管理

* preferred leader选举

  > 在一个Kafka集群中,broker服务器宕机或者崩溃是不可避免的.一旦发生这种情况,该broker上的那些leader副本将变为不可用,因此就必然要求Kafka把这些分区的leader转移到其他的broker上.即使崩溃broker重启回来,其上的副本也只能作为follower副本加入ISR中,不能再对外提供服务.
  >
  > 随着集群的不断运行,这种leader的不均衡现象开始出现,即集群中的一小部分broker上承载了大量的分区leader副本.

* Kafka提供了两种方式帮助用户把指定分区的leader调整回他们的preffered replica.这个过程被称为preferred leader选举.

  * 第一种方式是使用Kafka自带`kafka-preferred-replica-election.sh`

    ```properties
    --zookeeper: 指定Zookeeper连接信息
    --path-to-json-file: 指定json文件路径,该文件包含了要为哪些分区执行preferred leader选举.也可不指定该参数,若不指定则表明为集群中所有分区都指定preferred leader选举
    ```

  * Kafka还提供了一个broker端参数`auto.leader.rebalance.enable` 默认值为true,表明每台broker启动后都会在后台自动地定期执行preferred leader选举.与之关联的还有broker端参数`leader.imbalance.check.interal.seconds`:  控制阶段性操作的时间间隔,当前默认值是300秒,即Kafka每5分钟就会尝试在后台运行一个preferred leader选举.

    `leader.imbalance.per.broker.percentage`: 用于确定需要执行preferred leader选举的目标分区,当前默认值为10,表示若broker上leader不均衡程度超过了10%,则Kafka需要为该broker上的分区执行preferred leader选举.Kafka计算不均衡程度的逻辑实际上非常简单--该broker上的leader不是preferred replica的分区数/broker上总的分区数.

* 分区重分配



​	

