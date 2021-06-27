---
title: Redis-哨兵模式(Sentinel)
date: 2021-06-04 22:54:54
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags:
- 哨兵模式
- Sentinel
categories:
- Redis
---

#### 为什么要引入哨兵模式?

> Redis 的主从复制模式下，一旦主节点由于故障不能提供服务，需要手动将从节点晋升为主节点，同时还要通知客户端更新主节点地址，这种故障处理方式从一定程度上是无法接受的。
>
> Redis 2.8 以后提供了 Redis Sentinel 哨兵机制来解决这个问题。
>
> Redis Sentinel 是 Redis 高可用的实现方案。Sentinel 是一个管理多个 Redis 实例的工具，它可以实现对 Redis 的监控、通知、自动故障转移。

![img](http://www.chenjunlin.vip/img/redis/Redis_Sentinel%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

#### 哨兵模式原理

> 哨兵模式的主要作用在于它能够自动完成故障发现和故障转移，并通知客户端，从而实现高可用。哨兵模式通常由一组 Sentinel 节点和一组（或多组）主从复制节点组成。

#### 哨兵如何实现相互监督的功能

* 第一: 哨兵通过发布订阅\_sentinel_:hello channel来实现这个功能.每个哨兵每隔2s会想自己监控的所有主从Redis节点发送hello message,包括自己的IP,端口,运行ID,自己监控的Master节点IP,Master节点端口;
* 第二: 所有主从Redis节点也会反馈这样的信息;

#### 哨兵如何检测故障

* 第一: 某个哨兵节点判定Master节点故障,它会投出一票S_DOWN
* 第二: 当有足够多的sentinel节点判定Master节点故障都投出S_DOWN票时,Master节点会被认为真正的下线了
* 也就是**基于多数投票原则**

#### **心跳机制**

* Sentinel与Redis Node

  > Redis Sentinel 是一个特殊的 Redis 节点。在哨兵模式创建时，需要通过配置指定 Sentinel 与 Redis Master Node 之间的关系，然后 Sentinel 会从主节点上获取所有从节点的信息，之后 Sentinel 会定时向主节点和从节点发送 info 命令获取其拓扑结构和状态信息。

*  Sentinel与Sentinel

  > 基于 Redis 的订阅发布功能， 每个 Sentinel 节点会向主节点的 **sentinel**：hello 频道上发送该 Sentinel 节点对于主节点的判断以及当前 Sentinel 节点的信息 ，同时每个 Sentinel 节点也会订阅该频道， 来获取其他 Sentinel 节点的信息以及它们对主节点的判断。
  >
  > 通过以上两步所有的 Sentinel 节点以及它们与所有的 Redis 节点之间都已经彼此感知到，之后每个 Sentinel 节点会向主节点、从节点、以及其余 Sentinel 节点定时发送 ping 命令作为心跳检测， 来确认这些节点是否可达。

#### **故障转移**

每个 Sentinel 都会定时进行心跳检查，当发现主节点出现心跳检测超时的情况时，此时认为该主节点已经不可用，这种判定称为**主观下线**。

之后该 Sentinel 节点会通过 sentinel ismaster-down-by-addr 命令向其他 Sentinel 节点询问对主节点的判断， 当 quorum（法定人数） 个 Sentinel 节点都认为该节点故障时，则执行**客观下线**，即认为该节点已经不可用。这也同时解释了为什么必须需要一组 Sentinel 节点，因为单个 Sentinel 节点很容易对故障状态做出误判。

> 这里 quorum 的值是我们在哨兵模式搭建时指定的，后文会有说明，通常为 Sentinel节点总数/2+1，即半数以上节点做出主观下线判断就可以执行客观下线。

因为故障转移的工作只需要一个 Sentinel 节点来完成，所以 Sentinel 节点之间会再做一次选举工作， 基于 Raft 算法选出一个 Sentinel 领导者来进行故障转移的工作。

**被选举出的 Sentinel 领导者进行故障转移的具体步骤如下**：

* 在从节点列表中选出一个节点作为新的主节点
  * 过滤不健康或者不满足要求的节点；
  * 选择 slave-priority（优先级）最高的从节点， 如果存在则返回， 不存在则继续；
  * 选择复制偏移量最大的从节点 ， 如果存在则返回， 不存在则继续；
  * 选择` runid` 最小的从节点。

* Sentinel 领导者节点会对选出来的从节点执行 `slaveof no one `命令让其成为主节点。

* Sentinel 领导者节点会向剩余的从节点发送命令，让他们从新的主节点上复制数据。

* Sentinel 领导者会将原来的主节点更新为从节点， 并对其进行监控， 当其恢复后命令它去复制新的主节点。

#### 选主机制

* **过程**

  > sentinel的选举过程基本上是Raft协议的实现，即所有节点会随机休眠一段时间，然后发起拉票，当某个节点获得的票数超过max(sentinel|/2 + 1), qurom时，该节点就被推选为leader节点。注意是哨兵去从节点里面选。

* **选谁**

  > **①根据指定的优先级选择**
  >
  > 管理员在启动redis从节点的时候，指定了其优先级，哨兵会先从优先级高的从节点去选择。
  >
  > ②**根据数据更新程度选择**
  >
  > 优先级相同，所有slave节点复制数据的时候都会记录复制偏移量，值越大说明与master节点的数据更一致。所以哨兵会选择复制偏移量最大的节点。
  >
  > ③**根据runid选择：**
  >
  > 到了这一步节点的孰优孰劣就没什么区别了，每个节点启动的时候都会有一个唯一的runId, 那么我们就选择runid最小的节点好了。
