---
title: Kafka架构原理
date: 2021-05-25 22:46:55
tags:
- 中间件
- Kafka
categories: 
- Java
- Kafka
---
## Kafka架构原理

* **Producer：**12264 生产者，发送消息的一方。生产者负责创建消息，然后将其发送到 Kafka。
* **Consumer：** 消费者，接受消息的一方。消费者连接到 Kafka 上并接收消息，进而进行相应的业务逻辑处理。
* **Consumer Group：** 一个消费者组可以包含一个或多个消费者。使用多分区 + 多消费者方式可以极大提高数据下游的处理速度，同一消费组中的消费者不会重复消费消息，同样的，不同消费组中的消费者消息消息时互不影响。Kafka 就是通过消费组的方式来实现消息 P2P 模式和广播模式。
* **Broker：** 服务代理节点。Broker 是 Kafka 的服务节点，即 Kafka 的服务器。
* **Topic：** Kafka 中的消息以 Topic 为单位进行划分，生产者将消息发送到特定的 Topic，而消费者负责订阅 Topic 的消息并进行消费。
* **Partition：** Topic 是一个逻辑的概念，它可以细分为多个分区，每个分区只属于单个主题。同一个主题下不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。
* **Offset：** offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说，Kafka 保证的是分区有序性而不是主题有序性。
* **Replication：** 副本，是 Kafka 保证数据高可用的方式，Kafka 同一 Partition 的数据可以在多 Broker 上存在多个副本，通常只有主副本对外提供读写服务，当主副本所在 broker 崩溃或发生网络异常，Kafka 会在 Controller 的管理下会重新选择新的 Leader 副本对外提供读写服务。
* **Record：** 实际写入 Kafka 中并可以被读取的消息记录。每个 record 包含了 key、value 和 timestamp。

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

    

