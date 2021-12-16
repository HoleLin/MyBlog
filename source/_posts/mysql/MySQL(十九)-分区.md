---
title: MySQL(十九)-分区
date: 2021-12-15 11:10:30
index_img: /img/cover/MySQL.jpg
cover: /img/cover/MySQL.jpg
tags:
- 分区
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

* 高性能MySQL(第三版)

### 分区

* MySQL在创建表时使用`PARTITION BY`子句定义每个分区存放的数据.在执行查询的时候,优化器会根据分区定义过滤那些没有我们需要数据的分区,这样查询就无需扫描所有分区,只需要查找包含需要数据的分区就可以了.
* 在下面的场景中,分区可以起到非常大的作用:
  * 表非常大以至于无法全部都放在内存中,或者只在表的最后部分有热点数据,其他均是历史数据.
  * 分区表的数据更容易维护.例如,想批量删除大量数据可以使用清除整个分区的方式,另外,还可以对一个独立分区进行优化,检查,修复等操作.
  * 分区表的数据可以分布在不同的物理设备上,从而高效地利用多个硬件设备.
  * 可以使用分区表来避免某些特殊的瓶颈,例如InnoDB的单个索引的互斥访问,ext3文件系统的inode锁竞争等.
  * 如果需要,还可以备份和恢复独立的分区,这在非常大的数据集的场景下效果非常好.
* 分区表本身也有一些限制
  * 一个表最多只能有1024分区.
  * 在MySQL5.1中,分区表达式必须是整数,或者是返回整数的表达式.在MySQL5.5中,某些场景中可以直接是用列来进行分区.
  * 如果分区字段中有主键或者唯一索引的列,那么所有主键列和唯一索引列都必须包含进来.
  * 分区表中无法使用外键约束.

#### MySQL提供的分区算法

> 分区依据的字段必须是主键的一部分，分区是为了快速定位数据，因此该字段的搜索频次较高应作为强检索字段，否则依照该字段分区毫无意义

#### hash(field)

* 相同的输入得到相同的输出。输出的结果跟输入是否具有规律无关。

* 仅适用于整型字段

  ```mysql
  create table article(
  	id int auto_increment PRIMARY KEY,
  	title varchar(64),
  	content text
  )PARTITION BY HASH(id) PARTITIONS 10
  ```

#### key(field)

* 和`hash(field)`的性质一样，只不过`key`是字符串的，比`hash()`多了一步从字符串中计算出一个整型在做取模操作。

  ```sql
  create table article_key(
  	id int auto_increment,
  	title varchar(64),
  	content text,
  	PRIMARY KEY (id,title)	-- 要求分区依据字段必须是主键的一部分
  )PARTITION by KEY(title) PARTITIONS 10
  ```

#### range算法

> 一种条件分区算法，按照数据大小范围分区（将数据使用某种条件，分散到不同的分区中）。

```mysql
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

#### list算法

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
