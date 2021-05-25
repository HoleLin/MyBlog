---
title: Redis数据结构以及使用场景
date: 2021-05-24 23:33:33
tags: 
- Redis
- 数据结构
categories: Redis
---
### Redis数据结构以及使用场景

#### **数据结构和内部编码**

* **对外的数据结构**

  * **string(字符串)**
  * **hash(哈希)**
  * **list(列表)**
  * **set(集合)**
  * **zset(有序集合)**
* 查看内部编码:**`object encoding key`

  | 数据结构 | 内部编码   |
  | -------- | ---------- |
  |          | raw        |
  | string   | int        |
  |          | embstr     |
  | hash     | hashtable  |
  |          | ziplist    |
  | list     | linkedlist |
  |          | ziplist    |
  |          | quicklist  |
  | set      | hashtable  |
  |          | intset     |
  | zset     | skiplist   |
  |          | ziplist    |

#### **单线程架构**

* Redis使用了单线程架构和I/O多路复用模型来实现高性能的内存数据库服务.Redis客户端与服务端的模型,每次客户端调用都经历了**发送命令,执行命令,返回结果**三个过程.
* 因为Redis是单线程来处理命令的,所以一条命令从客户端达到服务端不会立刻被执行,所有命令都会进入到一个队列中,然后逐个被执行.
* **单线程为什么还能这么快?**
  * 纯内存访问,Redis将所有数据放在内存中,内存的响应时长大约为100纳秒;
  * 非阻塞I/O,Redis使用epoll作为I/O多路复用技术的实现,在加上Redis自身的时间处理模型将epoll中的连接,读写,关闭都转换为事件,不在网络I/O上浪费过多的时间;
  * 单线程避免了线程切换和竞态产生的消耗;

#### 字符串

* 字符串类型的值可以是字符串(简单的字符串,复杂的字符串(JSON,XML)),数字(整数,浮点数),甚至为二进制(图片,音频,视频),但是值最大不能超过512MB.

* **常用命令**

  * **设置值**:`set key value [ex seconds] [px milliseconds] [nx|xx]`

    * ex seconds: 为键设置秒级过期时间;
    * px milliseconds: 为键设置毫秒级过期时间;
    * nx: 键必须不存在,才可以设置成功,用于添加.
    * xx: 与nx相反,键必须存在,才可以设置成功,用于更新.

  * **setex,setnx,setxx**

    * `setex key seconds value`
    * `setnx key value`
    * `setxx key value`
    * `setnx`和`setxx`使用场景: 如果有多个客户端同事执行`setnx key value`,根据`setnx`的特性只有一个客户端能设置成功,`setnx`可以作为分布式锁的一种实现方案.Redis官方给出了使用`setnx`实现分布式锁的方法:https://redis.io/topics/distlock

  * **获取值**: `get key`

    * 获取的键不存在时,则返回nill(空)

  * **批量设置值**: `mset key value [key value....]`

  * **批量获取值**: `mget key [key...]`

  * 批量操作可以有效的提高开发效率

    * 不使用mget命令: **n 次get时间 = n次网络时间 + n次命令时间**
    * 使用mget命令: **n次get时间 = 1次网络时间 + n次命令时间**

  * **计数**: 

    * **自增**:`incr key`
    * **自减**:`decr key`
    * **自增指定数字**: `incrby key increment`
    * **自减指定数字**: `decrby key decrment`
    * **自增浮点数**: `incrbyfloat key incremet`
    * incr命令用于对值做自增操作,返回结果分为三种情况:
      * 值不是整数,返回错误;
      * 值是帧数,返回自增后的结果;
      * 键不存在,按照值为0自增,返回结果为1;

  * **追加值**: `append key value` 向字符串尾部追加值

  * **字符串长度**:`strlen key`

  * **设置并返回原值**:`getset key value`

  * **设置指定位置的字符**: `setrange key offset value`

  * **获取部分字符串**: `getrange key start end`

    * start和end分别是开始和结束的偏移量,偏移量从0开始计算

  * **字符串类型命令时间复杂度**

    | 命令                           | 时间复杂度                                                   |
    | ------------------------------ | ------------------------------------------------------------ |
    | set key value                  | O(1)                                                         |
    | get key                        | O(1)                                                         |
    | del key [key ...]              | O(k) k是键的个数                                             |
    | mset key value [key value ...] | O(k) k是键的个数                                             |
    | mget key [key ...]             | O(k) k是键的个数                                             |
    | incr key                       | O(1)                                                         |
    | decr key                       | O(1)                                                         |
    | incrby key increment           | O(1)                                                         |
    | decrby key decrment            | O(1)                                                         |
    | incrbyfloat key increment      | O(1)                                                         |
    | append key value               | O(1)                                                         |
    | strlen key                     | O(1)                                                         |
    | setrange key offset value      | O(1)                                                         |
    | getrange key start end         | O(n) n为字符串长度,由于获取字符串非常快,若字符串不是很长,可视为O(1) |

  * **内存编码**

    * 字符串类型内部编码有3种:
      * int: 8个字节的长整型;
      * embstr: 小于等于39个字节的字符串;
      * raw: 大于39个字节的字符串;
    * Redis会更根据当前值的类型和长度决定使用哪种内部编码实现.

* **典型使用场景**

  * 缓存功能

  * 计数

  * 共享Session

  * 限速

    * 限制用户每分钟获取验证码的频率,如一分钟不能超过5次
    * 网站限制一个IP地址不能一秒钟之内访问超过n次

    ```
    伪代码:
    phoneNum = "132xxxxxx";
    key = "shortMsg:limit:" + phoneNum;
    // SET key value EX 60 NX
    isExists = redis.set(key,1,"EX 60","NX");
    if(isExists != null || redis.incr(key) <=5 ){
    	// 通过
    }else{
    	// 限速
    }
    ```

#### 哈希

* 键值对 value= {{field1,value},....{fieldN,valueN}}

* **命令**

  * **设置值:** `hset key field value`
  * **获取值**: `hget key field`
    * 若键或者field不存在,则会返回nil
  * **删除field**: `hdel key field [field ...]`
    * hdel会删除一个或多个field,返回结果为成功删除的field个数
  * **计算field个数**: `hlen key`
  * **批量设置或获取field-value**:
    * `hmget key field [field ....]`
    * `hmset key field value [field value ...]`
  * **判断field是否存在**: `hexists key field`
  * **获取所有field**: `hkeys key`
  * **获取所有的value**: `hvals key`
  * **获取所有的field-value**: `hgetall key`
    * 使用过hgetall的时候,若哈希元素个数比较多,会存在阻塞Redis的可能,若一定要获取全部field-value,可以使用`hscan`命令,该命令会渐进式遍历哈希类型.
  * **hincrby hincrbyfloat** 
    * `hincrby key field`
    * `hincrbyfloat key field`
  * **计算value的字符串长度(Redis3.2以上)**:`hstrlen key field`

* **哈希类型命令的时间复杂度**

  | 命令                                    | 时间复杂度        |
  | --------------------------------------- | ----------------- |
  | hset key field value                    | O(1)              |
  | hget key field                          | O(1)              |
  | hdel key field [field ...]              | O(k),k是field个数 |
  | hlen key                                | O(1)              |
  | hgetall key                             | O(n),n是field总数 |
  | hgmet key field [field ...]             | O(k),k是field个数 |
  | hmset key field value [field value ...] | O(k),k是field个数 |
  | hexists key field                       | O(1)              |
  | hkeys key                               | O(n),n是field总数 |
  | hvals key                               | O(n),n是field总数 |
  | hsetnx key field value                  | O(1)              |
  | hincrby key field increment             | O(1)              |
  | hincrbyfloat key field increment        | O(1)              |
  | hstrlen key field                       | O(1)              |

* **内部编码**

  * 哈希类型的内部编码有两种:
    * **zplist(压缩列表)**: 当**哈希类型元素个数小于hash-max-ziplist-entries配置(默认512个)**,**同时所有值都小于hash-max-ziplist-value配置(默认64字节)时,Redis会使用ziplist作为哈希的内部实现**,ziplist使用更加紧凑的结构实现多个元素的连续存储,所有在节省内存方面比hashtable更加优秀.
    * **hashtable(哈希表)**: 当哈希类型无法满足ziplist的条件时,Redis会使用hashtable作为哈希的内部实现,因此此时ziplist的读写效率会下降,而hashtable的读写时间复杂度为O(1).

#### 列表

* **描述**

  > 列表(list)类型是用来存储多个有序的字符串.列表中的每个字符串称为元素(element),一个列表最多可以存储2<sup>32</sup>-1个元素.
  >
  > Redis中,可以对列表两端插入(push)和弹出(pop),还可以指定范围的元素列表,获取指定索引下标的元素

* **特点**: 

  * 列表元素是有序的;
  * 列表元素是可以重复的;

* **命令**

  * **列表的四种操作类型**

    * **添加**: rpush lpush linsert
    * **查**: lrange lindex llen
    * **删除**: lpop rpop lrem ltrim
    * **修改**: lset
    * **阻塞操作**: blpop brpop

  * **添加操作**

    * 从右边插入元素: `rpush key value [value ...]` 
    * 从左边插入元素: `lpush key value [value ...]` 
    * 向某个元素前或者后插入元素:`linsert key before|after pivot value`
      * linsert命令会从列表中找到等于pivot的元素,在其前(before)或者后(after)插入一个新的元素value

  * **查找操作**

    * 获取指定范围内的元素列表:`lrange key strat end`

      * 索引下标有两个特点:

        > * 索引下标从左到右分别是0到N-1,但是从右到左分别是-1到N
        > * lrange中end选项包含了自身,**左闭由闭**

    * 获取列表指定索引下标元素: `lindex key index`

    * 获取列表长度: `llen key`

  * **删除**

    * 从列表左侧弹出元素: `lpop key`
    * 从列表右侧弹出元素: `rpop key`
    * 删除指定元素: `lrem key count value`
      * lrem会从列表中找到等于value的元素进行删除,根据count不同分为三种情况:
        * count>0,从左到右,删除最多count个元素
        * count<0,从右到左,删除最多count绝对值个元素
        * count=0,删除所有
    * 按照索引范围修剪列表:`ltrim key start end`

  * **修改**

    * 修改指定索引下标的元素:`lset key index newValue`

  * **阻塞操作**

    * 阻塞式弹出:

      * `blpop key [key...] timeout `
      * `brpop key [key...] timeout`
      * key [key...],多个列表的键
      * timeout: 阻塞时间(单位秒)

    * 列表为空:  若timeout=3,那么客户端要等到3秒后返回,若timeout=0,那客户端会一直阻塞等下去;

    * 列表不为空: 客户端会立即返回 

    * 使用brpop时,有两点注意:

      * 若是多个建,那么brpop会从左只有遍历键,一旦有一个键能弹出元素,客户端立即返回;
      * 若多个客户端对同一个键执行brpop,那么最新执行brpop命令的客户端可以获取弹出的值;

    * **列表命令时间复杂度**

      | 命令                                  | 时间复杂度                                |
      | ------------------------------------- | ----------------------------------------- |
      | rpush key value [value...]            | O(k) ,k是元素个数                         |
      | lpush key value [value...]            | O(k) ,k是元素个数                         |
      | linsert key before\|after pivot value | O(n),n是pivot距离列表头或为的距离         |
      | lrange key start end                  | O(s+n),s是start偏移量,n是start到end的范围 |
      | lindex key index                      | O(n),n是索引的偏移量                      |
      | llen key                              | O(1)                                      |
      | lpop key                              | O(1)                                      |
      | rpop key                              | O(1)                                      |
      | lrem count value                      | O(n),n是列表长度                          |
      | ltrim key start end                   | O(n),n是要剪裁的元素总数                  |
      | lset key index value                  | O(n),n是索引的偏移量                      |
      | blpop brpop                           | O(1)                                      |

* **内部编码**

  * 列表类型的内部编码有两种:
    * **ziplist(压缩列表)**: **当列表的元素个数小于list-max-ziplist-entries配置(默认512个)**,**同时列表中每个元素的值都小于list-max-ziplist-value配置时(默认64字节)**.
    * **linkedlist(链表)**: 当列表类型无法满足ziplist的条件时,Redis会使用linkedlist作为列表的内部实现;

* **使用口诀:**

  * lpush+lpop=Stact(栈)
  * lpush+rpop=Queue(队列)
  * lpush+ltrim=Capped Collection(有限集合)
  * lpush+brpop=Message Queue(消息队列)

#### 集合

* **理论**

  > * 集合(set)类型可以用来保存多个字符串元素,但是集合中不允许有重复元素,且集合的元素是无序的,不能通过索引下标获取元素.
  > * 一个集合最多可以存储2<sup>32</sup>-1个元素.
  > * Redis除了支持集合内的增删改查,同事还支持多个集合去交集,并集,差集.

* **命令**

  * 添加元素: `sadd key element [element...]`,返回添加成功的元素个数;
  * 删除元素: `srem key element [element...]`,返回添加删除的元素个数;
  * 计算元素个数:`scard key`
  * 判断元素是否在集合中: `sismenber key element`
    * 在,返回1
    * 不在,返回0
  * 随机从集合返回指定个数元素:`srandmember key [count]`
  * 随机从集合中弹出元素: `spop key`,会从集合中删除被弹出的元素
  * 获取所有元素:`smembers key`
  * 集合间操作
    * 求多个集合的**交集**: `sinter key [key...]`
    * 求多个集合的**并集**: `sunion key [key...]`
    * 求多个集合的**差集**: `sdiff key [key...]`
    * 将交集,并集,差集的结果保存
      * `sinterstore destination key [key...]`
      * `sunionstore destination key [key...]`
      * `sdiffstore destination key [key...]`

* **集合常用命令时间复杂度**

  | 命令                           | 时间复杂度                                     |
  | ------------------------------ | ---------------------------------------------- |
  | sadd key element [element ...] | O(k),k是元素个数                               |
  | srem key element [element...]  | O(k),k是元素个数                               |
  | scard key                      | O(1)                                           |
  | sismember key element          | O(1)                                           |
  | srandmember key [count]        | O(count)                                       |
  | spop key                       | O(1)                                           |
  | smembers key                   | O(n),n是元素总数                               |
  | sinter key [key...]            | O(n*k),k是多个集合中元素最少的个数,m是键的个数 |
  | sunion key [key...]            | O(k),k是多个集合元素个数和                     |
  | sdiff key [key...]             | O(k),k是多个集合元素个数和                     |

* **内部编码**

  * **intset(整数集合)**: 当集合中的元素都是整数且元素个数小于set-maxinset-entries配置(默认512个)时,Redis会选用intset来作为集合的内部实现,从而减少内存的使用.
  * **hashtable(哈希表)**: 当集合类型无法满足intset的条件是,Redis会使用hashtable作为集合的内部实现.

* **使用场景**

  * 标签(tag): 用户的兴趣爱好
  * sadd=Tagging(标签)
  * spop/srandmember=Random item(生成随机数,比如抽奖)
  * sadd+sinter=Social Graph(社交需求)

#### 有序集合

* **理论**

  > * 集合(无重复元素);
  > * 可以排序;
  > * 有序集合提供了获取指定分数和元素范围查询,计算成员排名等功能.
  > * 有序集合中元素不能重复,但是score可以重复;

* **列表,集合,有序集合三者的异同点**

  | 数据结构 | 是否允许重复元素 | 是否有序 | 有序实现方式 | 应用场景          |
  | -------- | ---------------- | -------- | ------------ | ----------------- |
  | 列表     | 是               | 是       | 索引下标     | 时间轴,消息队列等 |
  | 集合     | 否               | 否       | 无           | 标签,社交等       |
  | 有序集合 | 否               | 是       | 分值         | 排行榜系统,社交   |

* **命令**

  * **添加成员**:`zadd key socre member [score member ...]`

    * Redis3.2为zadd命令添加了nx,xx,ch,incr四个选项:
      * nx:member必须不存在,才可以设置成功,用于添加;
      * xx:member必须存在,才可以设置成功,用于更新;
      * ch:返回此次操作后,有序集合元素和分数变化的个数;
      * incr: 对socre做增加,相当于zincrby;
    * 有序集合相比集合提供了排序字段,但也产生了代价,zadd的时间复杂度为O(log(n)),sadd的时间复杂度为O(1);

  * 计算成员个数:`zcard key`

  * **计算某个成员的分数**:`zscore key member`

  * **计算成员的排名**

    * 分数从低到高排名:`zrank key member`
    * 分数从高到低排名:`zrevrank key member`

  * **删除成员**:`zrem key member [member...]`

  * **增加成员的分数:** `zincrby key increment member`

  * **返回指定排名范围的成员**

    * 按照分值排名,从低到高:`zrange key start end [withscores]`
    * 按照分值排名,从高到低:`zrevrange key start end [withscores]`

  * **返回指定分数范围的成员**

    * 按照分数从低到高:`zrangebyscore key min max [withscores] [limit offset count]`
    * 按照分数从高到低`zrevrangebyscore key max min [withscores] [limit offset count]`
    * withscores选项会同时返回每个成员的分数;
    * limit offset count 选项可以限制输出的起始位置和个数;
    * min和max还支持开区间(小括号)和闭区间(中括号),-inf和+inf分别代表无限小和无限大
      * `zrangebysocre user:ranking (200 +inf withsocres`

  * **返回指定分数范围成员个数**:`zcount key min max`

  * **删除指定排名内升序元素**:`zremrangebyrank key start end`

  * **删除指定分数范围的成员**:`zremrangebyscore key min max`

  * **集合间操作**

    * **交集**: `zinterscore destination numkeys key [key...] [weights weight [weight ...]] [aggregate sum|min|max]`
      * destination:交集计算结果保存到的键的名称
      * numkeys: 需要做交集计算键的个数;
      * key [key ...]: 需要做交集计算的键
      * weight weight[weight...]: 每个键的权重,在做交集计算时,每个键中的每个member会将自己分数乘以这个权重,每个键的权重默认为1;
      * aggregate sum|min|max: 计算成员交集后,分值可以按照sum(和),min(最小值),max(最大值)做汇总,默认值为sum
    * **并集**:`zunionscore destination numkeys key [key...] [weights weight [weight ...]] [aggregate sum|min|max]`

  * 有序集合命令的时间复杂度

    | 命令                                          | 时间复杂度                                                   |
    | --------------------------------------------- | ------------------------------------------------------------ |
    | zadd key score member [score member...]       | O(k*log(n)),k是添加成员的个数,n是当前有序集合成员个数        |
    | zcard key                                     | O(1)                                                         |
    | zscore key member                             | O(1)                                                         |
    | zrank key member                              | O(log(n)),n是当前有序集合成员个数                            |
    | zrevrank key member                           | O(log(n)),n是当前有序集合成员个数                            |
    | zrem key member [member...]                   | O(k*log(n)),k是删除成员的个数,n是当前有序集合成员个数        |
    | zincrby key increment member                  | O(log(n)),n是当前有序集合成员个数                            |
    | zrange key start end [withscores]             | O(k+log(n)),k是获取成员的个数,n是当前有序集合成员个数        |
    | zrevrange key start end [withscores]          | O(k+log(n)),k是获取成员的个数,n是当前有序集合成员个数        |
    | zrangebyscore key min max [withscores]        | O(k+log(n)),k是获取成员的个数,n是当前有序集合成员个数        |
    | zrebvrangebyscore key min max [withscores]    | O(k+log(n)),k是获取成员的个数,n是当前有序集合成员个数        |
    | zcount                                        | O(log(n)),n是当前有序集合成员个数                            |
    | zremrangebyrank key start end                 | O(k+log(n)),k是删除成员的个数,n是当前有序集合成员个数        |
    | zremrangebyscore key min max                  | O(k+log(n)),k是删除成员的个数,n是当前有序集合成员个数        |
    | zinterscore destination numkeys key [key ...] | O(n*k)+O(m*log(m)),n是成员数最小的有序集合成员个数,k是有序集合的个数,k是有序集合的个数,m是结果集中成员个数. |
    | zunionscore destination numkeys key [key ...] | O(n)+O(m*log(m)),n是所有有序集合成员个数和,m是结果集中成员个数. |

* **内部编码**

  * **ziplist(压缩列表)**: 当有序集合的元素个数小于zet-max-ziplist-entries配置(默认128个),同事每个元素的值都小于zset-max-ziplist-value配置(默认64字节)时,Reids会用ziplist作为有序集合的内部实现,ziplist可以有效减少内存的使用.
  * **skiplist(跳表)**: 当ziplist条件不满足时,有序集合会使用skiplist作为内部实现,因为此时ziplist的读写效率会下降.

* **使用场景**

  * 排行榜系统

#### Bitmaps

* Bitmaps可以想象成一个以位为单位的数组,数组的每个单元只能存储0或1,数组的下标在Bitmaps中叫做偏移量;

* **命令:**

  * 设置值:`setbit key offset value`
  * 获取值:`getbit key offset`
  * 获取Bitmaps指定范围值为1的个数: `bitcount [stard] [end]`
  * Bitmaps间的运算:`bitop op destkey key [key ...]`,bitop是一个复合操作,它可以做多个Bitmaps的支持and(交集),or(并集),not(非),xor(异或)操作并将结果保存到destkey中
  * 计算Bitmaps中第一个值为targetBit的偏移量:`bitpos key target [start][end]`

#### **HyperLogLog**

* **命令**
* 添加:`padd key element [element...]`
  * 计算独立用户数: `pfcount key [key...]`
* 合并: `pfmerge destkey sourcekey [sourcekey...]`
* HyperLogLog内存占用量非常小,但是存在错误率,故使用时要注意两点:
  * 只为了计算独立总数,不需要获取单条数据;
  * 可以容忍一定错误率