---
title: MySQL优化方法
mermaid: true
date: 2021-06-12 19:47:32
index_img: /img/cover/MySQL.jpg
cover: /img/cover/MySQL.jpg
tags:
- 数据库优化方法
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

* [3万字总结，Mysql优化之精髓](https://juejin.cn/post/6844904183850598407)

### 为什么要优化

* 系统的吞吐量瓶颈往往出现在数据库的访问速度上
* 随着应用程序的运行，数据库的中的数据会越来越多，处理时间会相应变慢
* 数据是存放在磁盘上的，读写速度无法和内存相比

### 优化的方向

- 设计数据库时：数据库表、字段的设计，存储引擎
- 利用好`MySQL`自身提供的功能，如索引等
- 横向扩展：`MySQL`集群、负载均衡、读写分离
- `SQL`语句的优化（收效甚微）

### 字段优化

> 类型的选择，设计规范，范式，常见设计案例

#### 原则：尽量使用整型表示字符串

#### 原则：定长和非定长数据类型的选择

> `decimal`不会损失精度，存储空间会随数据的增大而增大。
>
> double占用固定空间，较大数的存储会损失精度。
>
> 非定长的还有`varchar`、`text`

* **字符串存储**

  > 定长`char`，非定长`varchar、text`（上限65535，其中`varchar`还会消耗1-3字节记录长度，而`text`使用额外空间记录长度）

#### 原则：尽可能选择小的数据类型和指定短的长度

#### 原则：尽可能使用 not null

> 非`null`字段的处理要比`null`字段的处理高效些！且不需要判断是否为`null`。
>
> `null`在`MySQL`中，不好处理，存储需要额外空间，运算也需要特殊的运算符。如`select null = null`和`select null <> null`（`<>`为不等号）有着同样的结果，只能通过`is null`和`is not null`来判断字段是否为`null`。
>
> 如何存储？`MySQL`中每条记录都需要额外的存储空间，表示每个字段是否为`null`。因此通常使用特殊的数据进行占位，比如`int not null default 0`、`string not null default ‘’`

#### 原则：字段注释要完整，见名知意

#### 原则：单表字段不宜过多

> 二三十个就极限了

#### 原则：可以预留字段

### 关联表的设计

> 外键`foreign key`只能实现一对一或一对多的映射

#### 一对多

> 使用外键

#### 多对多

> 单独新建一张表将多对多拆分成两个一对多

#### 一对一

> 如商品的基本信息（`item`）和商品的详细信息（`item_intro`），通常使用相同的主键或者增加一个外键字段（`item_id`）

#### 范式 Normal Format

> 数据表的设计规范，一套越来越严格的规范体系（如果需要满足N范式，首先要满足N-1范式）

#### 第一范式1NF：字段原子性

> 字段原子性，字段不可再分割。

#### 第二范式：消除对主键的部分依赖

> 即在表中加上一个与业务逻辑无关的字段作为主键

#### 第三范式：消除对主键的传递依赖

### 存储引擎选择

> `MySQL`可以将数据以不同的技术存储在文件(内存)中,这种技术称为存储引擎;
>
> 每一种存储引擎使用不同的存储机制,索引技巧,锁定水平,最终提供广泛且不同的功能

```
mysql> show engines;
+--------------------+---------+------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                               	    | Transactions | XA   | Savepoints |
+--------------------+---------+------------------------------------------------------------+--------------+------+------------+
| FEDERATED          | NO      | Federated MySQL storage engine                             | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables  | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                         | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                      | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine(anything you write to it disappears) | NO        | NO   | NO         |
| CSV                | YES     | CSV storage engine                                         | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                     | NO           | NO   | NO         |
+--------------------+---------+------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.13 sec)
```

#### 各种存储引擎的特点

| 特点           | MyISAM | InnoDB | Memory | MERGRE | NDB  |
| -------------- | ------ | ------ | ------ | ------ | ---- |
| 存储限制       | 256TB  | 64TB   | 有     | 没有   | 有   |
| 事务安全       | -      | 支持   | -      | -      | -    |
| 支持索引       | 支持   | 支持   | 支持   | 支持   | 支持 |
| 锁颗粒         | 表所   | 行锁   | 表锁   | 表锁   | 行锁 |
| 数据压缩       | 支持   | -      | -      | -      | -    |
| 支持外键       | -      | 支持   | -      | -      | -    |
| B树索引        | 支持   | 支持   | 支持   | 支持   | 支持 |
| 哈希索引       | -      | -      | 支持   | -      | 支持 |
| 全文索引       | 支持   | -      | -      | -      | -    |
| 集群索引       | -      | 支持   | -      | -      | -    |
| 索引缓存       | 支持   | 支持   | 支持   | 支持   | 支持 |
| 空间使用       | 低     | 高     |        | 低     | 低   |
| 内存使用       | 低     | 搞     | 中等   | 低     | 高   |
| 批量插入的速度 | 高     | 低     | 高     | 高     | 高   |

* **MyISAM**

  > MyISAM 是MySQL 的默认存储引擎。MyISAM 不支持事务、也不支持外键，其优势是访问的速度快，对事务完整性没有要求或者以SELECT、INSERT 为主的应用基本上都可以使用这个引擎来创建表。

* **InnoDB**

  > InnoDB 存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全。但是对比MyISAM的存储引擎，InnoDB 写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。
  >
  > 对于InnoDB 表，自动增长列必须是索引。如果是组合索引，也必须是组合索引的第一列，但是对于MyISAM 表，自动增长列可以是组合索引的其他列，这样插入记录后，自动增长列是按照组合索引的前面几列进行排序后递增的。
  >
  > ​    MySQL 支持外键的存储引擎只有InnoDB，在创建外键的时候，要求父表必须有对应的索引，子表在创建外键的时候也会自动创建对应的索引。在创建索引的时候，可以指定在删除、更新父表时，对子表进行的相应操作，包RESTRICT、CASCADE、SET NULL 和NO ACTION。其中RESTRICT 和NO ACTION 相同，是指限制在子表有关联记录的情况下父表不能更新；CASCADE 表示父表在更新或者删除时，更新或者删除子表对应记录；SET NULL 则表示父表在更新或者删除的时候，子表的对应字段被SET NULL。

* **MEMORY**

  > MEMORY 存储引擎使用存在内存中的内容来创建表。每个MEMORY 表只实际对应一个磁盘文件，格式是.frm。MEMORY 类型的表访问非常得快，因为它的数据是放在内存中的，并且默认使用HASH 索引，但是一旦服务关闭，表中的数据就会丢失掉。
  >
  > 在启动MySQL 服务的时候使用--init-file 选项，把INSERT INTO ... SELECT 或LOAD DATA INFILE 这样的语句放入这个文件中，就可以在服务启动时从持久稳固的数据源装载表。
  >
  > 服务器需要足够内存来维持所有在同一时间使用的MEMORY 表，当不再需要MEMORY表的内容之时，要释放被MEMORY 表使用的内存，应该执行DELETE FROM 或TRUNCATE TABLE，或者整个地删除表（使用DROP TABLE 操作）。

* **MERGE**

  > MERGE 存储引擎是一组MyISAM 表的组合，这些MyISAM 表必须结构完全相同，MERGE表本身并没有数据，对MERGE 类型的表可以进行查询、更新、删除的操作，这些操作实际上是对内部的实际的MyISAM 表进行的。对于MERGE 类型表的插入操作，是通过INSERT_METHOD 子句定义插入的表，可以有3 个不同的值，使用FIRST 或LAST 值使得插入操作被相应地作用在第一或最后一个表上，不定义这个子句或者定义为NO，表示不能对这个MERGE 表执行插入操作。
  >
  > 可以对MERGE 表进行DROP 操作，这个操作只是删除MERGE 的定义，对内部的表没有任何的影响

#### **如何选择合适的存储引擎**

> ***\*在选择存储引擎时，应根据应用特点选择合适的存储引擎，对于复杂的应用系统可以根据实际情况选择多种存储引擎进行组合\****
>
> * MyISAM：默认的MySQL 插件式存储引擎。**如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常适合的。**MyISAM 是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。
> *  InnoDB：**用于事务处理应用程序**，支持外键。**如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询以外，还包括很多的更新、删除操作，那么InnoDB 存储引擎应该是比较合适的选择**。InnoDB 存储引擎除了有效地降低由于删除和更新导致的锁定，还可以确保事务的完整提交（Commit）和回滚（Rollback），对于类似计费系统或者财务系统等对数据准确性要求比较高的系统，InnoDB 都是合适的选择。
> * MEMORY：将所有数据保存在RAM 中，在需要快速定位记录和其他类似数据的环境下，可提供极快的访问。MEMORY 的缺陷是对表的大小有限制，太大的表无法CACHE 在内存中，其次是要确保表的数据可以恢复，数据库异常终止后表中的数据是可以恢复的。MEMORY 表通常用于更新不太频繁的小表，用以快速得到访问结果。
> * MERGE：用于将一系列等同的MyISAM 表以逻辑方式组合在一起，并作为一个对象引用它们。MERGE 表的优点在于可以突破对单个MyISAM 表大小的限制，并且通过将不同的表分布在多个磁盘上，可以有效地改善MERGE 表的访问效率。这对于诸如数据仓储等VLDB环境十分适合

### 索引

> 关键字与数据的映射关系称为索引（包含关键字和对应的记录在磁盘中的地址）。关键字是从数据当中提取的用于标识、检索数据的特定内容。

### 查询缓存

> 缓存`select`语句的查询结果

##### 配置缓存

* 在配置文件中开启缓存,在`[mysqld]`段中配置`query_cache_type`

  * 0：不开启

  * 1：开启，默认缓存所有，需要在SQL语句中增加`select sql-no-cache`提示来放弃缓存

    ```sql
    select sql-no-cache * from user;
    ```

  * 2：开启，默认都不缓存，需要在SQL语句中增加`select sql-cache`来主动缓存(常用)

    ```sql
    select sql_cache * from user;
    ```

* 更改配置后需要重启以使配置生效，重启后可通过`show variables like ‘query_cache_type’;`来查看：

#### 设置客户端缓存

* 通过配置项`query_cache_size`来设置,`set global query_cache_size=64*1024*1024; `

##### 重置缓存

```sql
reset query cache;
```

##### 缓存失效问题

* 当数据表改动时，基于该数据表的任何缓存都会被删除。（表层面的管理，不是记录层面的管理，因此失效率较高）

* 注意事项
  * 应用程序，不应该关心`query cache`的使用情况。可以尝试使用，但不能由`query cache`决定业务逻辑，因为`query cache`由DBA来管理。
  * 缓存是以SQL语句为key存储的，因此即使SQL语句功能相同，但如果多了一个空格或者大小写有差异都会导致匹配不到缓存。

### 分区

> 一般情况下我们创建的表对应一组存储文件，使用`MyISAM`存储引擎时是一个`.MYI`和`.MYD`文件，使用`Innodb`存储引擎时是一个`.ibd`和`.frm`（表结构）文件。
>
> 当数据量较大时（一般千万条记录级别以上），MySQL的性能就会开始下降，这时我们就需要将数据分散到多组存储文件，保证其单个文件的执行效率.
>
> 最常见的分区方案是按`id`分区，如下将`id`的哈希值对10取模将数据均匀分散到10个`.ibd`存储文件中：
>
> ```sql
> create table article(
> 	id int auto_increment PRIMARY KEY,
> 	title varchar(64),
> 	content text
> )PARTITION by HASH(id) PARTITIONS 10
> 
> ```
>
> 服务端的表分区对于客户端是透明的，客户端还是照常插入数据，但服务端会按照分区算法分散存储数据。

##### MySQL提供的分区算法

> 分区依据的字段必须是主键的一部分，分区是为了快速定位数据，因此该字段的搜索频次较高应作为强检索字段，否则依照该字段分区毫无意义

###### hash(field)

* 相同的输入得到相同的输出。输出的结果跟输入是否具有规律无关。
* 仅适用于整型字段

###### key(field)

* 和`hash(field)`的性质一样，只不过`key`是字符串的，比`hash()`多了一步从字符串中计算出一个整型在做取模操作。

  ```sql
  create table article_key(
  	id int auto_increment,
  	title varchar(64),
  	content text,
  	PRIMARY KEY (id,title)	-- 要求分区依据字段必须是主键的一部分
  )PARTITION by KEY(title) PARTITIONS 10
  ```

###### range算法

> 一种条件分区算法，按照数据大小范围分区（将数据使用某种条件，分散到不同的分区中）。

```
create table article_range(
	id int auto_increment,
	title varchar(64),
	content text,
	created_time int,	-- 发布时间到1970-1-1的毫秒数
	PRIMARY KEY (id,created_time)	-- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY RANGE(created_time)(
	PARTITION p201808 VALUES less than (1535731199),	-- select UNIX_TIMESTAMP('2018-8-31 23:59:59')
	PARTITION p201809 VALUES less than (1538323199),	-- 2018-9-30 23:59:59
	PARTITION p201810 VALUES less than (1541001599)		-- 2018-10-31 23:59:59
);

```

* 注意：条件运算符只能使用`less than`，这以为着较小的范围要放在前面，比如上述`p201808,p201819,p201810`分区的定义顺序依照`created_time`数值范围从小到大，不能颠倒。

  ```sql
  insert into article_range values(null,'MySQL优化','内容示例',1535731180);
  flush tables;	-- 使操作立即刷新到磁盘文件
  ```

  由于插入的文章的发布时间`1535731180`小于`1535731199`（`2018-8-31 23:59:59`），因此被存储到`p201808`分区中，这种算法的存储到哪个分区取决于数据状况。

###### list算法

> 也是一种条件分区，按照列表值分区（`in (值列表)`）。

```sql
create table article_list(
	id int auto_increment,
	title varchar(64),
	content text,
	status TINYINT(1),	-- 文章状态：0-草稿，1-完成但未发布，2-已发布
	PRIMARY KEY (id,status)	-- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY list(status)(
	PARTITION writing values in(0,1),	-- 未发布的放在一个分区	
	PARTITION published values in (2)	-- 已发布的放在一个分区
);

sert into article_list values(null,'mysql优化','内容示例',0);
flush tables;
```

