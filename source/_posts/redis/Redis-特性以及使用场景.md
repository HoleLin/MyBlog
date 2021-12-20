---
title:  Redis-特性以及使用场景
date: 2021-05-24 23:33:33
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags:
- 特性以及使用场景
categories: Redis
mermaid: true
---

### Redis特性以及使用场景

#### Redis的特性

* **速度快**
  * Redis所有数据都放在内存中;
  * Redis是用C语言实现的,一般来说C语言实现的程序离操作系统更近,执行速度相对会更快;
  * Redis使用了单线程架构,预防了多线程可能产生的竞争问题;
* **基于键值对的数据结构服务器**
  * Redis 全称为REmote Dictionary Server,它主要提供了5种数据结构:**字符串**、**哈希**、**列表**、**集合**、**有序列表**，同时在字符串的基础上演变出了位图（Bitmaps）和**HyperLogLog**,随着(LBS[Location Based Service 基于位置服务]的发展,Redis3.2版本中加入了有关**GEO(地理信息定位)**的功能.
* **丰富的功能**
  * 提供了键过期功能,可以用来实现缓存;
  * 提供发布订阅功能,可以实现消息系统;
  * 支持Lua脚本,可以利用Lua创造新的Redis命令;
  * 提供了简单的事务功能,能在一定程度上保证事务特性;
  * 提供了**流水线(Piepeline)**功能,客户端可以将一批命令一次性传到Redis中,减少了网络的开销;
* **简单稳定**
* **客户端语言多**
  * Redis提供了简单TCP通信协议,客户端语言几乎涵盖了主流的编程语言,如Java,Python,C,C++,Nodejs等.
* **持久化**
  * Redis提供了两种持久化方式:RDB和AOF,即可以用两种策略将内存的数据保存到硬盘中.
* **主从复制**
  * Redis提供复制给你,实现了多个相同数据的Redis副本,复制功能是分布式Redis的基础.
* **高可用和分布式**
  * Redis从2.8版本正式提供了高可用实现Redis Sentinel,它能够保证Redis节点的故障发现和故障自动转移;
  * Redis从3.0版本正式提供了分布式筛选Redis Cluster,它是Redis真正的分布式实现,提供了高可用,读写和同类的扩展.

##### Redis缺点

> Redis的主要缺点是数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。

#### Redis使用场景

* **缓存**
  * 会话缓存（Session Cache）
  * 全页缓存（FPC）
* **排行榜系统**
  * 列表和有序集合数据结构
* **计数器应用**
  * 视频网站播放次数,电商网站浏览数
  * Redis天然支持计数功能而且计数性能也非常好
* **社交网络**
* **消息队列系统**

#### Redis不可做的场景

* **从数据规模的角度来看**
  * 数据可分为大规模数据和小规模数据,对于大规模数据Redis不是很适合;
* **从数据冷热的角度来看**
  * 数据可分为冷数据和热数据,热数据通常为频繁操作的数据,反之为冷数据.对于冷数据的处理,Redis不是很合适.热数据可以放在Redis中加速读写,也可以减轻后端存储的负载.

#### **Redis Shell**

| 可执行文件       | 作用                               |
| ---------------- | ---------------------------------- |
| redis-server     | 启动Redis                          |
| redis-cli        | Redis命令行客户端                  |
| redis-benchmark  | Redis基准测试工具                  |
| redis-check-aof  | Redis AOF 持久化文件检测和修复工具 |
| redis-check-dump | Redis RDB 持久化文件检测和修复工具 |
| redis-sentinel   | 启动Redis Sentinel                 |

* **启动Redis**

  * 有三种启动方式

    * 默认配置启动

      ```shell
      ./redis-server
      ```

    * 运行配置启动

      ```shell
      ./redis-server --configKey1 configValue1 --configKey2 configValue2
      ```

    * 配置文件启动

      ```shell
      ./redis-server ${dir}/redis.conf
      ```

* **Redis命令行客户端**

  * 交互方式: `redis-cli -h {host} -p {port}`
  * 命令方式: `redis-cli -h {host} -p {port} {command}`
  * 停止服务: `redis-cli shutdown`
    * Redis关闭的过程: 断开与客户端的连接,持久化文件生成,是一种相对优雅的关闭方式.
    * 除了可以使用shutdown命令关门Redis服务外,还可以使用kill进程的方式关闭Redis,但是不要使用kill -9强制杀死Redis服务的方式,这样Redis不会做持久化操作,还会造成缓冲区等资源不能被优雅关闭,极端情况会造成AOF和复制丢失数据的情况;
    * `redis-cli  shutdown nosave|save`:代表是否在关闭Redis之前,生成持久化文件

* **redis-cli**

  * -r(repeat):代表命令将执行多次;
    * `redis-cli -r 3 ping`
  * -i(interval):代表每个几秒执行一次命令,但是-i选项必须和-r选项一起使用,**单位是秒,不支持毫秒为单位**;
  * -x:代表从标准输入(stdin)读取数据作为redis-cli的最后一个参数;
    * `echo "world" | redis-cli -x set hello`
  * -c(cluster):连接Redis cluster节点时需要使用,-c选项可以防止moved和ask异常;
  * -a(auth):若Redis配置了密码,则需要使用;
  * --scan和--pattern用于扫描指定模式的键,相当于使用sacan命令;
  * --slave:把当前客户端模拟成当前Redis节点的从节点,可以用来获取当前节点的更新操作;
  * --rdb:会请求Redis实例生成并发送RDB持久化文件,保存在本地;可以用来持久化文件的定期备份
  * --pipe:用于将命令封装成Redis同行协议定的格式,批量发送给Redis执行;
  * --bigkeys: 使用scan命令对Redis的键进行采样,从中找到内存占用比较大的键值.
  * --eval:用于执行Lua脚本
  * --latency:
    * --latency: 测试客户端到目标Redis的网络延迟;
    * --latency-history:分时段的形式展示延迟信息
    * --latency-dist: 使用统计图表的形式从控制台输出延迟统计信息
  * --stat: 实时获取Redis的重要统计信息
  * --raw和--no-raw
    * --no-raw要求命令返回结果必须是原始的格式
    * --no-raw要求命令返回结果必须是格式化后的

#### **全局命令**

  * 查看所有键: `keys *` (O(n))
  * 键总数: `dbsize`  (O(1))
  * 检查键是否存在: `exists key`
  * 删除键: `del key [key ....]`
  * 键过期: `expire key seconds`
  * 查看键剩余过期时间: `ttl key`,返回值有三种: 
    * 大于等于0的整数: 键剩余的过期时间;
    * -1: 键没设置过期时间;
    * -2: 键不存在;
  * 键的数据结构类型: `type key`,若key不存在则会返回`none`


#### 键管理

**单个键管理**

* **键重命名**:`rename key newkey`

  * 为了防止被强行rename,Redis提供了renamenx命令,确保只有newKey不存在的时候才被覆盖.

  * 由于重命名键期间会执行del命令杀出旧的键,如果键对应的值比较大,会存在阻塞Redis的可能性.

  * 如果rename和renamenx中key和newKey相同,在Redis3.2和之前版本返回结果略有不同

    > Redis 3.2 
    >
    > rename key key
    >
    > OK
    >
    > Redis3.2之前的版本会提示错误
    >
    > rename key key 
    >
    > (error) ERR souce and destination objects are the same

* **随机返回一个键**: `randomkey`

* **键过期**

  * `expire key seconds`: 键在seconds秒后过期
  * `expireat key timestamp`: 键在秒级时间戳timestamp后过期
  * `ttl`和`pttl`都可以查询键的剩余过期时间,但`pttl`精度更高可以达到毫秒级别,有3种返回值:
    * 大于等于0的整数:键剩余的过期时间(ttl是秒,pttl是毫秒);
    * -1: 键没有设置过期时间;
    * -2:键不存在;
  * `expireat`命令可以设置键的秒级过期时间戳
  * Redis2.6版本后提供毫秒级的过期方案:
    * `pexpire key milliseconds`: 键在milliseconds毫秒后过期.
    * `pexpireat key milliseconds-timestamp`: 键在毫秒级时间戳timestamp后过期;
  * `persist`命令可以将键的过期时间清除;
  * `setex`命令作为set+expire的组合,不但是原子执行,同时减少了一次网络通讯的时间;
  * **Redis不支持二级数据结构(列如哈希,列表)内部元素的过期功能,例如不能对列表类型的一个元素做过期时间设置;**

* **迁移键**

  * `move key db`

  * `dump key`

  * `restore key ttl value`

  * `dump+restore`可以实现在不同的Redis实例之间进行数据迁移的功能,整个迁移的过程分为两步:

    * 在源Redis上,`dump`命令会将键值序列化,格式采用的是RDB格式.
    * 在目标Redis上,`restore`命令将上面序列化的值进行复原,其中ttl参数代表过期时间,如果ttl=0代表没有过期时间.
    * `dump+restore`有两点需要注意:
      * 整个迁移过程并非与原子性的,而是通过客户端分步完成的
      * 迁移过程是开启了两个客户端连接,所以dump的过程不是源Redis和目标Redis之间进行传输.

  * `migrate host port key|""  destination-db timeout [copy] [replace] [keys key [key ...]]`

    >  host: 目标Redis的IP地址
    >
    >  port: 目标Redis的端口
    >
    >  key|"": 在Redis3.0.6版本之前,migrate只支持迁移一个键,所以此处是要迁移的键,但在Redis3.0.6版本之后支持迁移多个键,多个当前要迁移多个键,此处为空字符串"".
    >
    >  destination-db: 目标Redis的数据库索引,例如要迁移到0号数据库,这里就写0.
    >
    >  timeout: 迁移的超时时间(单位为毫秒)
    >
    >  [copy] : 如果添加此选项,迁移后并不删除源键
    >
    >  [replace] : 如果添加此选项,migrate不管目标Redis是否存在该键都会正常迁移进行数据覆盖.
    >
    >  [keys  key [key...]: 迁移多个键,列如要迁移key1,key2,key3,此处填写"keys key1 key2 key3"

    * migrate命令就是将dump,restore,del三个命令进行组合,从而简化操作流程.
    * migrate命令具有原子性
    * 实现过程和dump+restore基本类似,但有3点不太相同:
      * 整个过程是原子执行的,不需要在多个Redis实例上开启客户端,只需要在源Redis上执行migrate命令即可.
      * migrate命令的数据传输直接在源Redis和目标Redis上完成的.
      * 目标Redis完成restore后会发送OK给源Redis,源Redis接收后会根据migrate对应的选项来决定是否在源Redis上删除对应的键; 

  * move,dump+restore,migrate三个命令比较

    | 命令         | 作用域        | 原子性 | 支持多个键 |
    | ------------ | ------------- | ------ | ---------- |
    | move         | Redis实例内部 | 是     | 否         |
    | dump+restore | Redis实例之间 | 否     | 否         |
    | migrate      | Redis实例之间 | 是     | 是         |

* **遍历键**

  * 全量遍历键: `key pattern`
    * *代表匹配任意字符;
    * .代表匹配一个字符;
    * []代表匹配部分字符,例如[1,3]代表匹配1,3,[1-10]代表匹配1到10的任意数字;
    * \x用来做转义,例如要匹配星号,问号需要进行转义;
  * 渐进式遍历: `scan cursor [match pattern] [count number]`
    * cursor是必需参数,实际上cursor是一个游标,第一次遍历从0开始,每次sacn遍历完都会返回当前游标的值,直到游标为0,表示遍历结束.
    * match pattern是可选参数,它的作用是做模式的匹配的,和keys的模式匹配很像.
    * count number是可选参数,它的作用是表明每次要遍历的键个数,默认值是10,此参数可以适当增大.
    * Redis从2.8版本后,提供了新的命令scan,它能有效的解决keys命令可能存在阻塞问题,每次scan命令的时间复杂度是O(1)

#### 数据库管理

* 切换数据库: `select dbIndex`
* Redis默认配置中有16个数据库;
* 清除数据库
  * 清除当前数据库:`flushdb`
  * 清除所有数据库: `flushall`
  * 若当前数据库键值数量比较多,flushdb/flushall存在阻塞Redis的可能性;

#### 慢查询

* 所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间,当超过预设阀值,就将这条命令记录下来;

* Redis客户端执行一条命令分为如下四个部分:

  * 发送命令
  * 加入命令队列
  * **命令执行(慢查询只统计该步骤的耗时)**
  * 返回结果

* 预设阀值: `slowlog-log-slower-than`(单位微妙,1秒=1000毫秒=1000000微妙),默认值是10000(10毫秒)

  * 若 `slowlog-log-slower-than`=0,则会记录所有的命令;
  * 若 `slowlog-log-slower-than`<0,则不会记录任何记录;

* 慢查询日志最多存储多少条: `slowlog-max-len`

* 在Redis中有两种修改配置的方法:

  * 修改配置文件

  * 使用`config set`命令动态修改

    ```yaml
    config set slowlog-log-slower-than 20000
    config set slowlog-max-len 1000
    config rewrite
    ```

* 获取慢查询日志:`slowlog get [n]`

  * n指定条数
  * 每个慢查询日志有4个属性组成:**慢查询日志的标志id,发生时间戳,命令耗时,执行命令和参数**

* 获取慢查询日志列表当前长度:`slowlog len`

* 慢查询日志重置:`slowlog reset`

#### **Pipeline**

* Redis客户端执行一条命令分为如下四个部分:

  1. 发送命令

  2. 加入命令队列
  3. 命令执行
  4. 返回结果

* 1~4称为Round Trip Time (RTT,往返时间)

* Redis提供的批量操作命令(mget,mset),可以有效的节约RTT,使用Pipeline执行n次命令,整个过程需要1次RTT.

* 原生批量命令和Pipeline对比

  * 原生批量命令是原子的,Pipeline是非原子的
  * 原生批量命令是一个命令对应多个key,Pipeline支持多个命令
  * 原生批量命令是Redis服务端支持实现的,而Pipeline需要服务端和客户端的共同实现.

#### 事务

* Redis提供了简单的事务功能,将一组需要一起执行的命令放到`multi`和`exec`两个命令之间.`multi`命令代表事务开始,`exec`命令代表事务结束,它们之间的命令是原子顺序执行的.
* 若出现语法错误,会造成整个事务无法执行.

#### Lua脚本

* 将Lua脚本加载到Redis内存中:`script load`
* 判断Lua脚本加载到内存中的生成的sha码是否存在:`script exists sha [sha...]`
* 清除内存中已加载的所有Lua脚本:`script flush`
* 杀死正在执行的Lua脚本:`script kill`

#### **发布订阅**

* **命令:**
  * 发布消息:`publish channel message`
  * 订阅消息:`subscribe channel [channel...]`
  * 取消订阅:`unsubscribe channel [channel]`
  * 按照模式订阅和取消订阅:
    * `psubscribe pattern [pattern...]`
    * `punsubscribe pattern [pattern...]`
  * 查看活跃的频道:`pubsub channels [pattern]`
  * 查看频道订阅数:`pubsub numsub [channel...]`
  * 查看模式订阅数:`pubsub numpat`
* **注意点:**
  * **客户端在执行订阅命令之后进入订阅状态,只能接受subscribe,psubscribe,unsubscribe,punsubscribe四个命令;**
  * **新开启的订阅客户端,无法收到该频道之前的消息,因为Redis不会对发布的消息进行持久化.**

#### 客户端通信协议

* 几乎所有的主流编程语言都有Redis的客户端

  * 客户端与服务端之间的通信协议是在TCP协议之上构建的
  * Redis指定了RESP(REdis Serialization Protocol,Redis序列化协议)实现客户端与服务端的正常交互

* **发送命令格式**

  ```
  *<参数数量> CRLF
  $<参数1的字节数量> CRLF
  <参数1> CRLF
  ...
  $<参数N的字节数量> CRLF
  <参数N> CRLF
  
  // 示例 set hello world
  *3
  $3
  SET
  $5
  hello
  $5
  world
  ```

* **返回结果格式** 

  * Redis返回结果类型分为以下五种:
  * 状态回复: 在RESP中第一个字节为"+";
    * 错误回复: 在RESP中第一个字节为"-";
    * 证书回复: 在RESP中第一个字节为": ";
  * 字符串回复: 在RESP中第一个字节为"$";
  * 多条字符串回复: 在RESP中第一个字节为"*";

#### **客户端API**

**客户端API**

* **client list**

  * `client list`命令能列出与Redis服务端相连的所有客户端连接信息

    ```shell
    127.0.0.1:6379> client list
    id=47 addr=127.0.0.1:58708 fd=8 name= age=5 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client user=default
    
    * 标识: id,addr,fd name
    id: 客户端连接的唯一标识,这个id是随着Redis的连接自增的,重启Redis后会重置为0;
    addr: 客户端连接的ip和端口;
    fd: socket的文件描述符,与lsof命令结果中的fd是同一个,如果fd=-1代表当前客户端不是外部客户端,而是Redis内部的伪装客户端.
    name: 客户端的名字,可以通过client setName和client getName两个命令来设置和获取
    
    * 输入缓冲区: qbuf,qbuf-free
    Redis为每个客户端分配了输入缓存区,它的作用是将客户端发送的命令临时保存,同事Redis从输入缓冲区拉取命令并执行,输入缓冲区为客户端发送命令到Redis执行命令提供了缓存功能.
    qbuf和qbuf-free分别代表这个缓存区的总容量和剩余容量Redis没有提供相应的配置来规定每个缓冲区的大小,输入缓冲区会根据输入内容大小的不同动态调整,只是要求每个客户端缓冲区的大小不能超过1G,c超过后客户端将被关闭.
    ```

  * 输入缓冲区使用不当会产生两个问题:

    * 一旦某个客户端的输入缓冲区超过1G,客户端将会被关闭.
    * 输入缓冲区不收maxmemory控制,假设一个Redis实例设置了maxmenmory为4G,已经存储了2G数据,但是此时输入缓冲区使用了3G,已经超过maxmemory限制,可能产生数据丢失,键值淘汰,OOM等情况

  * 监控输入缓冲区异常的方法有两种:

    * 通过定期执行client list 命令,收集qbuf和qbuf-free找到异常的连接记录并分析,最终找到可能出现问题的客户端

    * 通过info命令的info clients模块,找到最大的输入缓冲区

      | 命令         | 优点                                     | 缺点                                                         |
      | ------------ | ---------------------------------------- | ------------------------------------------------------------ |
      | client list  | 能精准分析给个客户端来定位               | 执行速度较慢(尤其在连接数较多的情况下),频繁执行存在阻塞Redis的可能 |
      | info clients | 执行速度比client list快,分析过程较为简单 | 不能精准定位到客户端;不能显示所有缓冲区的总量,只能显示最大量 |

  * 输出缓冲区: obl,oll,omem

    * Redis为每个客户端分配了输出缓冲区,它的作用是保存命令执行的结果返回给客户端,为Redis和客户端交互返回结果提供缓冲;

    * 与输入缓冲区不同的是,输出缓冲区的容量可以通过参数`client-output-buffer-limit`来进行设置,并且输出缓冲区做的更加细致,按照客户端的不同分为三种:普通客户端,发布订阅客户端,slave客户端.

    * 对应的配置规则是:`client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>`

      * <class>: 客户端类型,分为三种:

        * normal: 普通客户端
        * slave: slave客户端,用于复制
        * pubsub: 发布订阅客户端

      * <hard limit>: 如果客户端使用的输出缓冲区大于<hard limit>,客户端会被立即关闭;

      * <soft limit>和<soft seconds>: 如果客户端使用的输出缓冲区超过了<soft limit>并且持续了<soft limit>秒,客户端会被立即关闭.

      * Redis的默认配置是:

        ```
        client-output-buffer-limit normal 0 0 0
        client-output-buffer-limit slave 256mb 64mb 60
        client-output-buffer-limit pubsub 32mb 8mb 60
        ```

      * 和输入缓冲区相同是,输出缓冲区也不受到maxmemory的限制,实际上输出缓冲区由两部分组成:固定缓冲区(16 KB)和动态缓冲区,其中固定缓冲区返回比较小的执行结果,而动态缓冲区返回比较大的结果

      * client list中的obl代表固定缓冲区的长度,oll代表动态缓冲区列表的长度,omem代表使用的字节数.

  * 客户端的存活状态

    * client list中age和idle分别代表当前客户端已经连接的时间和最近一次的空闲时间

  * 客户端的限制maxclients和timeout

    * maxclients参数来限制最大客户端连接数,一旦连接数超过maxclients,新的连接将被拒绝.maxclients默认值是100000
    * config set maxclients 对最大客户端连接数动态设置

  * 客户端类型

    * client list中flag是用于标识当前客户端的类型

    | 序号 | 客户端类型 | 说明                                              |
    | ---- | ---------- | ------------------------------------------------- |
    | 1    | N          | 普通客户端                                        |
    | 2    | M          | 当前客户端是master节点                            |
    | 3    | S          | 当前客户端是slave节点                             |
    | 4    | O          | 当前客户端正在执行monitor命令                     |
    | 5    | x          | 当前客户端正在执行事务                            |
    | 6    | b          | 当前客户端正在等待阻塞事件                        |
    | 7    | i          | 当前客户端正在等待VM I/O,但此状态目前已经废弃不用 |
    | 8    | d          | 一个受监视的键已被修改,EXEC命令将失败             |
    | 9    | u          | 客户端未被阻塞                                    |
    | 10   | c          | 回复完整输出后,关闭连接                           |
    | 11   | A          | 尽可能地快速关闭连接                              |

* client list命令结果的全部属性

  | 序号 | 参数      | 含义                                                    |
  | ---- | --------- | ------------------------------------------------------- |
  | 1    | id        | 客户端连接id                                            |
  | 2    | addr      | 客户端连接IP和端口                                      |
  | 3    | fd        | socket的文件描述符                                      |
  | 4    | name      | 客户端连接名称                                          |
  | 5    | age       | 客户端连接存活时间                                      |
  | 6    | idle      | 客户端连接空闲时间                                      |
  | 7    | flags     | 客户端类型标识                                          |
  | 8    | db        | 当前客户端正在使用的数据库索引下标                      |
  | 9    | sub/psub  | 当前客户端订阅的频道或者模式数                          |
  | 10   | multi     | 当前事务中已执行命令个数                                |
  | 11   | qbuf      | 输入缓冲区总容量                                        |
  | 12   | qbuf-free | 输入缓冲区剩余容量                                      |
  | 13   | obl       | 固定缓冲区的长度                                        |
  | 14   | oll       | 动态缓冲区的长度                                        |
  | 15   | omem      | 固定缓冲区和动态缓冲区使用的容量                        |
  | 16   | events    | 文件描述符事件(r/w): r和w分别代表客户端套接字可读和可写 |
  | 17   | cmd       | 当前客户端最后一次执行的命令,不包含参数                 |

* 用于杀掉指定IP地址和端口的客户端:`client kill ip:port`

* 用于阻塞客户端timeout毫秒数,在此期间客户端连接将被阻塞: `client pause timeout(毫秒)`

  * client pause只对普通和发布订阅客户端有效,对主从复制(从节点内部伪装了一个客户端)是无效的,因此在期间主从复制是正常进行的
  * client pause可以用一种可控的方式将客户端连接从一个Redis节点切换到另一个Redis节点

* **monitor**

  * monitor命令用于监控Redis正在执行的命令;

* 客户端相关配置

  * timeout: 检测客户端空闲连接的超时时间,一旦idle时间达到了timeout,客户端将会被关闭,如果设置为0则不会进行检测;
  * maxclients: 客户端最大连接数
  * tcp-keepalive: 检测TCP连接活性的周期,默认值为0,也就是并进行检测,若要设置,则可以设置为60,那么Redis会每隔60秒对它创建的TCP连接进行活性检测,防止大量死链接占用系统资源.
  * tcp-backlog: TCP三次握手后,会将接受的连接放入队列中,tcp-backlog就是队列的大小,它在Redis中Redis中的默认值是511.通常来讲这个参数不需要调整,但是这个参数会受到操作系统的影响.
