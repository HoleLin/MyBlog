---
title: MySQL(二十一)-锁分类
date: 2022-04-04 22:14:30
cover: /img/cover/MySQL.jpg
tags:
categories:
- MySQL
- 锁
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

* [MySQL锁系列（一）之锁的种类和概念](https://keithlan.github.io/2017/06/05/innodb_locks_1/)
* [15.7.1 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

### 锁的种类

#### 共享锁和排他锁([Shared and Exclusive Locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-shared-exclusive-locks))

* 共享锁(Shared Lock): permits the **transaction** that holds the lock to **read** a row.

  * 简称`S`锁.

  ```mysql
  SELECT ... LOCK IN SHARE MODE
  ```

* 排他锁(Exclusive Lock): permits the **transaction** that holds the lock to **update** or **delete** a row.

  * 简称`X`锁.

  ```mysql
  SELECT ... FOR UPDATE
  ```

#### 意向锁(Intention Locks)

* 意向锁是**表级锁**,表示事务稍后将对表中的行需要哪种类型的锁(共享或者独占).

* 有两种类型的意向锁:

  * 意向共享锁(`IS`):表示事务打算在表中的各个行上设置共享锁

    ```mysql
    SELECT ... FOR SHARE
    ```

  * 意向排它锁(IX):表示事务打算对表中的各个行设置排他锁。

    ```mysql
    SELECT ... FOR UPDATE
    ```

* 意向锁有以下规则:

  * 在事务可以获取表中行的共享锁之前，它必须首先获取`IS`表上的锁或更强的锁。
  * 在事务可以获取表中行的排他锁之前，它必须首先获取`IX` 表上的锁。

* 意图锁不会阻塞除了全表请求(`LOCK TABLES ... WRITE`)之外的任何东西.意向锁的主要目的是表明有人正在锁定一行，或者要锁定表中的一行。

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
  ```

##### 表级锁类型矩阵

|      | `X`      | `IX`     | `S`      | `IS`     |
| ---- | -------- | -------- | -------- | -------- |
| `X`  | Conflict | Conflict | Conflict | Conflict |
| `IX` | Conflict | √        | Conflict | √        |
| `S`  | Conflict | Conflict | √        | √        |
| `IS` | Conflict | √        | √        | √        |

#### 记录锁(Record Locks)

* 记录锁是对索引记录的锁.

  ```mysql
  SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;
  ```

* 记录锁总是**锁定索引记录**，即使定义的表没有索引。对于这种情况， `InnoDB`创建一个隐藏的聚集索引并将该索引用于记录锁定。

* 记录锁可以有两种类型：

  * **lock_mode X locks rec but not gap** 
  * **lock_mode S locks rec but not gap**

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
  trx id 10078 lock_mode X locks rec but not gap
  Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
   0: len 4; hex 8000000a; asc     ;;
   1: len 6; hex 00000000274f; asc     'O;;
   2: len 7; hex b60000019d0110; asc        ;;
  ```

#### 间隙锁

* 间隙锁是在**索引记录之间的间隙上的锁**，或在第一条索引记录之前或最后一条索引记录之后的间隙上的锁。

  > 例如，`SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;`阻止其他事务将值`15`插入 column `t.c1`，无论该列中是否已经存在任何此类值，因为该范围内所有现有值之间的间隙都已锁定。

  * **锁住的不是记录，而是范围**,比如;(negative infinity, 10),(10, 11)区间,这里都是开区间

* 间隙可能跨越单个索引值、多个索引值，甚至是空的。

* 可以显式禁用间隙锁定。如果您将事务隔离级别更改为 ，则会发生这种情况 `READ COMMITTED`。在这种情况下，间隙锁定对搜索和索引扫描禁用，仅用于**外键约束检查**和**唯一键约束检查**.

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  RECORD LOCKS space id 281 page no 5 n bits 72 index idx_c of table `lc_3`.`a` trx id 133588125 lock_mode X locks gap before rec  
  ```
  
* 间隙锁的目的: **实现可重读,防止幻读的产生**

#### Next-Key Locks

* Next-Key Locks是索引记录上的记录锁和索引记录之前的间隙锁的组合。

  * 相当于`Next-Key Lock = Record lock + Gap Lock `,不仅仅锁住记录，还会锁住间隙
  * `(negative infinity, positive infinity]`

* 默认情况下，`InnoDB`在 `REPEATABLE READ`事务隔离级别下运行.在这种情况下，`InnoDB`使用 next-key 锁进行搜索和索引扫描，这可以防止幻行.

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
  trx id 10080 lock_mode X
  Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
   0: len 8; hex 73757072656d756d; asc supremum;;
  
  Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
   0: len 4; hex 8000000a; asc     ;;
   1: len 6; hex 00000000274f; asc     'O;;
   2: len 7; hex b60000019d0110; asc        ;;
  ```

#### 插入意向锁(Insert Intention Locks)

* 可以理解为特殊的Gap锁的一种，用以提升并发写入的性能

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
  trx id 8731 lock_mode X locks gap before rec insert intention waiting
  Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
   0: len 4; hex 80000066; asc    f;;
   1: len 6; hex 000000002215; asc     " ;;
   2: len 7; hex 9000000172011c; asc     r  ;;...
  ```

#### 自增锁(AUTO-INC Locks)

* AUTO-INC Lock是一种特殊的表级锁,由插入到具有 `AUTO_INCREMENT`列的表中的事务使用

* 在`SHOW ENGINE INNODB STATUS`中 类似于以下内容：

  ```mysql
  TABLE LOCK table xx trx id 7498948 lock mode AUTO-INC waiting
  ```

#### 元数据锁(Metadata Lock)

* MySQL 使用元数据锁定来管理对数据库对象的并发访问并确保数据一致性。元数据锁定不仅适用于表，还适用于模式、存储程序（过程、函数、触发器、计划事件）、表空间、使用 `GET_LOCK()`函数获取的用户锁.

* Performance Schema `metadata_locks`表公开了元数据锁信息,这对于查看哪些会话持有锁、被阻塞等待锁等等很有用.

  > 当你执行select的时候，如果这时候有ddl语句，那么ddl会被阻塞，因为select语句拥有metadata lock，防止元数据被改掉. 

### 锁的其他分类

#### 显式锁和隐式锁

* 显式锁(`Explicit Lock`)显式的加锁，在`SHOW ENGINE INNODB STATUS`中能够看到,会在内存中产生对象,占用内存.

  ```mysql
  SELECT ... FOR UPDATE;
  SELECT ... LOCK IN SHARE MODE;
  ```

* 隐式锁(`Implicit Lock`)是在索引中对记录逻辑的加锁，但是实际上不产生锁对象,不占用内存空间 

  ```mysql
  INSERT INTO xx VALUES( xx );
  -- 会对辅助索引加implicit lock
  UPDATE xx SET t = t + 1 WHERE id = 1;
  ```

* `Implicit Lock `转换成 `Explicit Lock`的时机:

  * 只有`Implicit Lock `产生冲突的时候，会自动转换成`Explicit Lock`,这样做的好处就是降低锁的开销.

    > 比如：我插入了一条记录10，本身这个记录加上implicit lock，如果这时候有人再去更新这条10的记录，那么就会自动转换成explicit lock

* 数据库怎么知道`Implicit Lock`的存在呢？如何实现锁的转化呢?

  * 对于聚集索引上面的记录，有db_trx_id,如果该事务id在活跃事务列表中，那么说明还没有提交，那么`Implicit Lock`则存在  
  * 对于非聚集索引：由于上面没有事务id，那么可以通过上面的主键id，再通过主键id上面的事务id来判断，不过算法要非常复杂.

### 锁操作(待补充)

#### 锁迁移

* 又名锁继承 
* 满足的场景条件:
  * 锁住的记录是一条已经被标记为删除的记录，但是还没有被puge  
  * 然后这条被标记为删除的记录，被purge掉了  
  * 那么上面的锁自然而然就继承给了下一条记录，称之为锁迁移

#### 锁升级

* 一条全表更新的语句，那么数据库就会对所有记录进行加锁，那么可能造成锁开销非常大，可能升级为页锁，或者表锁。
* MySQL 没有锁升级

#### 锁分裂

* InnoDB的实现加锁，其实是在页上面做的，没有办法直接对记录加锁  
* 一个页被读取到内存，然后会产生锁对象，锁对象里面会有位图信息来表示哪些heapno被锁住，heapno表示的就是堆的序列号，可以认为就是定位到某一条记录  
* 由于B+tree的存在，当insert的时候，会产生页的分裂动作  
* 如果页分裂了，那么原来对页上面的加锁位图信息也就变了，为了保持这种变化和锁信息，锁对象也会分裂，由于继续维护分裂后页的锁信息

#### 锁合并

