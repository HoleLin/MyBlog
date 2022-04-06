---
title: MySQL(二十四)-解读死锁日志
date: 2022-04-06 13:49:03
cover: /img/cover/MySQL.jpg
tags:
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

* [手把手教你解数据库死锁](https://juejin.cn/post/6944615453700390919)

### 死锁日志

#### 前置知识

* `supremum`记录

  > supremum这条记录是啥？
  >
  > 我们可以简单理解为是数据页中的一条“伪记录”。mysql的数据页中，不管有多少自己的记录，始终会存在两条虚拟的记录，也就是伪记录，分别是“Infimum”（最小记录）和“Supremum”（最大记录）。要知道，一个数据页中多条记录存储类似于链表，Infimum->1->2->3->...->Supremum，这种。

* 通过`SHOW ENGINE INNODB STATUS`查看

### 日志解析

* 日志来自于[MySQL(二十二)-死锁分析](https://www.holelin.cn/2022/04/04/mysql/MySQL(%E4%BA%8C%E5%8D%81%E4%BA%8C)-%E6%AD%BB%E9%94%81%E5%88%86%E6%9E%90/)的唯一键死锁 (Delete + Insert)案例

```mysql
------------------------
LATEST DETECTED DEADLOCK
------------------------
2022-04-05 16:11:20 0x16e617000
*** (1) TRANSACTION:
TRANSACTION 383343, ACTIVE 51 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 1137, OS thread handle 6161444864, query id 9523 localhost root update
insert into uk values(1)

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 3791 page no 4 n bits 72 index uniq_a of table `test_data`.`uk` trx id 383343 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 32
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d96a; asc      j;;
 2: len 7; hex 02000002a90d70; asc       p;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3791 page no 4 n bits 72 index uniq_a of table `test_data`.`uk` trx id 383343 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 32
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d96a; asc      j;;
 2: len 7; hex 02000002a90d70; asc       p;;


*** (2) TRANSACTION:
TRANSACTION 383344, ACTIVE 43 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 1138, OS thread handle 6162526208, query id 9539 localhost root update
insert into uk values(1)

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 3791 page no 4 n bits 72 index uniq_a of table `test_data`.`uk` trx id 383344 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 32
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d96a; asc      j;;
 2: len 7; hex 02000002a90d70; asc       p;;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3791 page no 4 n bits 72 index uniq_a of table `test_data`.`uk` trx id 383344 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 32
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d96a; asc      j;;
 2: len 7; hex 02000002a90d70; asc       p;;

*** WE ROLL BACK TRANSACTION (2)
```

#### 事务一相关

##### `TRANSACTION 383343, ACTIVE 51 sec inserting`

* `TRANSACTION 383343` 表示事务的ID
* ` ACTIVE 51 sec` 表示事务活动时间
* `inserting` 为事务当前正在运行的状态,其中常见的事务的状态有以下几种:
  * `fetching rows`
  * ``updating`
  * `deleting`
  * `inserting`

##### `mysql tables in use 1, locked 1`

* `tables in use 1` 表示有一个表被使用
* `locked 1` 表示有一个表锁

##### `LOCK WAIT 3 lock struct(s), heap size 1128, 2 row lock(s)`

* `LOCK WAIT` 表示事务正在等待锁
* `3 lock struct(s)` 表示该事务的锁链表的长度为3,每个链表节点代表该事务持有的一个锁结构,包括表锁,记录锁以及`AUTO-INC`锁等.
* `heap size 1128` 为事务分配的锁堆内存大小
* `2 row lock(s)` 表示当前事务持有的行锁个数,通过遍历上面提到的3个锁结构,找出其中类型为`LOCK_REC`的记录数

##### `MySQL thread id 1137, OS thread handle 6161444864, query id 9523 localhost root update`

* `MySQL thread id 1137` 表示MySQL的进程ID
* `query id 9523` 表示SQL的ID
* `localhost root update` 表示root@`localhost`执行的update操作

##### `insert into uk values(1)`

* 这里显示的是正在等待锁的 SQL 语句，可以看到，在执行insert时发生了锁等待，处于阻塞

##### `*** (1) HOLDS THE LOCK(S):`

* 表示事务一持有的锁的信息

##### `*** (1) WAITING FOR THIS LOCK TO BE GRANTED:`

* 字面意思，即表明当前事务正在等待的锁

##### `RECORD LOCKS space id 3791 page no 4 n bits 72 index uniq_a of table test_data.uk trx id 383343 lock_mode X locks rec but not gap waiting`

* `RECORD LOCKS` 事务在等待的锁为记录锁
* `space id 3791` 表空间号为3791
* ` page no 4` 数据页的页号为4
* `n bits 72` 对于行锁来说，一条记录就对应着一个比特位，一个页面中包含很多记录，用不同的比特位来区分到底是哪一条记录加了锁。为此在行锁结构的末尾放置了一堆比特位，这个n_bits属性代表使用了多少比特位。
* `index uniq_a of table test_data.uk` 索引信息,表明是二级索引`uniq_a`上的锁
* `lock_mode X locks rec but not gap`  记录锁
* `waiting` 等待中

> 如何根据日志去辨别究竟是什么锁呢？
>
> * 记录锁 - **lock_mode X locks rec but not gap**
> * 间隙锁 - **lock_mode X locks gap before rec**
> * 邻键锁（Next-key锁） - **lock_mode X**
> * 插入意向锁 - **lock_mode X locks gap before rec insert intention**
> 
> supremum记录上的锁的特殊性
> * 记录锁 - lock_mode X locks rec but not gap
> * 间隙锁 - lock_mode X
> * 邻键锁（Next-key锁） - lock_mode X
> * 插入意向锁 - lock_mode X insert intention

* InnoDB 首次创建索引时，会在根页面自动设置一条下确界记录和一条上确界记录，并且永远不会删除它们。它们为导航设置了一个有用的障碍，因此“ get-prev ”不会越过开头，“ get-next ”不会越过结尾。此外，infimum 记录可以是临时记录锁定的虚拟目标。
* 下确界和上确界记录可以被认为是索引页开销的一部分。最初，它们都存在于根页上，但随着索引的增长，下确界记录将存在于第一个或最低叶页上，而上确界记录将存在于最后一个或最大关键页上的最后一条记录。
