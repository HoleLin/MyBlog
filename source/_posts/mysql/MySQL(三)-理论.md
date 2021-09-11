---
title: MySQL(三)-理论
mermaid: false
date: 2021-06-16 18:31:42
cover: /img/cover/MySQL.jpg
tags:
- 理论
categories:
- MySQL
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [MySQL 数据库面试题（2021最新版）](https://mp.weixin.qq.com/s/8uMUu5vqPShVv4LatCD5qg)
* 极客时间--MySQL实战45讲

### **数据库三大范式**

- 第一范式：每个列都不可以再拆分。
- 第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。
- 第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

### MySQL查询语句的执行流程

<img src="https://www.chenjunlin.vip/img/mysql/MySQL%E5%9F%BA%E6%9C%AC%E6%9E%B6%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.png" alt="img" style="zoom:67%;" />

* 大体来说,MySQL可分为Server层和存储引擎层两部分
  * Server层包括连接器,查询缓存,分析器,优化器,执行器等,涵盖MySQL的大多数核心服务功能,以及所有的内置函数(如日期,时间,数字和加密函数等)
  * 而在存储引擎层负责的数据存储和提取.其架构模式是插件式的,支持InnoDB,MyISAM,Memory等多个存储引擎.现在最常用的存储引擎是InnoDB,它是从MySQL5.5.5版本看是称为默认存储引擎.

##### 连接器

* 连接器负责跟客户端建立连接,获取权,维持和管理连接.连接命令一般为`mysql -h$ip -P$port -u$user -p`

  * 连接命令中的mysql是客户端工具,用来跟服务端建立连接.在完成经典的TCP握手后,连接器开始验证身份,此时用的就是输入的用户名和密码

    * 若用户名或密码不对,你就会收到一个"Access denied for user"的错误,然后客户端程序结束执行.
    * 若用户名密码通过认证,连接器会到权限表里面查询出当前登录的账号拥有的权限,之后,这个连接里面的权限判断逻辑,都将依赖于此时读取到的权限.
      * 这就意味着,一个用户的成功创建连接后,即使用管理员账号对这个用户的权限做了修改,也不会影响已经存在连接的权限.修改完成后,只有再新建的连接才会使用新的权限设置.

  * 连接完成后,如果没有后续的动作,这个连接就处于空闲状态(Command列显示为Sleep),可以通过`show processlist`命令中看到

    ```mysql
    mysql> show processlist;
    +------+-----------------+-----------+------+---------+---------+------------------------+------------------+
    | Id   | User            | Host      | db   | Command | Time    | State                  | Info             |
    +------+-----------------+-----------+------+---------+---------+------------------------+------------------+
    |    5 | event_scheduler | localhost | NULL | Daemon  | 1905442 | Waiting on empty queue | NULL             |
    | 2042 | root            | localhost | NULL | Query   |       0 | init                   | show processlist |
    | 2043 | holelin         | localhost | NULL | Sleep   |       4 |                        | NULL             |
    +------+-----------------+-----------+------+---------+---------+------------------------+------------------+
    3 rows in set (0.00 sec)
    ```

    * 若客户端太长时间没动静,连接器就会自动断开.这个时间由参数`wait_timeout`控制的,默认值是8小时;

      ```mysql
      mysql> show variables like 'wait_timeout';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | wait_timeout  | 28800 |
      +---------------+-------+
      1 row in set (0.02 sec)
      ```

    * 若在连接被断开之后,客户端再次发送请求的话,就会收到一个错误提示:`Lost connetction to MySQL server during query`,若要继续,则需要重连,重新执行请求.

* 数据库里面,**长连接**是指连接成功后,如果客户端持续有请求,则一直使用同一个连接.**短连接**则是指每次执行完很少的几次查询后就断开连接,下次查询再重新建立一个.

  * 建立连接的过程通常是比较复杂的,所以在使用中尽量减少建立连接的动作,也尽量使用长连接.但是若全部使用长连接会导致MySQL占用内存涨得特别快,因为在MySQL在执行过程中临时使用的内存是管理在连接对象里面的.这些资源会在连接断开的时候才释放.故而长连接积累下来,可能会导致内存占用太大,被系统强制杀掉(OOM),从现象来看就是MySQL异常重启了.

  * 解决方法:

    * 定期断开长连接.使用一段时间,或者程序里面判断执行过一个占用内存的大查询后,断开连接,之后要查询再重连.

    * 若使用MySQL5.7或更新版本,可以在每次执行一个比较大的操作后,通过执行`mysql_reset_connection`[C的函数]来重新初始化连接资源.

      * 这个过程不需要重连和重新做权限验证,但是会将连接恢复到刚刚创建完时的状态.

      > **mysql_reset_connection()影响以下与会话状态相关的信息：**
      >
      > * 回滚活跃事务并重新设置自动提交模式
      > * 释放所有的表锁
      > * 关闭或删除所有的临时表
      > * 重新初始化会话的系统变量值
      > * 丢失用户定义的变量设置
      > * 释放prepared语句
      > * 关闭handler变量
      > * 将last_insert_id()的值设置为0
      > * 释放get_lock()获取的锁
      > * 清空通过mysql_bind_param()调用定义的当前查询属性

##### 查询缓存

* 建立完连接后,就可以执行SQL语句了,进入第二步:查询缓存
* MySQL拿到一个查询请求后,会先到查询缓存中看看,之前是不是执行过这条语句.之前执行过的语句以及其结果可能会以`key-value`对的形式,被直接缓存在内存中.key是查询语句,value是查询结果.若能查询到则直接返回给客户端.
* **MySQL8.0将查询缓存功能移除了.**

##### 分析器

* 若没有命中查询缓存,就要真正执行语句了.首先MySQL需要知道你要做什么,因此需要对SQL语句做解析.
* 分析器先会做"词法分析",将"SELECT"等关键字识别出来,然后进行"语法分析",根据词法分析的结果,语法分析器会根据语法规则,判断输入的SQL语句是否满足MySQL语法.
  * 若语法不对,就会收到`You have an error in your SQL syntax;`的错误提示.一般关注`use near`后面的内容.

##### 优化器

* 优化器是在表里面有多个索引,决定使用哪个索引;或者在一个语句有多表关联(join)的时候,决定各个表的连接顺序.
* 优化器选择索引的目的,是找到一个最有的执行方案,并用最小的代价去执行语句.在数据库里面,扫描行数是影响执行代价的因素之一.扫描的行数越少,意味着访问磁盘数据越少,消耗的CPU资源越少.
  * 当然,扫描行数并不是唯一的判断标准,优化器还会结合是否使用临时表,是否排序等因素进行综合判断.

###### 优化器扫描行数是怎么判断的

* MySQL在真正开始执行语句之前,并不能精确的知道满足这个条件的记录有多少条,而只能根据统计信息来估算记录数.
* 这个统计信息就是索引的"区分度".显然,一个索引上不同的值越多,这个索引的区分度就越好.而一个索引上不同的值的个数,被称为"基数"(cardinakity),也就是说,这个基数越大,索引的区分度越好.
* 可以使用`show index from table_name`,看到一个表中索引的基数.

###### MySQL是怎样得到索引的基础的呢?

* MySQL采用采样统计的方法.因为把整张表取出来一行一行的统计,虽然可以得到精确的结果,但是代价太高了,所以只能选择采样统计.
  * 采样统计的时候,InnoDB默认会选择N个数据页,统计这些页面上的不同值,得到一个平均值,然后乘以这个索引的页面数,就得到这个索引的基数.
  * 而数据表会持续更新的,索引统计信息也不会固定不变.所以当变更的数据行数超过`1/M`的时候,就会自动触发重新做一次索引统计.
* 在MySQL中,有两种存储索引统计的方式,可以通过设置参数`innodb_stats_persistent`的值来选择
  * 设置为on的时候,表示统计信息会持久化存储,这时,默认的N是20,M是10.
  * 设置为off的时候,表示统计信息只会存储在内存中.这时,默认的N是8,M是16.

##### 执行器

* 开始执行的时候,需要先判断一下当前连接是否拥有对该表的执行查询权限,若没有则会返回没有权限的错误;若有权限就打开表继续执行.打开表的时候,执行器就会根据表的引擎定义,去使用这个引擎提供的接口.
* 执行器的执行流程
  * 调用InnoDB引擎接口获取这个表的第一行,若满足条件则将该行存入结果集中;不满足则跳过
  * 调用引擎接口获取"下一行",重复相同的判断逻辑,直到取到这个表的最后一行.
  * 执行器将上述遍历过程中满足条件的行组成记录集作为结果集返回客户端.
* 数据库的慢查询日志中有一个`rows_examined`的字段,表示这个语句执行过程中扫描了多少行.这个值就是执行器每次调用引擎获取数据行的时候累加的.
  * 在有些场景下,执行器调用一次,在引擎内部则扫描了多行,因此引擎扫描行数跟`rows_examined`并不是完全相同的.
* 对于由于索引信息不准确导致的问题,可以使用`analyze table`来解决.
* 对于其他优化器误判的情况,可以在应用端用`force index`来强行指定索引,也可以通过修改语句来引导优化器,还可以通过增加或者删除索引来绕过这个问题,

### MySQL更新语句的执行流程

* 更新流程主要涉及两个重要的日志模块
  * redo log(重做日志)
  * binlog(归档日志)

#### `redo log`

* `WAL(Write-Ahead Logging)`,它的关键点就是**先写日志,再写磁盘**.

  * 当有一条记录需要更新的时候,InnoDB引擎就会**先把记录写到redo log里面,并更新内存,这个时候更新算完成了**.同时,InnoDB引擎会在合适的时候,将这个操作记录更新到磁盘里面,而这个更新往往是在系统比较空闲的时候做.

* InnoDB的`redo log`是固定大小的,比如可以配置一组4个文件,每个文件的大小为1G,那这个日志总共可以记录4GB的操作.从头开始写,写到末尾就又回到开头循环写.

  <img src="https://www.holelin.cn/img/mysql/redolog%E7%BB%93%E6%9E%84.png" alt="img" style="zoom:50%;" />

  * `write pos`是当前记录的位置,一边写一边后移,写到3号文件末尾就回到0号文件开头.
  * `checkpoint`是当前要擦除的位置,也就是往后推移并且循环的,擦除记录要把记录更新到数据文件.
  * `write pos`和`checkpoint`之间的还有空着的部分,可以用用来记录新的操作.
  * 如果`write pos`追上`checkpoint`,表示`redo log`满了,这时不能再执行新的更新,得停下来先擦掉一些记录,把`checkpoint`推进一下.

* 有了`redo log`,InnoDB就可以保证即使数据库发送异常重启,之前提交的记录都不会丢失,这个能力称为**`crash-safe`**.

* `redo log`用于保证`crash-safe`能力.`innodb_flush_log_at_trx_commit`这个参数设置成1的时候,表示每次事务的`redo log`都持久化到磁盘.这个参数建议设置成1,这样可以保证MySQL异常重启之后数据不丢失.

* **`redo log`不是记录数据页"更新之后的状态",而是记录这个页"做了什么改动".**

#### `biglog`

* MySQL整体来看就两块:一块是Server层,它主要做的是MySQL功能层面的事情;还有一块是引擎层面,负责存储相关具体事宜;
* `redo log`是InnoDB引擎特有的日志,而Server层的日志称为`binlog`(归档日志);
* `sync_binlog`这个参数设置为1的时候,表示每次事务的`binlog`,表示每次事务的`binlog`都持久化磁盘.这个参数建议设置为1,这样可以保证MySQL异常重启之后`binlog`不丢失.
* `binlog`有两种模式`statement`格式是记录SQL语句,`row`格式会记录行的内容,记录两条,更新前和更新后都有.
  * statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
  * row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
  * mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

#### `redo log`和`binlog`的区别

* `redo log`是InnoDB引擎特有的;`binlog`是MySQL的Server层实现的,所有引擎都可以使用.
* **`redo log`是物理日志**,记录的是"在某个数据页上做了什么修改";**`binlog`是逻辑日志**,记录的是这个语句的原始逻辑,比如"给ID=2这一行的c字段加1".
* `redo log`是循环写的,空间固定会用完;`binlog`是可以追加写入."追加写"是指`binlog`文件写到一定大小后会切换到下一个,并不会覆盖以前的日志.

#### `update`语句的执行流程

```mysql
update T set c=c+1 where ID=2;
```

* 执行器先找到引擎ID=2这一行.ID主键,引擎直接用树搜索找到这一行.如果ID=2这一行所在的数据页本来就在内存中,就直接返回给执行器;否则需要先从磁盘读如内存,然后再返回;
* 执行器拿到引擎给的行数据,把这个值加上1,比如原来是N,现在就是N+1,得到新的一行数据,再调用引擎接口写入这行新数据;
* 引擎将这行新数据更新到内存中,同时将这个更新操作记录到`redo log`里面,此时`redo log`处于`prepare`状态.然后告知执行器完成了,随时可以提交事务;
* 执行器生成这个操作的`binlog`,并把`binlog`写入磁盘;
* 执行器调用引擎的提交事务接口,引擎把刚刚写入的`redo log`改成提交(commit)状态,更新完成;

<img src="https://www.holelin.cn/img/mysql/update%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png" alt="img" style="zoom:67%;" />

#### 两阶段提交

* 为什么必须有两阶段提交呢?这是为了让两份日志之间的逻辑一致.
* 如果不使用两阶段提交,假设当前ID=2的行,字段c的值是0,再假设执行update语句过程中在写完第一个日志后,第二个日志还没有写完期间发生了crash,会出现什么情况?
  * **先写redo log后binlog**.假设`redo log`写完,`binlog`还没有写完的时候,MySQL进程异常重启.`redo log`写完之后,系统即使崩溃,任然能够把数据恢复回来,所以恢复这一行c的值是1.但是由于`binlog`没写完就crash了,这时候`binlog`里面没有记录这个语句.因此,之后备份日志的时候,存起来的`binlog`里面就没有这条语句.如果需要用这个binlog来恢复临时库的话,由于这个语句的`binlog`丢失,这个临时库就会少了这一次更新,恢复出来的这一行c的值就是0,与原库的值不同.
  * **先写binlog后写redo log**.若`binlog`写完之后crash,由于`redo log`还没写,崩溃恢复以后这个事务无效,所以这一行c的值是0.但是binlog里面已经记录了"把c从0改成1"这个日志.所以,在之后用`binlog`来恢复的时候就多了一个事务出来,恢复出来的这一行c的值就是1,与原库的值不同.
* `redo log`和`binlog`都可以用于表示事务的提交状态,而两阶段提交就是让这个两个状态保持逻辑上的一致.

### SQL如何执行

#### Oracle

```
SQL语句-->
语法检查-->
语义检查-->
权限检查-->
共享池检查
	-->硬解析-->优化器-->执行
	-->软解析-->执行
```

#### MySQL

```
SQL语句-->
缓存查询
-->没找到
  解析器-->
  优化器-->
  执行器-->
  输出
-->找到
输出

```

* 查看一条SQL语句到执行时间分析

  ```mysql
  -- 查看是否开启，开启后可以让MySQL收集在SQL执行时所使用的资源情况
  SELECT @@profiling;
  -- 0代表关闭，1代表打开 
  SET profiling=1;
  
  show profile;
  
  show profile for query 2;
  ```

  ![img](https://www.chenjunlin.vip/img/mysql/show_profile.png)



### **MySQL有关权限的表**

>  MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由`mysql_install_db`脚本初始化。这些权限表分别`user，db，table_priv，columns_priv和host`。

* user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。

* db权限表：记录各个帐号在各个数据库上的操作权限。

* table_priv权限表：记录数据表级的操作权限。

* columns_priv权限表：记录数据列级的操作权限。

* host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。



### 触发器

- **Before Insert**
- **After Insert**
- **Before Update**
- **After Update**
- **Before Delete**
- **After Delete**

#### **使用场景**

- 可以通过数据库中的相关表实现级联更改。
- 实时监控某张表中的某个字段的更改而需要做出相应的处理。
- 例如可以生成某些业务的编号。
- 注意不要滥用，否则会造成数据库及应用程序的维护困难。
- 大家需要牢记以上基础知识点，重点是理解数据类型CHAR和VARCHAR的差异，表存储引擎InnoDB和MyISAM的区别。

### 常见问题

#### **char、varchar的区别是什么？**

> varchar是变长而char的长度是固定的。

#### **FLOAT和DOUBLE的区别是什么?**

- FLOAT类型数据可以存储至多8位十进制数，并在内存中占4字节。
- DOUBLE类型数据可以存储至多18位十进制数，并在内存中占8字节。

#### **请说明varchar和text的区别**

- varchar可指定字符数，text不能指定，内部存储varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，text是实际字符数+2个字节。
- text类型不能有默认值。
- varchar可直接创建索引，text创建索引要指定前多少个字符。varchar查询速度快于text,在都创建索引的情况下，text的索引几乎不起作用。
- 查询text需要创建临时表。

#### **varchar(50)中50的含义**

* 最多存放50个字符，varchar(50)和(200)存储hello所占空间一样，但后者在排序时会消耗更多内存，因为order by col采用fixed_length计算col长度(memory引擎也一样)。

#### **int(20)中20的含义**

* 是指显示字符的长度，不影响内部存储，只是当定义了ZEROFILL时，前面补多少个 0

#### SQL执行加载顺序

```sql
SELECT DISTINCT
	< select_list > 
FROM
	< left_table > 
	< join_type > JOIN < right_table > ON < join_condition > 
WHERE
	< where_condition > 
GROUP BY
	< group_by_list > 
HAVING
	< having_condition > 
ORDER BY
	< order_by_condition > 
	LIMIT < limit_number>
```

* 动态调整后执行顺序

```sql
FROM < left_table > 
ON < join_condition > 
< join_type > JOIN < right_table > 
WHERE < where_condition > 
GROUP BY < group_by_list > 使用聚集函数计算
HAVING < having_condition > 
SELECT 
DISTINCT < select_list > 
ORDER BY < order_by_condition > 
LIMIT < limit_number>
```

#### `EXISTS`和`IN`的比较

```mysql
select * from where cc IN (select cc from b)
select * from a where EXISTS (select c from b where b.cc=a.cc)
```

* 当表a的数据量小于b，用`EXISTS`,因为`EXISTS`相当于外表循环，实现的逻辑类似于：

  ```
  for i in a
  	for j in b
  		if j.cc == i.cc
  ```

* 当表a的数据量大于b，用`IN`实现的逻辑类似于：

  ```
  for i in b
  	for j in a
  	 	if j.cc == i.cc
  ```

  
