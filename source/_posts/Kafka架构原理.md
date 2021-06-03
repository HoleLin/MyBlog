---
title: Kafka架构原理
date: 2021-05-25 22:46:55
index_img: /img/cover/Kafka.jpg
tags:
- 中间件
- Kafka
categories: 
- Java
- Kafka
mermaid: true
---
## Kafka架构原理

* **Producer：** 生产者，发送消息的一方。生产者负责创建消息，然后将其发送到 Kafka。
* **Consumer：** 消费者，接受消息的一方。消费者连接到 Kafka 上并接收消息，进而进行相应的业务逻辑处理。
* **Consumer Group：** 一个消费者组可以包含一个或多个消费者。使用多分区 + 多消费者方式可以极大提高数据下游的处理速度，同一消费组中的消费者不会重复消费消息，同样的，不同消费组中的消费者消息消息时互不影响。Kafka 就是通过消费组的方式来实现消息 P2P 模式和广播模式。
* **Broker：** 服务代理节点。Broker 是 Kafka 的服务节点，即 Kafka 的服务器。
* **Topic：** Kafka 中的消息以 Topic 为单位进行划分，生产者将消息发送到特定的 Topic，而消费者负责订阅 Topic 的消息并进行消费。
* **Partition：** Topic 是一个逻辑的概念，它可以细分为多个分区，每个分区只属于单个主题。同一个主题下不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。
* **Offset：** offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说，Kafka 保证的是分区有序性而不是主题有序性。
* **Replication：** 副本，是 Kafka 保证数据高可用的方式，Kafka 同一 Partition 的数据可以在多 Broker 上存在多个副本，通常只有主副本对外提供读写服务，当主副本所在 broker 崩溃或发生网络异常，Kafka 会在 Controller 的管理下会重新选择新的 Leader 副本对外提供读写服务。
* **Record：** 实际写入 Kafka 中并可以被读取的消息记录。每个 record 包含了 key、value 和 timestamp。

#### **Topic and Logs**

> Segment的常用配置有：
>
> ```
> #server.properties
> 
> #segment文件的大小，默认为 1G
> log.segment.bytes=1024*1024*1024
> #滚动生成新的segment文件的最大时长
> log.roll.hours=24*7
> #segment文件保留的最大时长，超时将被删除
> log.retention.hours=24*7
> ```
>
> Segment的常用配置有：
>
> ```
> #server.properties
> #segment文件的大小，默认为 1G
> log.segment.bytes=1024*1024*1024
> #滚动生成新的segment文件的最大时长
> log.roll.hours=24*7
> #segment文件保留的最大时长，超时将被删除
> log.retention.hours=24*7
> ```
>
> Partition 目录下包括了数据文件和索引文件，下图是某个 Partition 的目录结构：
>
> ![图片](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrh59mjySyeAibIDBS76yvNvl0aibOzrbtSG2Iooum8sPUQgggHOOJgeibDBib4gUDL7aczaXc6SU2HKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> Index 采用稀疏存储的方式，它不会为每一条 Message 都建立索引，而是每隔一定的字节数建立一条索引，避免索引文件占用过多的空间。
>
> 缺点是没有建立索引的 Offset 不能一次定位到 Message 的位置，需要做一次顺序扫描，但是扫描的范围很小。
>
> 索引包含两个部分（均为 4 个字节的数字），分别为相对 Offset 和 Position。
>
> 相对 Offset 表示 Segment 文件中的 Offset，Position 表示 Message 在数据文件中的位置。
>
> **总结：**Kafka 的 Message 存储采用了分区（Partition），磁盘顺序读写，分段（LogSegment）和稀疏索引这几个手段来达到高效性。

####  Partition and Replica

> 一个 Topic 物理上分为多个 Partition，位于不同的 Broker 上。如果没有 Replica，一旦 Broker 宕机，其上所有的 Patition 将不可用。
>
> 每个 Partition 可以有多个Replica（对应server.properties/default.replication.factor），分配到不同的 Broker 上。
>
> 其中有一个 Leader 负责读写，处理来自 Producer 和 Consumer 的请求；其他作为 Follower 从 Leader Pull 消息，保持与 Leader 的同步。
>
> 如何分配 Partition 和 Replica 到 Broker 上？步骤如下：
>
> - 将所有 Broker（假设共 n 个 Broker）和待分配的 Partition 排序。
> - 将第 i 个 Partition 分配到第（i mod n）个 Broker 上。
> - 将第 i 个 Partition 的第 j 个 Replica 分配到第（(i + j) mode n）个 Broker 上。
>
> 根据上面的分配规则，若 Replica 的数量大于 Broker 的数量，必定会有两个相同的 Replica 分配到同一个 Broker 上，产生冗余。因此 Replica 的数量应该小于或等于 Broker 的数量。

#### Leader 选举

> Kafka 在 Zookeeper 中（/brokers/topics/[topic]/partitions/[partition]/state）动态维护了一个 ISR（in-sync replicas）。
>
> ISR 里面的所有 Replica 都"跟上"了 Leader，Controller 将会从 ISR 里选一个做 Leader。
>
> 具体流程如下：
>
> - Controller 在 Zookeeper 的 /brokers/ids/[brokerId] 节点注册 Watcher，当 Broker 宕机时 Zookeeper 会 Fire Watch。
> - Controller 从 /brokers/ids 节点读取可用 Broker。
> - Controller 决定 set_p，该集合包含宕机 Broker 上的所有 Partition。
> - 对 set_p 中的每一个 Partition，从/brokers/topics/[topic]/partitions/[partition]/state 节点读取 ISR，决定新 Leader，将新 Leader、ISR、controller_epoch 和 leader_epoch 等信息写入 State 节点。
> - 通过 RPC 向相关 Broker 发送 leaderAndISRRequest 命令。
>
> 当 ISR 为空时，会选一个 Replica（不一定是 ISR 成员）作为 Leader；当所有的 Replica 都歇菜了，会等任意一个 Replica 复活，将其作为 Leader。
>
> ISR（同步列表）中的 Follower 都"跟上"了Leader，"跟上"并不表示完全一致，它由 server.properties/replica.lag.time.max.ms 配置。
>
> 表示 Leader 等待 Follower 同步消息的最大时间，如果超时，Leader 将 Follower 移除 ISR。配置项 replica.lag.max.messages 已经移除。

#### **Replica 同步**

> Kafka 通过"拉模式"同步消息，即 Follower 从 Leader 批量拉取数据来同步。
>
> 具体的可靠性，是由生产者（根据配置项 producer.properties/acks）来决定的。

#### Producer 如何发送消息？

> Producer 首先将消息封装进一个 ProducerRecord 实例中。
>
> ![图片](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrh59mjySyeAibIDBS76yvNvzsGftAaic5Ma5ib52VZa0ia0oPjDFTQW9Q0Xdp5ZdBcWbLPd7jh72R0zA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> #### 消息路由：
>
> - 发送消息时如果指定了 Partition，则直接使用。
> - 如果指定了 Key，则对 Key 进行哈希，选出一个 Partition。这个 Hash（即分区机制）由 producer.properties/partitioner.class 指定的类实现，这个路由类需要实现 Partitioner 接口。
> - 如果都未指定，通过 Round-Robin 来选 Partition。
>
> 消息并不会立即发送，而是先进行序列化后，发送给 Partitioner，也就是上面提到的 Hash 函数，由 Partitioner 确定目标分区后，发送到一块内存缓冲区中（发送队列）。
>
> Producer 的另一个工作线程（即 Sender 线程），则负责实时地从该缓冲区中提取出准备好的消息封装到一个批次内，统一发送到对应的 Broker 中。
>
> 其过程大致是这样的：
>
> ![图片](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQrh59mjySyeAibIDBS76yvNvickyJHjEymkvZtsqqr6DTVXDESLCFbsBVul5EOS09CXGZRFO3FoFibZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### Consumer 如何消费消息？

> 每个 Consumer 都划归到一个逻辑 Consumer Group 中，一个 Partition 只能被同一个 Consumer Group 中的一个 Consumer 消费，但可以被不同的 Consumer Group 消费。
>
> 若 Topic 的 Partition 数量为 p，Consumer Group 中订阅此 Topic 的 Consumer 数量为 c， 则：
>
> ```
> p < c: 会有 c - p 个 consumer闲置，造成浪费
> p > c: 一个 consumer 对应多个 partition 
> p = c: 一个 consumer 对应一个 partition
> ```
>
> 应该合理分配 Consumer 和 Partition 的数量，避免造成资源倾斜，最好 Partiton 数目是 Consumer 数目的整数倍。
>
> #### **①如何将 Partition 分配给 Consumer**
>
> 生产过程中 Broker 要分配 Partition，消费过程这里，也要分配 Partition 给消费者。
>
> 类似 Broker 中选了一个 Controller 出来，消费也要从 Broker 中选一个 Coordinator，用于分配 Partition。
>
> 当 Partition 或 Consumer 数量发生变化时，比如增加 Consumer，减少 Consumer（主动或被动），增加 Partition，都会进行 Rebalance。
>
> 其过程如下：
>
> - Consumer 给 Coordinator 发送 JoinGroupRequest 请求。这时其他 Consumer 发 Heartbeat 请求过来时，Coordinator 会告诉他们，要 Rebalance了。其他 Consumer 也发送 JoinGroupRequest 请求。
> - Coordinator 在 Consumer 中选出一个 Leader，其他作为 Follower，通知给各个 Consumer，对于 Leader，还会把 Follower 的 Metadata 带给它。
> - Consumer Leader 根据 Consumer Metadata 重新分配 Partition。
> - Consumer 向 Coordinator 发送 SyncGroupRequest，其中 Leader 的 SyncGroupRequest 会包含分配的情况。Coordinator 回包，把分配的情况告诉 Consumer，包括 Leader。
>
> #### **②Consumer Fetch Message**
>
> Consumer 采用"拉模式"消费消息，这样 Consumer 可以自行决定消费的行为。
>
> Consumer 调用 Poll（duration）从服务器拉取消息。拉取消息的具体行为由下面的配置项决定
>
> ```
> #consumer.properties
> 
> #消费者最多 poll 多少个 record
> max.poll.records=500
> 
> #消费者 poll 时 partition 返回的最大数据量
> max.partition.fetch.bytes=1048576
> 
> #Consumer 最大 poll 间隔
> #超过此值服务器会认为此 consumer failed 
> #并将此 consumer 踢出对应的 consumer group 
> max.poll.interval.ms=300000
> ```
>
> 在 Partition 中，每个消息都有一个 Offset。新消息会被写到 Partition 末尾（最新的一个 Segment 文件末尾）， 每个 Partition 上的消息是顺序消费的，不同的 Partition 之间消息的消费顺序是不确定的。
>
> 若一个 Consumer 消费多个 Partition, 则各个 Partition 之前消费顺序是不确定的，但在每个 Partition 上是顺序消费。
>
> 若来自不同 Consumer Group 的多个 Consumer 消费同一个 Partition，则各个 Consumer 之间的消费互不影响，每个 Consumer 都会有自己的 Offset。
>
> Consumer A 和 Consumer B 属于不同的 Consumer Group。Cosumer A 读取到 Offset=9， Consumer B 读取到 Offset=11，这个值表示下次读取的位置。
>
> 也就是说 Consumer A 已经读取了 Offset 为 0~8 的消息，Consumer B 已经读取了 Offset 为 0～10 的消息。
>
> 下次从 Offset=9 开始读取的 Consumer 并不一定还是 Consumer A 因为可能发生 Rebalance。

#### Offset 如何保存？

> Consumer 消费 Partition 时，需要保存 Offset 记录当前消费位置。
>
> Offset 可以选择自动提交或调用 Consumer 的 commitSync() 或 commitAsync() 手动提交，相关配置为：
>
> ```
> #是否自动提交 offset
> enable.auto.commit=true
> 
> #自动提交间隔。enable.auto.commit=true 时有效
> auto.commit.interval.ms=5000
> ```
>
> Offset 保存在名叫 __consumeroffsets 的 Topic 中。写消息的 Key 由 GroupId、Topic、Partition 组成，Value 是 Offset。
>
> 一般情况下，每个 Key 的 Offset 都是缓存在内存中，查询的时候不用遍历 Partition，如果没有缓存，第一次就会遍历 Partition 建立缓存，然后查询返回。
>
> __consumeroffsets 的 Partition 数量由下面的 Server 配置决定：
>
> ```
> offsets.topic.num.partitions=50
> ```
>
> Offset 保存在哪个分区上，即 __consumeroffsets 的分区机制，可以表示为：
>
> ```
> groupId.hashCode() mode groupMetadataTopicPartitionCount
> ```
>
> groupMetadataTopicPartitionCount 是上面配置的分区数。因为一个 Partition 只能被同一个 Consumer Group 的一个 Consumer 消费，因此可以用 GroupId 表示此 Consumer 消费 Offeset 所在分区。

#### 消息系统可能遇到哪些问题？

> Kafka 支持 3 种消息投递语义：
>
> - **at most once：**最多一次，消息可能会丢失，但不会重复
>   获取数据 -> commit offset -> 业务处理
> - **at least once：**最少一次，消息不会丢失，可能会重复
>   获取数据 -> 业务处理 -> commit offset。
> - **exactly once：**只且一次，消息不丢失不重复，只且消费一次（0.11 中实现，仅限于下游也是 Kafka）
>
> #### **①如何保证消息不被重复消费？（消息的幂等性）**
>
> 对于更新操作，天然具有幂等性。对于新增操作，可以给每条消息一个唯一的 id，处理前判断是否被处理过。这个 id 可以存储在 Redis 中，如果是写数据库可以用主键约束。
>
> #### **②如何保证消息的可靠性传输？（消息丢失的问题）**
>
> 根据 Kafka 架构，有三个地方可能丢失消息：Consumer，Producer 和  Server。
>
> **消费端弄丢了数据：**当 server.properties/enable.auto.commit 设置为 True 的时候，Kafka 会先 Commit Offset 再处理消息，如果这时候出现异常，这条消息就丢失了。
>
> 因此可以关闭自动提交 Offset，在处理完成后手动提交 Offset，这样可以保证消息不丢失；但是如果提交 Offset 失败，可能导致重复消费的问题， 这时保证幂等性即可。
>
> **Kafka 弄丢了消息：**如果某个 Broker 不小心挂了，此时若 Replica 只有一个，Broker 上的消息就丢失了。
>
> 若 Replica>1，给 Leader 重新选一个 Follower 作为新的 Leader，如果 Follower 还有些消息没有同步，这部分消息便丢失了。
>
> 可以进行如下配置，避免上面的问题：
>
> - **给 Topic 设置 replication.factor 参数：**这个值必须大于 1，要求每个 Partition 必须有至少 2 个副本。
> - **在 Kafka 服务端设置 min.insync.replicas 参数：**这个值必须大于 1，这个是要求一个 Leader 至少感知到有至少一个 Follower 还跟自己保持联系，没掉队，这样才能确保 Leader 挂了还有一个 Follower 吧。
> - **在 Producer 端设置 acks=all：**这个是要求每条数据，必须是写入所有 Replica 之后，才能认为是写成功了。
> - 在 Producer 端设置 retries=MAX（很大很大很大的一个值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试，卡在这里了。
>
> **Producer弄丢了消息：**在 Producer 端设置 acks=all，保证所有的 ISR 都同步了消息才认为写入成功。
>
> #### ** ③如何保证消息的顺序性？**
>
> Kafka 中 Partition 上的消息是顺序的，可以将需要顺序消费的消息发送到同一个 Partition 上，用单个 Consumer 消费。

## Zookeeper

* **Controller**
  
  * Controller 是从 Broker 中选举出来的，负责分区 Leader 和 Follower 的管理。当某个分区的 leader 副本发生故障时，由 Controller 负责为该分区选举新的 leader 副本。当检测到某个分区的 ISR(In-Sync Replica)集合发生变化时，由控制器负责通知所有 broker 更新其元数据信息。当使用`kafka-topics.sh`脚本为某个 topic 增加分区数量时，同样还是由控制器负责分区的重新分配。
  * Kafka 中 Contorller 的选举的工作依赖于 Zookeeper，成功竞选为控制器的 broker 会在 Zookeeper 中创建`/controller`这个临时（EPHEMERAL）节点。
  * **选举过程**
    * Broker启动的时候尝试去读取/controller节点的brokerid的值;
    * 如果brokerid的值不等于-1,则表明已经有其他的Broker成功成为Controller节点,当前Broker主动放弃竞选;
    * 如果不存在/controller节点,或者brokerid数值异常,当前Broker尝试去创建/controller这个节点,此时也有可能其他broker同时去尝试创建这个节点;
    * 只有创建成功的那个broker才会成为控制器,而创建失败的broker则表示竞选失败.每个broker都会在内存中保存当前控制器的brokerid值,这个值可以标识为activeControllerId.
  * **实现**
    * Controller读取Zookeeper中节点数据,初始化上下文(Controller Context),并管理节点变化,变更上下文,同事也需要将这些变更信息同步到其他普通的broker节点中.
    * Controller通过定时任务,或者监听器模式获取Zookeeper信息,事件监听会更新上下文信息,Contronller内部也采用生产者-消费者实现模式,Controller将Zookeeper的变动通过事件的方式发送给事件队列,队列就是一个LinkedBlockQueue,事件消费者线程组通过消费时间,将相应的事件同步到各个Broker节点.
  * **职责**
    * Controller被选举出来,作为整个Broker集群的管理者,管理所有的集群信息和员数据信息.主要责任包括:
      * 处理 Broker 节点的上线和下线，包括自然下线、宕机和网络不可达导致的集群变动，Controller 需要及时更新集群元数据，并将集群变化通知到所有的 Broker 集群节点；
      * 创建 Topic 或者 Topic 扩容分区，Controller 需要负责分区副本的分配工作，并主导 Topic 分区副本的 Leader 选举。
      * 管理集群中所有的副本和分区的状态机，监听状态机变化事件，并作出相应的处理。Kafka 分区和副本数据采用状态机的方式管理，分区和副本的变化都在状态机内会引起状态机状态的变更，从而触发相应的变化事件。
  
* **分区状态机**

  * PartitionStateChange:管理Topic的分区,它有以下4中状态:

    * NonExistentPartition：该状态表示分区没有被创建过或创建后被删除了。
    * NewPartition：分区刚创建后，处于这个状态。此状态下分区已经分配了副本，但是还没有选举 leader，也没有 ISR 列表。
    * OnlinePartition：一旦这个分区的 leader 被选举出来，将处于这个状态。
    * OfflinePartition：当分区的 leader 宕机，转移到这个状态。

  * 状态之间的切换

    ```mermaid
    graph TB
    NonExistentPartition -->|从zk加载分配的replicas|NewPartition 
    NewPartition-->|1. 将第一个replica作为leader,其余ISR,更新数据到zk,2. 发送LeaderAndLsr请求给存活的replica,发送UpdateMetaData请求给存活的Broker| OnlinePartition
    OnlinePartition-->|标记分区下线|OfflinePartition
    OfflinePartition-->|1. 选举新的leader更新zk,2. 发送LeaderAndLsr请求给存活的replica,发送UpdateMetaData请求给存活的Broker|OnlinePartition
    OfflinePartition-->|标记分区不存在|NonExistentPartition
    ```

* **副本状态机**

  * ReplicaStateChange，副本状态，管理分区副本信息，它也有 4 种状态：

    * NewReplica: 创建 topic 和分区分配后创建 replicas，此时，replica 只能获取到成为 follower 状态变化请求。
    * OnlineReplica: 当 replica 成为 parition 的 assingned replicas 时，其状态变为 OnlineReplica, 即一个有效的 OnlineReplica。
    * OfflineReplica: 当一个 replica 下线，进入此状态，这一般发生在 broker 宕机的情况下；
    * NonExistentReplica: Replica 成功删除后，replica 进入 NonExistentReplica 状态。

  * 状态之间的切换

    ```mermaid
    graph TD
    NonExistentReplica -->|向新的replica发送LeaderAndLsr请求,告诉它当前Leader和LSR信息并为分区发送UpdateMetaData请求到存活的Broker|NewReplica 
    NewReplica-->|如果需要的话,将新的replica添加到assigned replica list| OnlineReplica
    OnlineReplica-->|1.向replica发送StopReplicaRequest请求,2.从LSR中移除当前Replica,想Leader副本发送LeaderAndLsr请求,并为分区发送UpdateMetaData请求到存活的Broker|OfflineReplica
    OfflineReplica-->|向新的replica发送LeaderAndLsr请求,告诉它当前leader和lsr信息并为分区发送UpdateMetaData请求到存活的Broker|OnlineReplica
    OfflineReplica-->|向replica发送StopReplicaRequest|NonExistentReplica
    ```

    

