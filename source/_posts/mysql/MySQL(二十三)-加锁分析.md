---
title: MySQL(二十三)-加锁分析
date: 2022-04-05 18:19:29
cover: /img/cover/MySQL.jpg
tags:
categories:
- MySQL
- 加锁分析
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

* [解决死锁之路 - 常见 SQL 语句的加锁分析](https://www.aneasystone.com/archives/2017/12/solving-dead-locks-three.html)
* 何登成--管中窥豹——MySQL(InnoDB)死锁分析之道
* [MySQL · 引擎特性 · InnoDB隐式锁功能解析](http://mysql.taobao.org/monthly/2020/09/06/)

### 加锁的目的

* 数据库中的锁: 确保并发更新场景下的数据正确性.
* ACID中的I(Isolation)

### 锁的持有周期

* 加锁:实际访问到某个待更新的行时,对其加锁(而非一开始就将所有的锁都一次性持有).
* 解锁:事务提交/回滚时(而非语句结束时,就释放).

### 锁粒度

* 数据库级别的锁
* 表级别的锁
* 页级别的锁
* 行级别的锁
* 锁定的粒度越细,并发级别越高.

### MySQL中会加锁的操作

#### 常见加锁操作

* `SELECT ... `语句正常情况下为快照读，不加锁；
* `SELECT ... LOCK IN SHARE MODE `语句为当前读，加 S 锁；
* `SELECT ... FOR UPDATE `语句为当前读，加 X 锁；
* 常见的 DML 语句（如 `INSERT`、`DELETE`、`UPDATE`）为当前读，加 X 锁；
* 常见的 DDL 语句（如` ALTER`、`CREATE` 等）加表级锁，且这些语句为隐式提交，不能回滚；
* 备份常用`FLUSH TABLES WITH READ LOCK`
* 唯一性约束检查(`PRIMARY KEY/UNIQUE KEY`)

#### 表锁

* 表锁（分 S 锁和 X 锁）
  * `LOCK TABLE...READ/WRITE`
* 意向锁（分 IS 锁和 IX 锁）
* 自增锁（一般见不到，只有在 innodb_autoinc_lock_mode = 0 或者 Bulk inserts 时才可能有）

#### 行锁

* 记录锁（分 S 锁和 X 锁）
* 间隙锁（分 S 锁和 X 锁）
* Next-key 锁（分 S 锁和 X 锁）
* 插入意向锁

##### 行锁分析

* 行锁都是加在索引上的，最终都会落在聚簇索引上；
* 加行锁的过程是一条一条记录加的；

#### MySQL特有的加锁操作

* Purge操作加锁

#### 不同隔离级别下的锁

* 在`Read Uncommitted`级别下，读取数据不需要加共享锁S，这样就不会跟被修改的数据上的排他锁X冲突

* 在`Read Committed`级别下，读操作需要加共享锁S，但是在语句执行完以后释放共享锁S；

* 在`Repeatable Read`级别下，读操作需要加共享锁S，但是在事务提交之前并不释放共享锁，也就是必须等待事务执行完毕以后才释放共享锁。

* `Serializable`是限制性最强的隔离级别，因为该级别锁定整个范围的键，并一直持有锁，直到事务完成。

----



### MySQL锁模式

#### 常规锁模式

* `LOCK_S`(读锁,共享锁,2)
* `LOCK_X`(写锁,排它锁,3)

#### 锁的属性

* `LOCK_REC_NOT_GAP`(锁记录,1024)
* `LOCK_GAP`(锁记录前的GAP,512)
* `LOCK_ORDINARY`(同时锁记录+记录前的GAP,Next-Key锁,0)
* `LOCK_INSERT_INTENTION`(插入意向锁,2048)

#### 锁组合(属性+模式)

* 锁的属性可以与锁模式任意组合,如`LOCK_REC_NOT_GAP(1023)+LOCK_X(3)`

### 锁冲突矩阵

* S 锁和 S 锁兼容，X 锁和 X 锁冲突，X 锁和 S 锁冲突；

| 列:存在锁<br>行:待加锁 | S(Not Gap) | S(Gap) | S(Ordinary) | X(Not Gap) | X(Gap) | X(Ordinary) | Insert Intention |
| ---------------------- | ---------- | ------ | ----------- | ---------- | ------ | ----------- | ---------------- |
| S(Not Gap)             |            |        |             | ×          |        | ×           |                  |
| S(Gap)                 |            |        |             |            |        |             | ×                |
| S(Ordinary)            |            |        |             | ×          |        | ×           | ×                |
| X(Not Gap)             | ×          |        | ×           | ×          |        | ×           |                  |
| X(Gap)                 |            |        |             |            |        |             | ×                |
| X(Ordinary)            | ×          |        | ×           | ×          |        | ×           | ×                |
| Insert Intention       |            |        |             |            |        |             |                  |

----



### 操作与加锁的对照关系

* 注: **以隔离级别为RC为例**

#### `INSERT`

##### 无`UNIQUE KEY`:

* ` LOCK_X+LOCK_REC_NOT_GAP(3+1024=1027)`

##### 有`UNIQUE KEY`

  * 唯一性约束检查: `LOCK_S+LOCK_ORDINARY(0+2=2)`
  * 插入的位置有GAP锁:`LOCK_INSERT_INTENTION(2048)`
  * 新数据插入:`LOCK_X+LOCK_REC_NOT_GAP(3+1024=1027)`

#### `DELETE`

* 满足删除条件的所有记录:`LOCK_X+LOCK_REC_NOT_GAP`

#### `UPDATE`

##### `UPDATE`操作分解

1. 定位到**下一条**满足查询条件的记录(查询过程,类似于Select/Delete)
2. 删除当前定位到的记录(标记为删除状态)
3. 拼装更新后项,根据更新后项定位到**新的插入位置**
4. 在新的插入位置,判断是否存在`UNIQUE`冲突(有`UNIQUE KEY`时)
5. 插入更新后项(不存在`UNIQUE`冲突时)
6. 重复1~5的操作,直到扫描完整个查询范围

##### `UPDATE`操作分析

* 1~2: `DELETE`
* 3,4,5: `INSERT`

##### `UPDATE`操作与加锁对照关系

###### 无`UNIQUE KEY`

* 查询范围中的所有记录:`LOCK_X+LOCK_REC_NOT_GAP`

###### 有`UNIQUE KEY`

* 查询满足条件的记录: 查询范围内的所有记录加锁,`LOCK_X+LOCK_REC_NOT_GAP`
* 更新后项存在唯一性冲突: 冲突项上的加锁,`LOCK_S+LOCK_ORDINARY`
* 更新后项不存在唯一性冲突: 更新位置后项加锁,`LOCK_S+LOCK_GAP`
* 实际更新操作:可看作插入了一条新记录,`LOCK_X+LOCK_REC_NOT_GAP`

#### 加`GAP`锁的情况

* `READ COMMITTED`(**RC**): `UNIQUE KEY`唯一性约束检查,`Purge`操作;
* `REPEATABLE READ`(**RR**): RC的基础上,**所有需要加锁的索引范围扫描**;

##### 原则

> * GAP锁很复杂,为了减少GAP锁,减少GAP导致的死锁,尽量选择`READ COMMITTED`隔离级别(RC+row based binlog,基本上能够解决所有问题,无需使用`REPEATABLE READ`)
> * 适当的减少`UNIQUE`索引,能够减少GAP锁导致的死锁(根据业务情况而定).
