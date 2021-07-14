---
title: Redis-主从复制
date: 2021-05-24 23:33:33
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags: 
- 主从复制
categories:
- Redis
mermaid: true 
updated: 2021-07-13 22:41:24
---

### Redis主从复制

> 参与复制的Redis实例划分为主节点(master)和从节点(slave).
>
> **主从复制的开启，完全是在从节点发起的；不需要我们在主节点做任何事情。**
>
> 默认情况下,Redis都是主节点.每个从节点只能有一个主节点,而主节点可以同时有多个从节点.
>
> **复制的数据流是单向的,只能由主节点复制到从节点;**

#### Redis主从复制的作用

* **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
* **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
* **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务，分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
* **高可用基石**：主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

<img src="http://www.chenjunlin.vip/img/redis/copy/Redis%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E4%B8%89%E4%B8%AA%E9%98%B6%E6%AE%B5.png" alt="img" style="zoom:80%;" />

#### **Redis 的同步机制**

> Redis 可以使用主从同步，从从同步。第一次同步时，主节点做一次 bgsave，并同时将后续修改操作记录到内存 buffer，待完成后将 rdb 文件全量同步到复制节点，复制节点接收完成后将 rdb 镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。

#### **建立复制** 

* 配置复制的方式有三种:
  * 在配置文件中加入`slaveof {masterHost} {masterPort}`随Redis启动生效
  * 在`redis-server`启动命令后加入`--slaveof {masterHost} {masterPort}`
  * 登录客户端后,直接使用命令:`slaveof {masterHost} {masterPort}`

* `slaveof`本身是异步命令,执行`slaveof`命令时,节点只保存主节点信息后返回,后续复制流程在节点内部异步执行.
    * 可以通过`info replication`查看建立的信息;
  * `slaveof`还可以实现**切主节点操作**,`slaveof {newMasterIp} {newMasterPort}`,主要流程:
    * 断开与旧节点复制关系
    * 与新主节点建立复制关系
    * **删除从节点当前所有数据**
    * 对新主节点进行复制操作
    * **注: 从节点首次与主节点建立主从关系时,也会经历切主节点操作,即会"删除从节点当前所有数据"**
  * 若主节点设置了`requirepass`参数进行密码验证,这时从节点与主节点的复制连接是通过一个特殊标识的客户端来完成的,因此需要在从节点配置`masterauth`参数与主节点密码保持一致,这样才能保证从节点正确的连接到主节点并发起复制流程;

  ```
  建立主从复制成功后,从节点Redis服务端日志: 
  18378:S 24 Apr 2021 22:30:04.627 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize w
  18378:S 24 Apr 2021 22:30:04.627 * REPLICAOF 127.0.0.1:6379 enabled (user request from 'id=4 addr=127.0.0.1:35846 fd=8 name= age=54 idle=0 flags=N db=0 sub=0 s=r cmd=slaveof user=default')
  18378:S 24 Apr 2021 22:30:05.542 * Connecting to MASTER 127.0.0.1:6379
  18378:S 24 Apr 2021 22:30:05.543 * MASTER <-> REPLICA sync started
  18378:S 24 Apr 2021 22:30:05.543 * Non blocking connect for SYNC fired the event.
  18378:S 24 Apr 2021 22:30:05.543 * Master replied to PING, replication can continue...
  18378:S 24 Apr 2021 22:30:05.543 * Trying a partial resynchronization (request 4d21b3738e5cd62e4819cecf0a1b6aa78043b621:1).
  18378:S 24 Apr 2021 22:30:05.544 * Full resync from master: 87fcdf7ac1958375aeea7e62a27427fb28d24f59:0
  18378:S 24 Apr 2021 22:30:05.544 * Discarding previously cached master state.
  18378:S 24 Apr 2021 22:30:05.616 * MASTER <-> REPLICA sync: receiving 175 bytes from master to disk
  18378:S 24 Apr 2021 22:30:05.617 * MASTER <-> REPLICA sync: Flushing old data
  18378:S 24 Apr 2021 22:30:05.617 * MASTER <-> REPLICA sync: Loading DB in memory
  18378:S 24 Apr 2021 22:30:05.617 * Loading RDB produced by version 6.0.5
  18378:S 24 Apr 2021 22:30:05.617 * RDB age 0 seconds
  18378:S 24 Apr 2021 22:30:05.617 * RDB memory usage when created 1.85 Mb
  18378:S 24 Apr 2021 22:30:05.617 * MASTER <-> REPLICA sync: Finished with success
  
  
  建立主从复制成功后,主节点Redis服务端日志:
  1263:M 24 Apr 2021 22:17:00.406 * Replica 127.0.0.1:6380 asks for synchronization
  1263:M 24 Apr 2021 22:17:00.406 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '880b469fd4d33a88e0a105a506de61542933659d', my replication IDs are 'a058d0cf5dbe08daff7afb8422488c347a71dfa3' and '0000000000000000000000000000000000000000')
  1263:M 24 Apr 2021 22:17:00.406 * Replication backlog created, my new replication IDs are 'b05ab495c0a8821102208dccc0030e6c57edd0cb' and '0000000000000000000000000000000000000000'
  1263:M 24 Apr 2021 22:17:00.406 * Starting BGSAVE for SYNC with target: disk
  1263:M 24 Apr 2021 22:17:00.407 * Background saving started by pid 12633
  12633:C 24 Apr 2021 22:17:00.411 * DB saved on disk
  12633:C 24 Apr 2021 22:17:00.411 * RDB: 0 MB of memory used by copy-on-write
  1263:M 24 Apr 2021 22:17:00.445 * Background saving terminated with success
  1263:M 24 Apr 2021 22:17:00.445 * Synchronization with replica 127.0.0.1:6380 succeeded
  
  
  Redis3.0之后在输出的日志开头会有**M,S,C**等标志,对应的含义是:
  M=当前为主节点日志
  S=当前为从节点日志
  C=子进程日志
  ```

#### **断开复制**

  * 在从节点执行`slaveof no one`来断开与主节点复制关系
  * 断开复制主要流程:
    * 断开与主节点复制关系;
    * 从节点晋升为主节点;
  * 从节点断开后并不会删除原来复制过来的数据,只是无法再获取主节点上的数据变化;

* **配置从节点只读**

  > 默认情况下,从节点使用`slave-read-only=yes`配置为只读模式.由于复制只能从主节点到从节点,对于从节点的任何修改主节点都无法感知,修改会造成主从数据不一致.

* **传输延迟**

  * 主从节点一般部署在不同机器上,复制时的网络延迟成为需要考虑的问题,Redis提供了`repl-disable-tcp-nodelay`参数用于控制是否关闭**TCP_NODELAY**,默认关闭
    * 关闭时,主节点产生的命令数据无论大小都会及时地发送给从节点,这样主从之间延迟会变小,但增加了网络带宽的消耗.适合主从之间网络环境良好的场景.
    * 开启时,主节点会合并较小的TCP数据包从而节省带宽.默认发送时间间隔取决于Linux的内核,一般默认为40毫秒.这样配置节省了带宽但增加了主从之间的延迟.适用于主从网络环境复杂或带宽紧张的场景.

#### **拓扑结构**

  * Redis的复制拓扑结构可以支持单层或多层复制关系:
    * **一主一从**(故障转移)
      * 用于主节点出现宕机是从节点提供故障转移支持.当应用写命令并发量较高且需要持久化时,可以只在从节点上开启AOF,这样既保证数据安全性同时也避免了持久化对主节点的性能干扰.
      * 但需要注意的是,当主节点关闭持久化功能时,如果主节点脱机要避免自动重启操作.因为主节点之前没有开启持久化功能自动重启数据集为空,这时从节点如果继续复制从节点会导致从节点数据也被清空的情况,丧失了持久化的意义.
      * 安全的做法是在从节点上执行`slaveof no one`断开与主节点的复制关系,再重启主节点从而避免这个问题;
    * **一主多从**(读写分离)
      * 又称为星形拓扑结构使得应用端可以利用多个节点实现读写分离.
      * 对于读占比较大的场景,可以把读命令发送到从节点来分担主节点压力.同时在日常开发中如果需要执行一些比较耗时的读命令,如keys,sort等,可以在其中一台从节点上执行,防止慢查询对主节点造成阻塞从而影响线上服务的稳定性.
      * 对于写并发量较高的场景,多个从节点会导致主节点写命令的多次发送从而过度消耗网络带宽,同时也加重了主节点的负载影响服务稳定性.
    * **树状主从结构**
      * 又称为树状拓扑结构使用从节点不但可以复制主节点数据,同时可以作为其他从节点的主节点继续向下层复制.通过引入复制中间层,可以有效降低主节点负载和需要传送给从节点的数据量.
      
      * 数据实现一层一层的向下复制,当主节点需要挂载多个节点时为了避免对主节点的性能干扰,可以采用树状主从结构降低节点压力.

#### 建立主从复制流程

* **1. 保存主节点信息**

  * 执行`slaveof`后从节点只保存主节点的地址信息便直接返回,这时建立复制流程还没开始,在从节点上执行`info replication`可以看到如下信息:

    ```
    master_host:127.0.0.1
    master_port:6379
    master_link_status:down
    master_last_io_seconds_ago:-1
    master_sync_in_progress:0
    ```

  * 执行`slaveof`后Redis服务端会打印如下日志:

    ```
    SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=2 addr=127.0.0.1:8317 fd=7 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
    ```

* **2. 主从建立socket连接**

  * 从节点(slave)内部通过每秒运行的定时任务维护复制相关逻辑,当定时任务发现存在新的主节点后,会尝试与该节点建立网络连接,从节点会建立一个socket套接字,**专门用于接受主节点发送的复制命令**;
  * **如果从节点无法建立连接,定时任务会无限重试知道成功或者执行`slaveof no one`取消复制**

* **3. 发送ping命令**

  * 连接建立成功后从节点发送ping请求进行首次通信,ping请求主要目的如下

    * **检测主从节点之间网络套接字是否可用;**
    * **检测主节点当前是否可以接受命令;**

  * 若发送ping命令后,从节点没有收到主节点的pong回复或者超时,从节点会断开复制连接,下次定时任务会发起重连;

    ```
    从节点发送的ping命令返回,Redis打印如下日志
    [23912] 24 Apr 20:11:45.188 * Master replied to PING, replication can continue...
    ```

* **4.权限验证**

  * 若主节点设置了`requirepass`参数,则需要密码验证,从节点必须配置masterauth参数保证与主节点相同的密码才能通过验证;如果验证失败复制将终止,从节点重新发起复制流程;

* **5.同步数据集**

  * 主从复制连接正常通信后,对于首次建立复制的场景,主节点会把持有的数据全部发送给从节点,这部分操作是耗时最长的步骤.Redis在2.8版本以后采用新的复制命令`psync`进行数据同步,原来的`sync`命令依然支持,保证新旧版本的兼容性.

* **6.命令持续复制**

  * 当主节点把当前的数据同步给从节点后,便完成了复制的建立流程,然后主节点会持续地把写命令发送给从节点,保证主从数据一致性.


#### **数据同步**

>  Redis在2.8版本以后采用新的复制命令`psync`进行数据同步,同步过程分为:**全量复制和部分复制**.

  * **全量复制:**
    * 一般用于初次复制场景,Redis早期支持的复制功能只有全量复制,它会把主节点全部数据一次性发送给从节点,当数据量较大时,会对主从节点和网络造成很大的开销;
  * **部分复制:**
      * 用于处理在主从复制中因网络闪断等原因造成的数据丢失场景,当从节点再次连上主节点后,如果条件允许,主节点会补发丢失数据给从节点.因为补发的数据远远小于全量数据,可以有效避免全量复制的过高开销;
      * 部分复制是对老版复制的重大优化,有效避免了不必要的全量复制操作.因此当使用功能时,尽量采用2.8以上版本的Redis.
      * `psync`命令运行需要一下组件支持:
        * 主从节点各自复制偏移量;
        * 主节点复制积压缓冲区;
        * 主节点运行id;


  * **复制偏移量**

    * 参与复制的主从节点都会维护自身复制偏移量.**主节点(master)**在处理完写入命令后,会把命令的字节长度做累加记录,统计信息在`info replication` 中的`master_repl_offset`指标中;
    * 从节点(slave)每秒种上报自身的复制偏移量给主节点,因此主节点也会保存从节点的复制偏移量;
    * **从节点(slave)**在接收到主节点发送的命令后,也会累加记录自身的偏移量.统计信息在`info replication`中的`slave_repl_offset`指标中;
    * 可以通过主节点的统计信息,计算出**master_repl_offset**-**slave_repl_offset**字节量,判断主从节点复制相差的数据量,根据这个差值判定当前复制的健康度.如果主从之间复制偏移量相差较大,则可能是网络延迟或命令阻塞等原因引起;

  * **复制积压缓冲区**

      * **复制积压缓冲区是保存在主节点上的一个固定长度的队列,默认大小为1MB**,当主节点有连接从节点(slave)时被创建,这时主节点(master)响应写命令时,不但会把命令发送给从节点,还会写入复制积压缓冲区;

      ```mermaid
      graph LR
      master-->|传播命令|slave
      master-->累加主偏移量
      slave-->累加从偏移量
      style 累加主偏移量 stroke:#000,stroke-width:2px,stroke-dasharray:5,5;
      style 累加从偏移量 stroke:#000,stroke-width:2px,stroke-dasharray:5,5;
      ```

      ```mermaid
      graph LR
      master-->|传播命令|slave
      master-->复制积压缓冲区
      style 复制积压缓冲区 stroke:#000,stroke-width:2px,stroke-dasharray:5,5;
      ```

      * 由于缓冲区本质上是先进先出的定长队列,所以能实**现保存最近已复制数据的功能**,用于部分复制和复制命令丢失的数据补救.复制缓存区相关统计信息保存在主节点的`info replication`中:

        ```
        # Replication
        role:master
        connected_slaves:1
        slave0:ip=127.0.0.1,port=6380,state=online,offset=2899,lag=1
        master_repl_offset:2899
        // 开启复制缓冲区
        repl_backlog_active:1
        // 缓存区最大长度
        repl_backlog_size:1048576
        // 起始偏移量,计算当前缓存区可用范围
        repl_backlog_first_byte_offset:2
        // 已保存数据的有效长度
        repl_backlog_histlen:2898
        ```

      * 根据统计指标,可计算出复制积压缓冲区内的可用偏移量范围[repl_backlog_first_byte_offset,repl_backlog_first_byte_offset+repl_backlog_histlen]

  * **主节点运行ID**

      * 每个Redis节点启动后都会动态分配一个40位的十六进制字符串作为运行ID.运行ID的主要作用是用来唯一识别Redis节点.如果只使用ip+port的方式识别主节点,那么主节点重启变更了整体数据集(替换RDB/AOF文件),从节点再基于偏移量复制数据将是不安全的,因此当运行ID变化后从节点将做全量复制.

      * 如何在不改变运行ID的情况下重启呢?

        * 当需要调优一些内存相关配置,例如:`hash-max-ziplist-value`等,这些配置需要Redis重新加载才能已存在的数据,这时可以使用`redis-cli debug reload`命令重新加载RDB保持运行ID不变,从而有效避免不必要的全量复制.

          ```
          redis-cli -p 6379 info server |grep run_id
          redis-cli debug reload
          redis-cli -p 6379 info server |grep run_id
          ```

        * **`debug reload`命令会阻塞当前Redis节点的主线程,阻塞期间会删除本地RDB快照并清空数据之后再加载RDB文件.隐藏对于大数据量的主节点和无法容忍阻塞的场景,谨慎使用**

  * **psync命令**

      * 从节点使用psync命令完成部分复制和全量复制功能

        * 命令格式:`psync {runId} {offset}`

        * runId: 从节点所复制主节点的运行id;

        * offset:当前从节点已复制的数据偏移量;

          ```mermaid
          sequenceDiagram
              participant slave
              participant master
                  slave ->> master : "psync {runId} {offset}"
                  master->> slave  : "1: +FULLRESYNC"
                  master->> slave : "2: +CONTINUE"
                  master->> slave : "3: +ERR"
          ```

    * 从节点(slave)发送psync命令给主节点,参数runId是当前从节点保存的主节点运行ID,如果没有则默认值为-1,参数offset是当前从节点保存的复制偏移量,如果是第一次参与复制则默认值为-1

      * 主节点(master)根据psync参数和自身数据情况决定响应结果:
        * 若回复"+FULLRESYNC",那么从节点将触发全量复制流程;
        * 若回复"+CONTINUE": 从节点将触发部分复制流程;
        * 若回复"+ERR": 说明主节点版本低于Redis2.8,无法识别psync命令,从节点将发送旧版的sync命令触发全量复制流程;

  * **全量复制流程说明**

    1. 发送psync命令进行数据同步,由于是第一次进行复制,从节点没有复制偏移量和主节点的运行ID,所以发送`psync -1`

    2. 主节点根据`psync -1`解析出当前为全量复制,回复"`+FULLRESYNC `"响应;

    3. 从节点接收主节点的响应数据保存运行ID和偏移量offset,执行到当前步骤是从节点打印如下日志:

       ```
       18378:S 24 Apr 2021 22:30:05.543 * Trying a partial resynchronization (request 4d21b3738e5cd62e4819cecf0a1b6aa78043b621:1).
       18378:S 24 Apr 2021 22:30:05.544 * Full resync from master: 87fcdf7ac1958375aeea7e62a27427fb28d24f59:0
       ```

    4. 主节点执行`bgsave`保存RDB文件到本地;

    5. 主节点发送RDB文件给从节点,从节点把接收到的RDB文件保存在本地直接作为从节点的数据文件,接收完RDB后从节点打印相关日志,可以在日志看到主节点发送的数据量.

       ```
        MASTER <-> REPLICA sync: receiving 175 bytes from master to disk
       ```

    * **注**:对于数据量较大的主节点,比如生成的RDB文件超过6GB以上要格外小心.传输文件这一步操作非常耗时,速度取决于主从节点之间的网络带宽,通过细致分析Full resync和 MASTER <-> REPLICA可以算出RDB文件从创建到传输完毕消耗的总时间.
      * 若总时间超过`repl_timeout`所配置的值(默认60秒),从节点将放弃接受RDB文件并清理已经下载的临时文件,导致全量复制失败.
        * 针对数据量较大的节点,建议调大`repl_timeout`参数防止出现全量同步数据超时;
      * 无盘复制:通过`repl-diskless-sync`参数控制,默认关闭;
        * 使用场景: 主节点所在机器磁盘性能较差但网络带宽比较充裕的场景;

    6. 对于从节点开始接收RDB快照到接收完成期间,主节点任然响应读写命令,因此主节点会把这期间写命令数据保存在**复制客户端缓冲区**内,当从节点加载完RDB文件后,主节点再把缓冲区内的数据发送给从节点,保证主从节点之间数据一致性.
       * 若主节点创建和传输RDB的时间过长,对于高流量写入场景非常容易造成主节点复制客户端缓冲区溢出.默认配置为`client-output-buffer-limit slave 256MB 64MB 60`
         * 若60秒内缓冲区消耗持续大于64MB或者直接超过256MB,主节点将直接关闭复制客户端连接,造成同步失败
         * 需要根据主节点数据量和写命令并发量调整`client-output-buffer-limit slave`配置,避免全量复制期间客户端缓冲区溢出
    7. 从节点接收完主节点传送来的全部数据会清空自身旧数据;
    8. 从节点清空数据后开始加载RDB文件,对于较大的RDB文件,这一步依然比较耗时.
       * 对于线上做读写分离的场景,从节点也负责响应读命令.如果此时从节点正处于全量复制阶段或者复制中断,那么从节点响应读命令可能拿到过期或者错误数据.
         * 对于这种场景,Redis复制提供了`replica-serve-stale-data`参数,默认开启状态.如果开启则从节点依然响应所有命令
         * 对于无法容忍数据不一致的应用场景可以设置no来关闭执行,此时从节点除了info和slaveof命令之外所有命令只返回"SYNC with master in progress"
    9. 从节点成功加载完RDB后,如果当前节点开启了AOF持久化功能,它会立刻做`bgrewirteaof`操作,为了保证全量复制后AOF持久化文件立刻可用.

    ![img](http://www.chenjunlin.vip/img/redis/copy/Redis%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6-%E5%85%A8%E9%87%8F%E5%A4%8D%E5%88%B6.png)

  * **全量复制主要开销**

    * 主节点bgsave时间

    * RDB文件网络传输时间
    * 从节点清空数据时间

    * 从节点加载RDB时间
    * 可能的AOF重写时间

  * **部分复制**

    * 部分复制主要是Redis针对全量复制的过高开销做出的一种优化措施,使用`psync {runId} {offset}`命令实现.
      * 当从节点正在复制主节点时,若出现网络闪断或者命令丢失等异常情况是,从节点会向主节点要求补发丢失的命令数据;若主节点的复制积压缓冲区内存在这部分数据则直接发给从节点,从而保证数据一致性.

  * **部分复制流程分析**

    1. 当主从节点之间出现网络中断时,若超过`repl-timeout`时间,主节点会认为从节点故障并中断复制连接,
    2. 主从连接中断期间主节点依然响应命令,但因复制连接中断命令无法发送给从节点,不过主节点内部存在的复制积压缓冲区,依然可以保存最近一段时间的写命令数据,默认最大缓存1MB;
    3. 当主从节点网络恢复后,从节点会再次连上主节点,并打印连接日志;
    4. 当主从连接恢复后,由于从节点之前保存了自身已复制的偏移量和主节点的运行ID.因此会把它们当做`psync`参数发给主节点,要求进行部分复制操作.
    5. 主节点接到`psync`命令后首先核对参数runId是否与自身一致,如果一致,说明之前的是当前主节点,之后根据参数offset在自身**复制积压缓冲区**查找,如果偏移量之后的数据存在缓冲区中,则对从节点发送"+CONTINUE"响应,表示可以进行部分复制.从节点接到回复后打印日志;
    6. 主节点根据偏移量把复制积压区里的数据发送给从节点,保证主从复制进入正常状态.
    
    <img src="http://www.chenjunlin.vip/img/redis/copy/Redis%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6-%E5%A2%9E%E9%87%8F%E5%A4%8D%E5%88%B6.png" alt="img" style="zoom:80%;" />

#### 心跳

> 主从节点在建立复制后,它们维护着长连接并彼此发送心跳命令

```mermaid
graph LR
master-->|ping|slave
slave-->|"replconf ack {offset}"|master
```

* **主从心跳判断机制**

  1. 主从节点彼此都有心跳检测机制,各自模拟成对方的客户端进行通信,通过`client list`命令查看复制相关客户端信息,主节点的连接状态为flags=M,从节点的连接状态为flags=S

     ```
     client list 主节点
     id=6 addr=127.0.0.1:8383 fd=8 name= age=96 idle=1 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=replconf
     
     client list 从节点
     id=25 addr=127.0.0.1:6379 fd=8 name= age=24 idle=5 flags=M db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
     ```

  2. 主节点默认每隔10秒对从节点发送ping命令,判断从节点的存活性和连接状态,可通过参数`repl-ping-replica-period`控制发送频率;

  3. 从节点在主线程中每隔1秒发送`replconf ack {offset}`命令,给主节点上报自身当前的复制偏移量.`replconf `命令主要作用:

     * 实时监测主从节点网络状态
     * 上报自身复制偏移量,检查复制数据是否丢失,如果从节点数据丢失,再从主节点的复制缓冲区拉取丢失数据;
     * 实现保证从节点的数据量和延迟性功能,通过`min-replicas-to-write,min-replicas-max-lag`参数配置定义;

  4. 主节点根据`replconf`命令判断从节点超时时间,体现在`info replication`统计中的lag信息中

     * lag表示从节点最后一次通信延迟的秒数,正常延迟应该在0和1之间.如果超过`repl-timeout`配置的值(默认60秒),则判定从节点下线并断开复制客户端连接.即使主节点判定从节点下线后,如果从节点重新恢复,心跳检测会继续进行.

#### 异步复制

> 主节点不但负责数据读写,还负责把写命令同步给从节点.写命令的发送过程是异步完成,也就是说主节点自身处理完写命令后直接返回给客户端,并不等待从节点复制完成.

* 主节点复制流程
  * 主节点接收处理命令
  * 命令完成后返回响应结果
  * 对于修改命令异步发送给从节点,从节点在主线程中执行复制的命令

#### 可能遇到的问题

* **读写分离**
  * **复制数据延迟**
    * Redis复制数据的延迟由于异步复制特性是无法避免的,延迟取决于**网络带宽**和**命令阻塞**情况;
      * 比如刚在主节点写入数据后立即在从节点上读取可能获取不到,需要业务场景允许短时间内的数据延迟.
      * 对于无法容忍大量延迟的场景,可以编写外部监控程序监听主从节点的复制偏移量,当延迟较大时触发报警或者通知客户端避免读取延迟过高的从节点.
  * **读到过期数据**
    * 当主节点存储大量设置超时的数据时,此时数据大量超时,主节点的采样速度跟不上过期速度且主节点没有读取过期键的操作,那么从节点将无法收到del命令.此时在从节点上可以读取到已经超时的数据.Redis在3.2版本解决了这个问题,从节点读取数据之前会检查键的过期时间来决定是否返回数据.
  * **从节点故障**
  
* **主从配置不一致**

  >  对于有些配置主从之间是可以不一致的,如主节点关闭AOF在从节点上开始,但是在对于内存相关配置必须要一直,如`maxmemory`,`hash-max-ziplist-entries`等参数

* **规避全量复制**

  * **第一次建立复制**: 建议在低峰时进行操作,或者尽量规避使用大数据量的Redis节点
  * **运行节点ID不匹配**
    * 当主从复制关系建立后,从节点会保存主节点的运行ID,若此时主节点因故障重启,那么它的运行ID就会改变,从节点发现主节点运行ID不匹配,会认为自己复制的是一个新的主节点从而进行全量复制.
  * **复制积压缓冲区不足**
    * 当主从节点网络中断,从节点再次连上主节点时会发送`psync {runId} {offset} `命令请求部分复制,若此时请求的偏移量不在主节点的复制积压缓冲区内,则无法提供给从节点数据,因此部分复制会退化为全量复制.

* **规避复制风暴**

  > 复制风暴指的是大量从节点对同一主节点或者对同一台机器的多个主节点短时间内发起全量复制的过程.
  >
  > 复制风暴会对发起复制的主节点或者机器造成大量开销,导致CPU,内存,宽带消耗.

  * **单节点复制风暴**
    * **解决方案**
      * 减少主节点挂载从节点的数据量
      * 采用树型复制结构,加入中间层从节点用来保护主节点
