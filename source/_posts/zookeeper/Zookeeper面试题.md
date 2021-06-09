---
title: Zookeeper面试题-Kafka相关(待重新整理)
date: 2021-06-03 23:37:06
index_img: /img/cover/Zookeeper.jpg
cover: /img/cover/Zookeeper.jpg
tags:
- 中间件
- 面试题
categories:
- Zookeeper
---

#### 参考文献

*  [Kafka 常见面试题整理](https://mp.weixin.qq.com/s/pShUnc1pyjg3alIwxDFkPw)

#### Zookeeper简介

> ZooKeeper 的数据是保存在节点上的，每个节点也被称为znode，znode 节点是一种树形的文件结构，它很像 Linux 操作系统的文件路径，ZooKeeper 的根节点是 /。
>
> znode 根据数据的持久化方式可分为临时节点和持久性节点。持久性节点不会因为 ZooKeeper 状态的变化而消失，但是临时节点会随着 ZooKeeper 的重启而自动消失。
>
> znode 节点有一个 Watcher 机制：当数据发生变化的时候， ZooKeeper 会产生一个 Watcher 事件，并且会发送到客户端。Watcher 监听机制是 Zookeeper 中非常重要的特性，我们基于 Zookeeper 上创建的节点，可以对这些节点绑定监听事件，比如可以监听节点数据变更、节点删除、子节点状态变更等事件，通过这个事件机制，可以基于 ZooKeeper 实现分布式锁、集群管理等功能。

#### **Controller的选举**

> Kafka 当前选举控制器的规则是：Kafka 集群中第一个启动的 broker 通过在 ZooKeeper 里创建一个临时节点 /controller 让自己成为 controller 控制器。其他 broker 在启动时也会尝试创建这个节点，但是由于这个节点已存在，所以后面想要创建 /controller 节点时就会收到一个节点已存在 的异常。然后其他 broker 会在这个控制器上注册一个 ZooKeeper 的 watch 对象，/controller 节点发生变化时，其他 broker 就会收到节点变更通知。这种方式可以确保只有一个控制器存在。那么只有单独的节点一定是有个问题的，那就是单点问题。
>
> 如果控制器关闭或者与 ZooKeeper 断开链接，ZooKeeper 上的临时节点就会消失。集群中的其他节点收到 watch 对象发送控制器下线的消息后，其他 broker 节点都会尝试让自己去成为新的控制器。其他节点的创建规则和第一个节点的创建原则一致，都是第一个在 ZooKeeper 里成功创建控制器节点的 broker 会成为新的控制器，那么其他节点就会收到节点已存在的异常，然后在新的控制器节点上再次创建 watch 对象进行监听。

#### **Controller的作用**

> Kafka 被设计为一种模拟状态机的多线程控制器，它可以作用有下面这几点：
>
> 控制器相当于部门（集群）中的部门经理（broker controller），用于管理部门中的部门成员（broker）
>
> 控制器是所有 broker 的一个监视器，用于监控 broker 的上线和下线在
>
> broker 宕机后，控制器能够选举新的分区 Leader
>
> 控制器能够和 broker 新选取的 Leader 发送消息
>
> 具体分为如下 5 点：
>
> **主题管理** : Kafka Controller 可以帮助我们完成对 Kafka 主题创建、删除和增加分区的操作，简而言之就是对分区拥有最高行使权。换句话说，当我们执行kafka-topics 脚本时，大部分的后台工作都是控制器来完成的。
>
> **分区重分配**: 分区重分配主要是指，kafka-reassign-partitions 脚本提供的对已有主题分区进行细粒度的分配功能。这部分功能也是控制器实现的。
>
> **Prefered 领导者选举** : Preferred 领导者选举主要是 Kafka 为了避免部分 Broker 负载过重而提供的一种换 Leader 的方案。
>
> **集群成员管理**: 主要管理 新增 broker、broker 关闭、broker
>
> **宕机数据服务**: 控制器的最后一大类工作，就是向其他 broker 提供数据服务。控制器上保存了最全的集群元数据信息，其他所有 broker 会定期接收控制器发来的元数据更新请求，从而更新其内存中的缓存数据。
>
> 当控制器发现一个 broker 离开集群（通过观察相关 ZooKeeper 路径），控制器会收到消息：这个 broker 所管理的那些分区需要一个新的 Leader。控制器会依次遍历每个分区，确定谁能够作为新的 Leader，然后向所有包含新 Leader 或现有 Follower 的分区发送消息，该请求消息包含谁是新的 Leader 以及谁是 Follower 的信息。随后，新的 Leader 开始处理来自生产者和消费者的请求，Follower 用于从新的 Leader 那里进行复制。
>
> 当控制器发现一个 broker 加入集群时，它会使用 broker ID 来检查新加入的 broker 是否包含现有分区的副本。如果有控制器就会把消息发送给新加入的 broker 和 现有的 broker。

#### **broker controller 数据存储**

> 上面我们介绍到 broker controller 会提供数据服务，用于保存大量的 Kafka 集群数据。主要分为三类：
>
> broker 上的所有信息，包括 broker 中的所有分区，broker 所有分区副本，当前都有哪些运行中的 broker，哪些正在关闭中的 broker 。
>
> 所有主题信息，包括具体的分区信息，比如领导者副本是谁，ISR 集合中有哪些副本等。
>
> 所有涉及运维任务的分区。包括当前正在进行 Preferred 领导者选举以及分区重分配的分区列表。Kafka 是离不开 ZooKeeper的，所以这些数据信息在 ZooKeeper 中也保存了一份。每当控制器初始化时，它都会从 ZooKeeper 上读取对应的元数据并填充到自己的缓存中。

#### **broker controller 故障转移**

> broker controller 只有一个，那么必然会存在单点失效问题。kafka 为考虑到这种情况提供了故障转移功能，也就是 Fail Over。
>
> 最一开始，broker1 会抢先注册成功成为 controller，然后由于网络抖动或者其他原因致使 broker1 掉线，ZooKeeper 通过 Watch 机制觉察到 broker1 的掉线，之后所有存活的 brokers 开始竞争成为 controller，这时 broker3 抢先注册成功，此时 ZooKeeper 存储的 controller 信息由 broker1 -> broker3，之后，broker3 会从 ZooKeeper 中读取元数据信息，并初始化到自己的缓存中。
>
> 注意：ZooKeeper 中存储的不是缓存信息，broker 中存储的才是缓存信息。

#### **broker controller 存在的问题**

> 在 Kafka 0.11 版本之前，控制器的设计是相当繁琐的。我们上面提到过一句话：Kafka controller 被设计为一种模拟状态机的多线程控制器，这种设计其实是存在一些问题的。controller 状态的更改由不同的监听器并发执行，因此需要进行很复杂的同步，并且容易出错而且难以调试。状态传播不同步，broker 可能在时间不确定的情况下出现多种状态，这会导致不必要的额外的数据丢失。controller 控制器还会为主题删除创建额外的 I/O 线程，导致性能损耗。controller 的多线程设计还会访问共享数据，我们知道，多线程访问共享数据是线程同步最麻烦的地方，为了保护数据安全性，控制器不得不在代码中大量使用ReentrantLock 同步机制，这就进一步拖慢了整个控制器的处理速度。

#### **broker controller 内部设计原理**

> 在 Kafka 0.11 之后，Kafka controller 采用了新的设计，把多线程的方案改成了单线程加事件队列的方案。主要所做的改变有下面这几点：
>
> 第一个改进是增加了一个 Event Executor Thread，事件执行线程，不管是 Event Queue 事件队列还是 Controller context 控制器上下文都会交给事件执行线程进行处理。将原来执行的操作全部建模成一个个独立的事件，发送到专属的事件队列中，供此线程消费。
>
> 第二个改进是将之前同步的 ZooKeeper 全部改为异步操作。ZooKeeper API 提供了两种读写的方式：同步和异步。之前控制器操作 ZooKeeper 都是采用的同步方式，这次把同步方式改为异步，据测试，效率提升了10倍。
>
> 第三个改进是根据优先级处理请求，之前的设计是 broker 会公平性的处理所有 controller 发送的请求。什么意思呢？公平性难道还不好吗？在某些情况下是的，比如 broker 在排队处理 produce 请求，这时候 controller 发出了一个 StopReplica 的请求，你会怎么办？还在继续处理 produce 请求吗？这个 produce 请求还有用吗？此时最合理的处理顺序应该是，赋予 StopReplica 请求更高的优先级，使它能够得到抢占式的处理。
