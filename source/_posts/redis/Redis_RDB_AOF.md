---
title: Redis持久化
date: 2021-05-24 23:33:33
index_img: /img/cover/Redis.jpg
tags: 
- Redis
- 持久化
categories: Redis
mermaid: true
---

### Redis持久化

> 持久化（Persistence），即把数据（如内存中的对象）保存到可永久保存的存储设备中（如磁盘）。持久化的主要应用是将内存中的对象存储在数据库中，或者存储在磁盘文件中、XML数据文件中等等。
>
> 还可以从如下两个层面简单的理解持久化 ：
>
> - 应用层：如果关闭(shutdown)你的应用然后重新启动则先前的数据依然存在。
> - 系统层：如果关闭(shutdown)你的系统（电脑）然后重新启动则先前的数据依然存在。

#### Redis为什么要持久化

> Redis是内存数据库，为了保证效率所有的操作都是在内存中完成。数据都是缓存在内存中，当你重启系统或者关闭系统，之前缓存在内存中的数据都会丢失再也不能找回。因此为了避免这种情况，Redis需要实现持久化将内存中的数据存储起来。

#### **Redis如何实现持久化？**

> Redis官方提供了不同级别的持久化方式：
>
> - RDB持久化：能够在指定的时间间隔能对你的数据进行快照存储。
> - AOF持久化：记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾。Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
> - 不使用持久化：如果你只希望你的数据在服务器运行的时候存在，你也可以选择不使用任何持久化方式。
> - 同时开启RDB和AOF：你也可以同时开启两种持久化方式，在这种情况下当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。

#### RDB(Redis DataBase)

* RDB持久化是把当前进程数据生成快照保存到硬盘的过程,触发RDB持久化过程分为手动触发和自动触发.

* **手动触发分别对应save和bgsave命令**

  * `save`命令: 阻塞当前Redis服务器,直到RDB过程完成为止,对于内存比较大的实例会造成长时间阻塞[**废弃**]
  * `bgsave`命令:Redis进程执行fork操作创建子进程,RDB持久化过程有子进程负责,完成后自动结束.阻塞只发生在fork阶段,一般时间很短

  <img src="http://www.chenjunlin.vip/img/redis/RDB%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png" alt="img" style="zoom:50%;" />

* **除了执行命令手动触发外,Redis内部还存在自动触发的RDB的持久化机制:**

  * 使用save相关配置,如"save m n",表示m秒内数据集存在n次修改时,自动触发`bgsave`;
  * 若从节点执行全量复制操作,主节点自动执行bgsave生成RDB文件并发送给从节点;
  * 执行`debug reload`命令重新加载Redis,也会自动触发`bgsave`操作;
  * 默认情况下执行`shutdown`命令,若没有开启AOF持久化则自动执行`bgsave`;

* **bgsave是主流的触发RDB持久化方式**

  * 执行`bgsave`命令,Redis父进程判断当前是否存在正在执行的子进程,如RDB/AOF子进程,若存在`bgsave`命令直接返回;
  * 父进程执行fork操作创建子进程,fork操作过程中父进程会阻塞,通过`info stats`命令查看`latest_fork_usec`选项,可以获取最近一个fork操作的耗时,单位为微妙;
  * 父进程fork完成后,bgsave命令返回"Backgroud saving started"信息并不再阻塞父进程,可以继续响应其他命令;
  * 子进程创建RDB文件,根据父进程内存生成临时快照文件,完成后对原有文件进行原子替换.执行`lastsave`命令可以获取最后一次生成RDB的时间,对应info统计的`rdb_last_time`选项;
  * 进程发送信号给父进程表示完成,父进程更新统计信息,具体见`info Persistence`下rdb_*相关选项.

* **RDB文件的处理**

  * **保存**

    * RDB文件保存在dir配置指定的目录下,文件名通过dbfilename配置指定.可以通过执行`config set dir {newDir}`和`config set dbfilename {newFileName}`运行期动态执行,当下次运行是RDB文件会保存到新的目录中
    * 当遇到坏盘或磁盘写满等情况,可以通过`config set dir {newDir}`在线修改文件路径到可用磁盘路径,之后执行`bgsave`进行磁盘切换,同样试用于AOF持久化文件.

  * **压缩**

    * Redis默认采用LZF算法对生成的RDB文件做压缩处理,压缩后的文件远远小于内存大小,默认开启,可以通过参数`config set rdbcompression {yes|no}`动态修改.
    * 虽然压缩RDB会消耗CPU,但可大幅降低文件的体积,方便保存到硬盘或者通过网络发送给从节点,因此线上建议开启.

  * **校验**

    * 如果Redis加载损坏的RDB文件时拒绝启动,并打印如下日志:

      ```
      # Short read or OOM loading DB. Unrecoverable error,aborting now
      ```

    * 这时可以使用Redis提供的redis-check-dump工具检测RDB文件并获取对应的错误报告.

* **RDB的优缺点**

  * **优点**
    * RDB是一个紧凑压缩的二进制文件,代表Redis在某个时间点上的数据快照,非常使用与备份,全量复制等场景,比如每6个小时bgsave备份,并把RDB文件拷贝到远程机器或者文件系统中hdfs,用于灾难恢复.
    * Redis加载RDB恢复数据远远快于AOF的方式
  * **缺点**
    * RDB方式数据没办法做到实时持久化/秒级持久化,因为bgsave每次运行都要执行fork操作创建子进程,属于重量级操作,频繁执行成本过高.
    * RDB文件使用特定二进制格式保存,Redis版本演进过程中有多个格式的RDB版本,存在老版本Redis服务无法兼容新版RDB格式的问题

#### AOF(append only file)

> AOF持久化:以独立日志的方式记录每次命令,重启时再重新执行AOF文件中命令达到恢复数据的目的.
>
> AOF的主要作用是解决了数据持久化的实时性,目前已经是Redis持久化的主流方式.

* 开启AOF功能需要设置配置:`appendonly yes`,默认不开启.

* AOF文件名通过`appendfilename`配置设置,默认文件名是appendonly.aof.保存路径桶RDB持久化方式一致,通过dir配置指定.

* AOF的工作流程操作: **命令写入(append)**、**文件同步(sync)**、**文件重写(rewrite)**、**重启加载(load)**

  * **命令写入(append)**: 所有的写入命令会追加到`aof_buf(缓冲区)`中;

    * AOF命令写入的内容是文本协议格式,目的如下:
      * 文本协议具有很好的兼容性;
      * 开启AOF后,所有写入命令都包含追加操作,直接采用协议格式,避免了二次处理开销;
      * 文本协议具有可读性,方便直接修改和处理.
    * AOF把命令追加到`aof_buf`中,是因为Redis使用单线程响应命令,如果每次写AOF文件命令都直接追加到硬盘,那么性能完全取决于当前硬盘负载.先写入缓冲区`aof_buf`中,还有另一个好处,Redis可以提供多种缓冲区同步硬盘的策略,在性能和安全性方面做出平衡.

  * **文件同步(sync)**: AOF缓冲区根据对应的策略向硬盘做同步操作;

    * Redis提供多种AOF缓冲区同步文件策略,有参数`appendfsync`控制

      | 可配置值 | 说明                                                         |
      | -------- | ------------------------------------------------------------ |
      | always   | 命令写入aof_buf后调用系统fsync操作同步到AOF文件,fsync完成后线程返回 |
      | everysec | 命令写入aof_buf后调用系统write操作,write完成后线程返回,fsync同步操作由专门线程每秒调用一次 |
      | no       | 命令写入aof_buf后调用系统write操作,不对AOF文件做fsync同步,同步硬盘操作由操作系统负责,通常同步周期最长为30秒 |

    * 系统调用write和fsync说明:

      * write操作会触发延迟写(delayed write)机制.Linux在内核提供页缓冲区用来提高硬盘IO性能.write操作在写入系统缓冲区后直接返回.同步硬盘操作依赖于系统调用度机制,例如:缓冲区也空间写满或达到特定时间周期.同步文件之前,如果此时系统故障宕机,缓冲区内数据将丢失.
      * fsync针对单个文件操作(比如AOF文件),做强制硬盘同步,fsync将阻塞知道写入硬盘完成后返回,保证了数据持久化.

  * **文件重写(rewrite)**: 

    > * 随着AOF文件越来越大,需要定期对AOF文件进行重写,达到压缩的目的;
    > * 随着命令不断写入AOF,文件会越来越大,为了解决这个问题,Redis引入AOF重写机制来压缩文件体积,AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程.
    >
    > * **重写后的AOF文件变小的原因:**
    >   * 进程内已经超时的数据不再写入文件;
    >   * 旧的AOF文件含有无效命令,如 del key1,hdel key2...重写使用进程内数据直接生成,这样新的AOF文件只保留最终数据的写入命令;
    >   * 多条写明了可以合并为一个,为了防止单条命令过大造成客户端缓冲区溢出,对于list,set,hash,zset等类型操作,以64个元素为界拆分为多条.
    >
    > * AOF重写降低了文件占用空间,除此之外,另一个目的是,更小的AOF文件可以更快地被Redis加载;

    * **AOF重写重写可以手动触发和自动触发**:

      * **手动触发**: 直接调用`bgrewriteaof`命令

      * **自动触发**: 根据`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`参数确定自动触发时机;

        * `auto-aof-rewrite-min-size`: 表示运行AOF重写时文件最小体积,默认为64MB;
        * `auto-aof-rewrite-percentage`: 代表**当前AOF文件空间(aof_curent_size)**和**上次重写后AOF文件空间(aof_base_size)**的比值
        * 自动触发时机=**aof_current_size>auto-aof-rewrite-min-size&&(aof_current_size-aof_base_size)/aof_base_size>=auto-auto-aof-rewrite-percentage**
          * 其中`aof_current_size`和`aof_base_size`可以在`info Persistence`统计信息中查看

        ```mermaid
        graph TD
        bgrewriteaof -->|1|父进程
        父进程 -->|2| fork
        fork -->|3.1| aof_buff
        aof_buff --> 旧AOF文件
        fork -->|3.2| aof_rewrite_buf -->|5.2| 新AOF文件
        fork --> 子进程
        子进程 -->|5.1 信号通知父进程| 父进程
        子进程 -->|4| 新AOF文件
        新AOF文件 -->|5.3| 旧AOF文件
        ```

      * 1- 执行AOF重写请求

        * 若当前进程正在执行AOF重写,请求不执行并返回如下响应:

          > ERR Backgroud append only file rewrite already in progress

        * 若当前进程正在执行`bgsave`操作们,重写命令延迟到bgsave完成之后再执行,返回如下响应:

          > Backgroud append only file rewriting sceduled

      * 2-父进程执行fork创建子进程,开销等同于`bgsave`过程

      * 3.1-主进程fork操作完成后,继续响应其他命令.所有修改命令依然写入AOF缓冲区并更具appendfsync策略同步到硬盘,保证原有AOF机制正确性.

      * 3.2-由于fork操作运用**写时复制技术**,子进程只能共享fork操作时的内存数据.由于父进程依然响应命令,Redis使用"AOF重写缓冲区"保存这部分新数据,防止新AOF文件生成期间丢失这部分数据.

      * 4-子进程根据内存快照,按照命令合并规则写入新的AOF文件.每次批量写入硬盘数据量由配置`aof-rewrite-incremental-fsync`控制,默认为32MB,单次刷盘数据过多造成硬盘阻塞.

      * 5.1-新AOF文件写入完成后,子进程发送信号给父进程,父进程更新统计信息,具体见`info persistence`下aof_*相关统计.

      * 5.2-父进程把AOF重写缓冲区的数据写入到新的AOF文件.

      * 5.3-使用新AOF文件替换老文件,完成AOF重写;

  * **重启加载(load)**: 当Redis服务器重启时,可以加载AOF文件进行数据恢复;

    * AOF和RDB文件都可以用于服务器重启时的数据恢复.

    * Redis持久化文件加载流程:

      ``` mermaid
      graph TD
      redis启动-->开启AOF{开启AOF}
      开启AOF{开启AOF}-->|no|存在RDB?{存在RDB?}
      存在RDB?{存在RDB?}-->|no|启动成功
      存在RDB?{存在RDB?}-->|yes|加载RDB
      加载RDB-->成功?{成功?}
      成功?{成功?}-->|yes|启动成功
      成功?{成功?}-->|no|启动失败
      开启AOF{开启AOF}-->|yes|存在AOF?{存在AOF?}
      存在AOF?{存在AOF?}-->|no|存在RDB?{存在RDB?}
      存在AOF?{存在AOF?}-->|yes|加载AOF
      加载AOF-->成功?{成功?}
      ```

      

    * 1- AOF持久化开启且存在AOF文件时,优先加载AOF文件,打印如下日志

      > DB loaded from append only file: 5.841 seconds

    * 2- AOF关闭或者AOF文件不存在时,加载RDB文件,打印如下日志:

      > DB loaded from disk: 5.586 seconds

    * 3- 加载AOF/RDB文件后,Redis启动成功;

    * 4- AOF/RDB文件存在错误时,Redis启动失败并打印错误信息;

* **文件校验**

  * 加载损坏的AOF文件时会拒绝启动,并打印如下日志:

    > Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>

    * 对于错误格式的AOF文件,先进行备份,然后采用`redis-check-aof --fix`命令进行修复,修复后使用`diff -u`对比数据的差异,找出丢失的数据,有些可以人工修改不全.

    > AOF文件可能存在结尾不完整的情况,比如机器突然掉电导致AOF尾部文件命令写入不全.Redis为我们提供了`aof-load-truncated`配置来兼容这种情况,默认开启,加载AOF时,当遇到此问题是会忽略并继续启动,同时打印如下告警日志:
    >
    > \# !!! Waring: short read while loading the AOF file !!!
    >
    > \# !!! Truncating the AOF at offset 3997856725!!!
    >
    > \# AOF loaded anyway beacause aof-load-truncated is enabled

* **AOF追加阻塞**

  * 当开启AOF持久化时,常用的同步硬盘的策略是`everysec`,用于平衡性能和数据安全性.对于这种方式,Redis使用另一条线程每秒fsync同步硬盘,当系统硬盘资源繁忙时,会造成Redis主线程阻塞.

    ``` mermaid
    graph TB
    
    主线程 -->|1| AOF缓冲区
    AOF缓冲区 -->|2| 同步线程
    同步线程 --> 同步磁盘
    AOF缓冲区 -->|3| 对比上次fsync时间{对比上次fsync时间}
    对比上次fsync时间{对比上次fsync时间} -->|大于2秒| 阻塞 
    对比上次fsync时间{对比上次fsync时间} -->|小于2秒| 成功
    ```

* **阻塞流程分析**

  * 主线程复制写入AOF缓冲区
  * AOF线程复制每秒执行一次同步磁盘操作,并记录最近一次同步时间
  * 主线程复制对比上次AOF时间
    * 若距上次同步成功时间在2秒内,主线程直接返回;
    * 若距上次同步成功时间超过2秒,主线程将会阻塞,直到同步操作完成

* **AOF阻塞流程可以发现两个问题**

  * everysec配置最多可能丢失2秒数据,而不是1秒;
  * 若系统fsync缓慢,将会导致Redis主线程阻塞影响效率.

* **AOF阻塞问题定位**

  * 发生AOF阻塞时,Redis输出如下日志,用于记录AOF fsync阻塞导致拖慢Redis服务的行为:

    > Asynchronous AOF fsync is taking too long (disk is busy).Wirting the AOF buffe without waiting for fsync to complete,this may slow down Redis

  * 每当发生AOF追加阻塞事件发生时,在`info Persistence`统计中,aof_delayed_fsync指标会累加,查看这个指标方便定位AOF阻塞问题.

  * AOF同步最多运行2秒的延迟,当延迟发生时说明硬盘存在搞复杂问题,可以通过监控工具如iotop,定位消耗硬盘IO资源.

#### RDB和AOF的优缺点

##### RDB优点

* RDB 是一个非常紧凑的文件,它保存了某个时间点的数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集。
* RDB 是一个紧凑的单一文件,很方便传送到另一个远端数据中心，非常适用于灾难恢复。
* RDB 在保存 RDB 文件时父进程唯一需要做的就是 fork 出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他 IO 操作，所以 RDB 持久化方式可以最大化 Redis 的性能。
* 与AOF相比,在恢复大的数据集的时候，RDB 方式会更快一些。

##### AOF优点

* 你可以使用不同的 fsync 策略：无 fsync、每秒 fsync 、每次写的时候 fsync .使用默认的每秒 fsync 策略, Redis 的性能依然很好( fsync 是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据。
* AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题。
* Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写：重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
* AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。导出（export） AOF 文件也非常简单：举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

##### **RDB缺点**

- Redis 要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在 Redis 意外宕机,你可能会丢失几分钟的数据。
- RDB 需要经常 fork 子进程来保存数据集到硬盘上,当数据集比较大的时候, fork 的过程是非常耗时的,可能会导致 Redis 在一些毫秒级内不能响应客户端的请求。

##### **AOF缺点**

- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
- 数据恢复（load）时AOF比RDB慢，通常RDB 可以提供更有保证的最大延迟时间。

##### **RDB和AOF简单对比总结**

**RDB优点**：

- RDB 是紧凑的二进制文件，比较适合备份，全量复制等场景
- RDB 恢复数据远快于 AOF

**RDB缺点**：

- RDB 无法实现实时或者秒级持久化；
- 新老版本无法兼容 RDB 格式。

**AOF优点**：

- 可以更好地保护数据不丢失；
- appen-only 模式写入性能比较高；
- 适合做灾难性的误删除紧急恢复。

**AOF缺点**：

- 对于同一份文件，AOF 文件要比 RDB 快照大；
- AOF 开启后，会对写的 QPS 有所影响，相对于 RDB 来说 写 QPS 要下降；
- 数据库恢复比较慢， 不合适做冷备。