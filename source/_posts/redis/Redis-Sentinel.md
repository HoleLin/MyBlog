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
updated: 2021-07-13 22:41:24
---

### 参考文献



#### 为什么要引入哨兵模式?

* Redis 的主从复制模式下，一旦主节点由于故障不能提供服务，需要手动将从节点晋升为主节点，同时还要通知客户端更新主节点地址，这种故障处理方式从一定程度上是无法接受的。

* Redis 2.8 以后提供了 Redis Sentinel 哨兵机制来解决这个问题。

* Redis Sentinel 是 Redis 高可用的实现方案。Sentinel 是一个管理多个 Redis 实例的工具，它可以实现对 Redis 的监控、通知、自动故障转移。

#### 基本概念

| 名词             | 逻辑结构                   | 物理结构                            |
| ---------------- | -------------------------- | ----------------------------------- |
| 主节点(master)   | Redis主服务/数据库         | 一个独立的Redis进程                 |
| 从节点(slave)    | Redis从服务/数据库         | 一个独立的Redis进程                 |
| Redis数据节点    | 主节点和从节点             | 主节点和从节点的进程                |
| Sentinel节点     | 监控Redis数据节点          | 一个独立的Sentinel进程              |
| Sentinel节点集合 | 若干Sentinel节点的抽象组合 | 若干Sentinel节点进程                |
| Redis Sentinel   | Redis高可用实现方案        | Sentinel节点集合和Redis数据节点进程 |
| 应用方           | 泛指一个或多个客户端       | 一个或者多个客户端进程或线程        |

#### 主从复制的问题

* Redis主从复制模式可以将主节点的数据改变同步给从节点,这样从节点就有两个作用:
  * 作为主节点的一个备份,一旦主节点出了故障不可达的情况,从节点可以作为后备顶上来,并且保证数据尽量不丢失(主从复制是最终一致性);
  * 从节点可以扩展主节点的读能力,一旦主节点不能支撑住大并发量的读操作,从节点可以在一定程度上帮助主节点分担读压力;
* 但主从复制也带来了以下问题:
  * 一旦主节点出现故障,需要**手动将一个从节点晋升为主节点**,同时**需要修改应用方的主节点地址**,还**需要命令其他从节点复制新的主节点**,整个过程都需要人工干预;(**Redis高可用问题**)
  * 主节点的写能力受到单机的限制;**(Redis分布式问题)**
  * 主节点的存储能力受到单机的限制;**(Redis分布式问题)**

##### 高可用问题

> 示例: 一主二从的Redis主从复制模式下出现故障问题后,是如何进行故障转移的.

* 主节点发送故障后,客户端(Client)连接主节点失败,两个从节点与主节点连接失败造成复制终端;

  <img src="http://www.chenjunlin.vip/img/redis/sentinel/%E4%B8%80%E4%B8%BB%E4%BA%8C%E4%BB%8E%E5%87%BA%E7%8E%B0%E6%95%85%E9%9A%9C1.png" alt="img" style="zoom: 67%;" />

* 如果主节点无法正常启动,需要选出一个从节点(slave-1),对其执行`slaveof no one`命令使其成为新的主节点;

* 原来的从节点(slave-1)成为新的主节点后,更新应用方的主节点信息,重新启动应用方;

* 客户端命令另一个从节点(slave-2)去复制新的主节点(new master);

*  待原来的主节点恢复后,让它去复制新的主节点;

<img src="http://www.chenjunlin.vip/img/redis/sentinel/%E4%B8%80%E4%B8%BB%E4%BA%8C%E4%BB%8E%E5%87%BA%E7%8E%B0%E6%95%85%E9%9A%9C2.png" alt="img" style="zoom: 67%;" />

* 上述处理过程存在的问题
  * 判断节点不可达的机制是否健全和标准?
  * 如果有多个从节点,怎么保证只有一个晋升为主节点?
  * 通知客户端新的主节点机制是否足够健壮?
* Redis Sentinel正是用于解决这些问题

#### Redis Sentinel的高可用性

> 当主节点出现故障时,Redis Sentinel能自动完成故障发现和故障转移,并通知应用方,从而实现真正的高可用;

<img src="http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E6%9E%B6%E6%9E%84.png" alt="img" style="zoom:67%;" />

* Redis Sentinel处理故障过程
  * Redis Sentinel是一个分布式架构,其中包含若干个Sentinel节点和Redis数据节点,每个Sentinel节点会对数据节点和其余Sentinel节点进行监控,当他发现节点不可达时,会对节点做下线标识.如果被标识的是主节点,它还会和其他Sentinel节点进行"协商",当大多数Sentinel节点都认为不可达时,它们会选举出一个Sentinel节点来完成自动故障转移工作,同时将这个变化通知给Redis应用方.
  * 整个过程完全是自动的,不需要人工来介入;

##### Redis Sentinel处理高可用问题

> 示例: 一主二从的Redis Sentinel模式下出现故障问题后,是如何进行故障转移的

* 主节点出现故障,此时两个从节点与主节点失去连接,主从复制失败;
* 每个Sentinel节点通过定期监控发现主节点出现故障;
* 多个Sentinel节点对主节点的故障达成一致,选举出`sentinel-3`节点作为领导者负责故障转移;

<img src="http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentine%E5%A4%84%E7%90%86%E9%AB%98%E5%8F%AF%E7%94%A8%E9%97%AE%E9%A2%98.png" alt="img" style="zoom:67%;" />

* Sentinel领导者节点执行故障转移,整个过程和主从复制处理故障一致,只不过是自动完成的;

  <img src="http://www.chenjunlin.vip/img/redis/sentinel/Sentinel%E9%A2%86%E5%AF%BC%E8%80%85%E8%8A%82%E7%82%B9%E6%89%A7%E8%A1%8C%E6%95%85%E9%9A%9C%E8%BD%AC%E7%A7%BB%E7%9A%84%E5%9B%9B%E4%B8%AA%E6%AD%A5%E9%AA%A4.png" alt="img" style="zoom:67%;" />

* 故障转移后整个Redis Sentinel的拓扑结构图

  <img src="http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E6%95%85%E9%9A%9C%E8%BD%AC%E7%A7%BB%E5%90%8E%E7%9A%84%E6%8B%93%E6%89%91%E5%9B%BE.png" alt="img" style="zoom:67%;" />

#### Redis Sentinel具有的功能

* **监控**: Sentinel节点会定期监测Redis数据节点,其余Sentinel节点是否可达;
* **通知**: Sentinel节点会故障转移的结果通知给应用方;
* **主节点故障转移**: 实现从节点晋升为主节点并维护后续正确的主从关系;
* **配置提供者**: 在Redis Sentinel结构中,客户端在初始化的时候连接的是Sentinel节点集合,从中获取主节点信息;

* Redis Sentinel包含了若干Sentinel节点,这样做也带来了两个好处:
  * 对于节点的故障判断是由多个Sentinel节点共同完成,这样可以有效地防止误判;
  * Sentinel节点集合是由若干个Sentinel节点组成的,这样即使个别Sentinel节点不可用,整个Sentinel节点集合依然健壮的;
  * 但是Sentinel节点本身就是独立的Redis节点,只不过它们有一些特殊,它们不存储数据,支持部分命令;

#### Redis Sentinel配置

> 部署三个Sentinel节点,一个主节点,两个从节点组成一个Redis Sentinel

* 拓扑图

  <img src="http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E9%83%A8%E7%BD%B2%E6%8B%93%E6%89%91%E5%9B%BE.png" alt="img" style="zoom:67%;" />

* 具体物理结构

  | 角色       | IP        | Port  | 别名                    |
  | ---------- | --------- | ----- | ----------------------- |
  | master     | 127.0.0.1 | 6379  | 主节点或者6379节点      |
  | slave-1    | 127.0.0.1 | 6380  | slave-1或者6380节点     |
  | slave-2    | 127.0.0.1 | 6381  | slave-2或者6381节点     |
  | sentinel-1 | 127.0.0.1 | 26379 | sentinel-1或者26379节点 |
  | sentinel-2 | 127.0.0.1 | 26380 | sentinel-2或者26380节点 |
  | sentinel-3 | 127.0.0.1 | 26381 | sentinel-3或者26381节点 |

* 启动主节点

  * 配置

    ```
    port 6379
    daemonize yes
    pidfile /var/run/redis_6379.pid
    logfile "/usr/local/redis-6.2.2/logs/6379.log"
    dbfilename dump_6379.rdb
    dir /usr/local/redis-6.2.2/data/
    ```

* 启动两个从节点

  * 配置

    ```properties
    port 6380
    daemonize yes
    pidfile /var/run/redis_6380.pid
    logfile "/usr/local/redis-6.2.2/logs/6380.log"
    dbfilename dump_6380.rdb
    dir /usr/local/redis-6.2.2/data/
    slaveof 127.0.0.1 6379
    ```

    ```properties
    port 6381
    daemonize yes
    pidfile /var/run/redis_6381.pid
    logfile "/usr/local/redis-6.2.2/logs/6381.log"
    dbfilename dump_6381.rdb
    dir /usr/local/redis-6.2.2/data/
    slaveof 127.0.0.1 6379
    ```

* 确认主从关系

  * 主节点客户端操作`redis-cli -p 6379 -a holelin info replication`

    ```properties
    # Replication
    role:master
    connected_slaves:2
    slave0:ip=127.0.0.1,port=6380,state=online,offset=98,lag=0
    slave1:ip=127.0.0.1,port=6381,state=online,offset=98,lag=1
    master_failover_state:no-failover
    master_replid:72078d9d8d84bc7d89d6f25e7ee1de5366add480
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:98
    second_repl_offset:-1
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:1
    repl_backlog_histlen:98
    ```

  * 从节点操作:

    * `redis-cli -p 6380 -a holelin info replication`

      ```properties
      # Replication
      role:slave
      master_host:127.0.0.1
      master_port:6379
      master_link_status:up
      master_last_io_seconds_ago:8
      master_sync_in_progress:0
      slave_repl_offset:434
      slave_priority:100
      slave_read_only:1
      replica_announced:1
      connected_slaves:0
      master_failover_state:no-failover
      master_replid:72078d9d8d84bc7d89d6f25e7ee1de5366add480
      master_replid2:0000000000000000000000000000000000000000
      master_repl_offset:434
      second_repl_offset:-1
      repl_backlog_active:1
      repl_backlog_size:1048576
      repl_backlog_first_byte_offset:1
      repl_backlog_histlen:434
      ```

    * `redis-cli -p 6381 -a holelin info replication`

      ```properties
      # Replication
      role:slave
      master_host:127.0.0.1
      master_port:6379
      master_link_status:up
      master_last_io_seconds_ago:0
      master_sync_in_progress:0
      slave_repl_offset:504
      slave_priority:100
      slave_read_only:1
      replica_announced:1
      connected_slaves:0
      master_failover_state:no-failover
      master_replid:72078d9d8d84bc7d89d6f25e7ee1de5366add480
      master_replid2:0000000000000000000000000000000000000000
      master_repl_offset:504
      second_repl_offset:-1
      repl_backlog_active:1
      repl_backlog_size:1048576
      repl_backlog_first_byte_offset:15
      repl_backlog_histlen:490
      ```

* 部署Sentinel节点

  * 配置

    ```properties
    port 26379
    daemonize yes
    pidfile /var/run/redis-sentinel-26379.pid
    logfile "/usr/local/redis-6.2.2/logs/sentinel-26379.log"
    dir /usr/local/redis-6.2.2/data/
    # 表示sentinel-1节点需要监控127.0.0.1:6379这个节点 2代表判断主节点失败至少需要2个Sentinel节点同意
    # holelin是主节点的别名
    sentinel monitor holelin 127.0.0.1 6379 2
    # 主节点的认证密码
    sentinel auth-pass holelin holelin
    ```

  * 启动Sentinel节点

    * `redis-sentinel 配置文件`
    * `redis-server 配置文件 --sentinel`

  * 确认

    ```sh
    [root@holelin redis-6.2.2]# redis-cli -p 26379 info sentinel
    # Sentinel
    sentinel_masters:1
    sentinel_tilt:0
    sentinel_running_scripts:0
    sentinel_scripts_queue_length:0
    sentinel_simulate_failure_flags:0
    master0:name=holelin,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
    ```

#### 配置优化说明

```properties
port 26379
daemonize yes
pidfile /var/run/redis-sentinel-26379.pid
logfile "/usr/local/redis-6.2.2/logs/sentinel-26379.log"
dir /usr/local/redis-6.2.2/data/
# 表示sentinel-1节点需要监控127.0.0.1:6379这个节点 2代表判断主节点失败至少需要2个Sentinel节点同意
# holelin是主节点的别名
sentinel monitor holelin 127.0.0.1 6379 2
# 主节点的认证密码
sentinel auth-pass holelin holelin
sentinel down-after-milliseconds holelin 30000
sentinel parallel-syncs holelin 1
sentinel fail-over-timeout holelin 180000
# sentinel auth-pass <master-name> <password>
# sentinel notification-script <master-name> <scripte-path>
# sentinel client-reconfig-script <master-name> <script-path>
```

* `sentinel moitor <master-name> <ip> <port> <quorum>`

  * `<quorum>`代表要判定主节点最终不可达所需要的票数,但实际上Sentinel节点会对所有节点进行监控,但是Sentinel节点的配置中没有看到有关从节点和其余Sentinel节点的配置,那是因为Sentinel节点会从主节点中获取有关从节点以及其余Sentinel节点的相关信息.
  * `<quorum>`参数用于故障发现和判定,例如将`quorum`配置为2,则代表至少有2个Sentinel节点认为主节点不可达,那么这个不可达的判定才是客观的;
  * 一般建议设置为Sentinel节点数的一半加1;
  * 同时`quorum`还与Sentinel节点领导者选举有关,至少有`max(quorum,num(sentinels)/2+1)`个Sentinel节点参与选举,才能选出领导者Sentinel,从而完成故障转移.

* 当所有从节点启动后,配置文件中的内容会发生变化,体现在三个方面

  * Sentinel节点自动发现从节点,其余Sentinel节点

  * 去掉默认配置,如`parallel-syncs`,`fail-over-timeout`

  * 添加了配置纪元相关参数

    ```properties
    user default on nopass ~* +@all
    sentinel leader-epoch holelin 0
    sentinel known-replica holelin 127.0.0.1 6381
    sentinel known-replica holelin 127.0.0.1 6380
    sentinel known-sentinel holelin 127.0.0.1 26380 8a47d95157673f1a0d4ac67bbc323208daf80d49
    sentinel known-sentinel holelin 127.0.0.1 26381 13c895cd85d5614055f1a633e7939f29ef5f6f11
    sentinel current-epoch 0
    ```

* `sentinel down-after-milliseconds <master-name> <times>`

  * 每个Sentinel节点都要通过定期发送`ping`命令来判断Redis数据节点和其他Sentinel节点是否可达,如果超过了`down-after-milliseconds`配置的时间且没有有效的回复,则判定节点不可达,`<times>`单位为毫秒,就是超时时间

* `sentinel parallel-syncs <master-name> <nums>`

  * 当Sentinel节点集合对主节点故障判定达成一致时,Sentinel领导者节点会做故障转移操作,选出新的主节点,原来的从节点会向新的主节点发起复制操作;**`parallel-syncs`就是用来限制在一次故障转移之后,每次向新的主节点发起复制操作的从节点个数.**

* `sentinel fail-over-timeout <master-name> <times>`

  * `fail-over-timeout`: 故障转移超时时间,但实际上它的作用于故障转移的各个阶段
    * a) 选出合适从节点
    * b) 晋升选出的从节点为主节点
    * c) 命令其余从节点复制新的主节点
    * d) 等待原来节点恢复后命令它去复制新的主节点
  * `fail-over-timeout`的作用具体体现在四个方面
    * 若Redis Sentinel对一个主节点故障转移失败,那么下次再对该主节点做故障转移的起始时间是`fail-over-timeout`的2倍;
    * 在b)阶段,若Sentinel节点向a)阶段选出来的从节点执行`slaveof no one`一直失败,当此过程超过`fail-over-timeout`时,则表示故障转移失败;
    * 在b)阶段如果执行成功,Sentinel节点还会执行`info`命令来确认a)阶段选出的节点确定晋升为主节点,如果此过程执行时间超过`fail-over-timeout`时,则表示故障转移失败;
    * 若c)阶段执行时间超过`fail-over-timeout`(不包含复制时间),则表示故障转移失败.注意即使超过了这个时间,Sentinel节点也会最终配置从节点同步最新的主节点;

* `sentinel auth-pass <master-name> <password>`

  * 若Sentinel监控的主节点配置了密码,`sentinel auth-pass`配置通过添加主节点的密码,防止Sentinel节点对主节点无法监控;

* `sentinel notification-script <master-name> <script-path>`

  * `sentinel notification-script`作用是在故障转移期间,当一些告警级别的Sentinel事件发生(指重要事件,例如`-sdown`: 客观下线,`-odown`: 主观下线)时,会触发对应路径的脚本,并想脚本发送响应的事件参数;

    ```sh
    #!/bin/sh
    # 获取所有参数
    msg=$*
    # 报警脚本或接口,将msg作为参数
    exit 0
    ```

* `sentinel client-reconfig-script <master-name> <script-path>`

  * `sentinel client-reconfig-script`作用是在故障转移结束后,会触发对应路径的脚本,并向脚本发送故障转移结果的参数.

  * 当故障转移结束,每个Sentinel节点会将故障转移的结果发送的脚本,具体参数如下:

    ```
    <master-name> <role> <state> <from-ip> <from-port> <to-ip> <from-port>
    
    <master-name>: 主节点名
    <role>: Sentinel节点的角色,分别是leader和observer,leader代表当前Sentinel节点是领导者,是它进行的故障转移;observer是其余Sentinel节点
    <state>
    <from-ip>: 原主节点的ip地址
    <from-port>: 原主节点的端口
    <to-ip>: 新主节点的ip地址
    <from-port>: 新主节点的端口
    ```

  * 其中`sentinel notification-script`和`sentinel client-reconfig-script`有几点需要注意

    * `<script-path>`必须有可执行权限

    * `<script-path>`开头必须包含`shell`脚本(例如`#!/bin/sh`),否则事件发生时Redis将无法执行脚本产生如下错误:

      ```
      -script-error /usr/local/redis-6.2.2/script notification.sh
      ```

    * Redis规定脚本的最大执行时间不能超过60秒,超过后脚本将被杀掉

    * 如果`shell`脚本以`exit 1`结束,那么脚本稍后重试执行,如果以`exit 2`或者更高结束,那么脚本重试.正常返回值是`exit 0`

#### 调整配置

* Sentinel节点也支持动态地设置参数,而且和普通的Redis数据节点一样并不是支持所有的参数

  ```
  sentinel set <param> <value>
  ```

  | 参数                       | 使用说明                                                   |
  | -------------------------- | ---------------------------------------------------------- |
  | `quorm`                    | `sentinel set mymaster quorum 2`                           |
  | `down-after-millisenconds` | `sentinel set mymaster down-after-millisenconds 30000`     |
  | `failover-timeout`         | `sentinel set mymaster fail-over-timeout 360000`           |
  | `parallel-syncs`           | `sentinel set mymaster parallel-syncs 2`                   |
  | `notification-script`      | `sentinel set mymaster notification-script /opt/xx.sh`     |
  | `client-recoding-script`   | `sentinel set mymaster client-recoding-script  /opt/yy.sh` |
  | `auth-pass`                | `sentinel set mymaster auth-pass masterPassword`           |

  * `sentinel set`命令只对当前Sentinel节点有效
  * `sentinel set` 如果执行成功会立即刷新配置文件,这点和Redis普通数据节点设置配置需要执行`config rewrite`刷新配置文件不同;
  * 建议所有Sentinel节点的配置尽可能一致,这样在故障发现和转移是比较容易达成一致;
  * Sentinel对外不支持`config`命令

#### API

* `sentinel masters`: 展示所有被监控的主节点状态以及相关的统计信息;

* `sentinel mater <master-name>`: 展示指定`<master-name>`的主节点状态以及相关的统计信息;

* `sentinel slaves <master-name>`: 展示指定`<maste-name>`的从节点状态以及相关统计信息

* `sentinel sentinels <master-name>`: 展示指定`<master-name>`的Sentinel节点集合(**不包含当前Sentinel节点**)

* `sentinel get-master-addr-by-name <master-name>`: 返回指定`<master-name>`主节点的IP地址和端口

* `sentinel reset <pattern>`: 当前Sentinel节点对符合`<pattern>`(通配符风格)主节点的配置进行重置,包含清除主节点的相关状态(例如故障转移),重新发现从节点和Sentinel节点

* `sentinel failover <master-name>`
  * 对指定`<master-name>`主节点进行强制故障转移(没有和其他Sentinel节点协商),当故障转移完成后,其他Sentinel节点按照故障转移的结果更新自身配置.
  
* `sentinel chquorum <master-name>`
  * 检测当前可达的Sentinel节点总数是否达到`<quorum>`的个数.例如`quorum`=3,而当前可达的Sentinel节点的个数为2个,那么将无法记性故障转移,Redis Sentinel的高可用特性也将失去;
  
* `sentinel flushconfig`
  * 将Sentinel节点的配置强制刷到磁盘上,这个命令Sentinel节点自身用得比较多.
  
* `sentinel remove <master-name>`: 取消当前Sentinel节点对于制定`<master-name>`主节点的监控;

* `sentinel monitor <master-name> <ip> <port> <quorum>`

* `sentinel set <param> <value>`

* `sentinel is-master-down-by-addr <ip> <port> <current_epoch> <runid>` 

  ```
  ip: 主节点IP
  port: 主节点端口
  current_epoch: 当前配置纪元
  runid: 此参数有两种类型,不同类型决定了此API的作用
  * 当runid等于"*"时,作用是Sentinel节点直接交换主节点下线的判定
  * 当runid等于当前Sentinel节点的runid时,作用是当前Sentinel节点希望目标同意自己成为领导者的请求;
  
  返回结果包含三个参数:
  * down_state: 目标Sentinel节点对于主节点的下线判断,1是下线,0是在线;
  * leader_runid: 当leader_runid等于"*",代表返回结果是用来做主节点是否不可达;当leader_runid等于具体的runid,代表目标节点同意runid成为领导者
  * leader_epoch: 领导者纪元
  ```
  
* Sentinel节点之间用来交换对主节点是否下线的判断,根据参数的不同,还可以作为Sentinel领导者选举的通信方式;

#### Redis  Sentinel客户端基本实现原理

* 实现一个Redis Sentinel客户端的基本步骤

  * 遍历Sentinel节点集合获取一个可用的Sentinel节点,Sentinel节点之间可以共享数据,所有从任意一个Sentinel节点获取主节点的信息都是可以的;

    ![img](http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E5%AE%A2%E6%88%B7%E7%AB%AF1.png)

  * 通过`sentinel get-master-addr-by-name master-name`这个API来获取对应主节点的相关信息;

    ![img](http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E5%AE%A2%E6%88%B7%E7%AB%AF2.png)

  * 验证当前获取的"主节点"是真正的主节点,这样做的目的是为了防止故障转移期间主节点的变化;

    ![img](http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E5%AE%A2%E6%88%B7%E7%AB%AF3.png)

  * 保持Sentinel节点集合的"联系",时刻获取关于主节点的相关'"信息"

    ![img](http://www.chenjunlin.vip/img/redis/sentinel/Redis%20Sentinel%E5%AE%A2%E6%88%B7%E7%AB%AF4.png)

#### 哨兵模式原理

> 一套合理的监控机制是Sentinel节点判断节点不可达的重要保证,Redis Sentinel通过**三个定时监控任务**完成对各个节点发现和监控

* **每隔10秒,每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构;**
  * 这个定时任务的作用可以表现在三个方面:
    * 通过向主节点执行info命令,获取从节点的信息,这也是为什么Sentinel节点不需要显示配置监控从节点;
    * 当有新的从节点加入时都可以立刻感知出来;
    * 节点不可达或者故障转移后,可以通过info命令实时更新节点拓扑信息
  
  <img src="http://www.chenjunlin.vip/img/redis/sentinel/Sentinel%E8%8A%82%E7%82%B9%E5%AE%9A%E6%97%B6%E6%89%A7%E8%A1%8Cinfo%E5%91%BD%E4%BB%A4.png" alt="img" style="zoom:67%;" />
  
* **每隔2秒,每个Sentinel节点会向Redis master节点的`_sentinel_:hello`频道发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息,同时每个Sentinel也会订阅该频道,来了解其他Sentinel节点以及它们对主节点的判断,**所以这个定时任务完成一下两个工作:
  
  * **发现新的Sentinel节点**:通过订阅主节点的`_sentinel_:hello`了解其他的Sentinel节点信息,如果是新加入的Sentinel节点,将该Sentinel节点信息保存起来,并与该Sentinel节点创建连接;
  
  * **Sentinel节点之间交换主节点的状态,作为客观下线以及领导者选举的依据**;
  
  * Sentinel节点的publish的消息格式如下:
  
    ```
    <Sentinel节点Ip><Sentinel节点端口><Sentinel节点runId><Sentinel节点配置版本><主节点名字><主节点Ip><主节点端口><主节点配置版本>
    ```
  
  <img src="http://www.chenjunlin.vip/img/redis/sentinel/Sentinel%E8%8A%82%E7%82%B9%E5%8F%91%E5%B8%83%E5%92%8C%E8%AE%A2%E9%98%85_sentinel__hello%E9%A2%91%E9%81%93.png" alt="img" style="zoom:67%;" />
  
* **每隔1秒,每个Sentinel节点会向主节点,从节点,其余Sentinel节点发送一条ping命令做一次心跳检测,来确认这些节点当前是否可达.**

##### 主观下线

* 三个定时任务,每隔Sentinel节点会每隔1秒对主节点,从节点,其他Sentinel节点发送ping命令做心跳检测,当这些节点`ping`命令的时间超过`down-after-milliseconds`没有进行有效回复,Sentinel节点就会对该节点做失败判定,这个行为叫做**主观下线**;

  <img src="http://www.chenjunlin.vip/img/redis/sentinel/%E4%B8%BB%E8%A7%82%E4%B8%8B%E7%BA%BF.png" alt="img" style="zoom:67%;" />

* 从节点、Sentinel在主观下线后,没有后续的故障转移操作;

##### 客观下线

* 当Sentinel主观下线的节点是主节点,该Sentinel节点会通过`sentinel is-master-down-by-addr`命令向其他Sentinel节点询问对主节点的判断,当超过`<quorum>`个数,Sentinel节点认为主节点确实有问题,这是该Sentinel节点会做出客观下线的决定
* 客观下线的含义比较明显,也就是大部分Sentinel节点都对主节点的下线做了同意的判定.

#### 领导者Sentinel节点选举

* Redis采用`Raft`算法实现领导者选举,大致思路:
  * 在每个在线的Sentinel节点都有资格成为领导者,当它确认了主节点主观下线的时候,会向其他Sentinel节点发送`sentinel is-master-down-by-addr`命令要求将自己设置为领导者
  * 收到命令的Sentinel节点,如果没有同意过其他Sentinel节点的`sentinel is-master-down-by-addr`命令,则会同意当前请求,否则拒绝;
  * 若该Sentinel节点发现自己的票数已经大于等于`max(quorum,num(sentinels)/2+1)`,那么它将成为领导者;
  * 如果此过程没有选举领导者,将进入下一次选举;

#### 故障转移

* 被选举为的Sentinel领导者节点复制故障转移,具体步骤如下:

  * **在从节点列表中选出一个节点作为新的主节点**,选择方法如下:
    * 过滤"不健康"(主观下线,断线),5秒内没有回复过Sentinel节点ping响应,与主节点失联超过`down-after-miliseconds*10`秒的数据节点;
    * 选择`slave-priority`(从节点优先级)最高的从节点列表,如果存在则返回,不存在则继续;
    * 选择**复制偏移量最大**的从节点(复制的最完整的数据节点),如果存在则返回,不存在则继续
    * 选择`runid`最小的从节点;
    
  * Sentinel领导者节点会让第一步选出的从节点执行`slave no one`命令让其成为主节点;
  * Sentinel领导者节点会向剩余的从节点发送命令,让它们成为新主节点的从节点,复制规则和`parallel-syncs`参数有关;
  * Sentinel节点集合会将原来的主节点更新为从节点,并保持对其关注,当其回复后命令它去复制新的主节点;

#### Sentinel节点发布订阅信息

| 状态                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| +reset-master \<instance details>                            | 主节点被重置                                                 |
| +slave \<instance details>                                   | 一个新的从节点被发现并关联                                   |
| +failover-state-reconf-slaves \<instance details>            | 故障转移进入`reconf-slaves`状态                              |
| +slave-reconf-sent \<instance details>                       | 领导者Sentinel节点命令其他从节点复制新的主节点               |
| +slave-reconf-inprog \<instance details>                     | 从节点正在重新配置主节点的slave,但同步过程上未完成           |
| +slave-reconf-done \<instance details>                       | 其余从节点完成了和新主节点的同步                             |
| +sentinel \<instance details>                                | 一个新的sentinel节点被发现并关联                             |
| +sdown \<instance details>                                   | 添加对某个节点被主观下线                                     |
| -sdown \<instance details>                                   | 撤销对某个节点被主观下线                                     |
| +odown \<instance details>                                   | 添加对某个节点被客观下线                                     |
| -odown \<instance details>                                   | 撤销对某个节点被客观下线                                     |
| +new-epoch \<instance details>                               | 当前纪元被更新                                               |
| +try-failover \<instance details>                            | 故障转移开始                                                 |
| +elected-leader \<instance details>                          | 选出了故障转移的Sentinel节点                                 |
| +failover-state-select-slave \<instance details>             | 故障转移进入`select-slave`状态(寻找合适的从节点)             |
| no-good-slave \<instance details>                            | 没有找到合适的从节点                                         |
| selected-slave \<instance details>                           | 找到了合适的从节点                                           |
| failover-state-send-slaveof-noone \<instance details>        | 故障转移进入`failover-state-send-slaveof-noone`(对找到的从节点执行`slave no one`) |
| failover-end-for-timeout \<instance details>                 | 故障转移由于超时而终止                                       |
| failover-end \<instance details>                             | 故障转移顺利完成                                             |
| switch-master \<master-name>\<oldip>\<oldport>\<newip>\<newport> | 更新主节点信息,这个许多客户端重点关注的                      |

* \<instance details>格式如下

  ```
  <instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>
  # 示例
  +slave-reconf-done slave 1273.0.0.1:6381 @ holelin 127.0.0.1 6380
  ```

#### 运维实战

##### 节点下线区分

* 临时下线: 暂时将节点关掉,之后还会重新启动,继续提供服务;
* 永久下线: 将节点关掉后不再使用,需要做一些清理工作,如删除配置文件,持久化文件,日志文件;

##### 节点下线原因

* 节点所在的机器出现了不稳定或者即将过保被回收;
* 节点所在的机器性能比较差或者内存比较小,无法支撑应用方的需求;
* 节点自身出现不正常情况,需要快速处理;

##### 节点下线

* 对主节点进行手动下线
  * 比较合理的做法是选出"合适"(例如性能更高的机器)的从节点,使用`sentinel failover <master-name>` 功能将从节点晋升为主节点,只需要在任意可用的Sentinel节点中执行该命令;
  * Redis Sentinel存在多个从节点时,若想要指定从节点晋升为主节点,可以将其他从节点的`slavepriority`配置为0,但是需要在`failover`后,将`slavepriority`调回原值
* 对从节点和Sentinel节点进行手动下线
  * 需要确定好是临时下线和永久下线后执行响应操作即可,如果使用读写分离,下线从节点需要保证应用方可以感知从节点的下线变化,从而把读取请求路由到其他节点;
  * 需要注意的是,Sentinel节点依然会这些下线的节点进行定期监控,这是由Redis Sentinel的设计思路所决定的.可能会造成一定的网络资源浪费;

##### 节点上线

* 添加从节点
  * 原因
    * 使用了读写分离,但先有的从节点无法支撑应用方的流量;
    * 主节点没有可用的从节点,无法支持故障转移;
    * 添加一个性能更好的从节点,利用手动`failover`替换主节点;
  * 添加方法: 使用`slaveof <master-name> <master-port>`,节点启动后将被Sentinel节点自动发现;
* 添加Sentinel节点
  * 原因
    * 当前Sentinel节点数量不够,无法达到Redis Sentinel健壮性要求或者无法达到票数
    * 原Sentinel节点所在机器需要下线
  * 添加方法: 添加Sentinel节点,配置文件添加配置信息`sentinel monitor`主节点的配置,节点启动后将会被其他的Sentinel节点自动发现;
* 添加主节点
  * 因为Redis Sentinel中只能有一个主节点,所以不需要添加主节点

#### Sentinel支持的命令

![img](http://www.chenjunlin.vip/img/redis/sentinel/Sentinel%E6%94%AF%E6%8C%81%E7%9A%84%E5%91%BD%E4%BB%A4.png)

