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

  ![img](https://www.holelin.cn/img/mysql/show_profile.png)

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

  
