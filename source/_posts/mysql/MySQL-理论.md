---
title: MySQL-理论
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

### MySQL架构

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

    * 若使用MySQL5.7或更新版本,可以在每次执行一个比较大的操作后,通过执行`mysql_reset_connection`来重新初始化连接资源.

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

##### 执行器

* 开始执行的时候,需要先判断一下当前连接是否拥有对该表的执行查询权限,若没有则会返回没有权限的错误;若有权限就打开表继续执行.打开表的时候,执行器就会根据表的引擎定义,去使用这个引擎提供的接口.
* 执行器的执行流程
  * 调用InnoDB引擎接口获取这个表的第一行,若满足条件则将该行存入结果集中;不满足则跳过
  * 调用引擎接口获取"下一行",重复相同的判断逻辑,直到取到这个表的最后一行.
  * 执行器将上述遍历过程中满足条件的行组成记录集作为结果集返回客户端.
* 数据库的慢查询日志中有一个`rows_examined`的字段,表示这个语句执行过程中扫描了多少行.这个值就是执行器每次调用引擎获取数据行的时候累加的.
  * 在有些场景下,执行器调用一次,在引擎内部则扫描了多行,因此引擎扫描行数跟`rows_examined`并不是完全相同的.

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

### **MySQL的binlog录入格式**

- statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
- mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

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

  
