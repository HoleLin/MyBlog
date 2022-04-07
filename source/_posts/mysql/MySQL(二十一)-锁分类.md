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
* [MySQL · 引擎特性 · InnoDB隐式锁功能解析](http://mysql.taobao.org/monthly/2020/09/06/)

### 锁的种类

> 在数据库中，通常使用**锁机制**来协调多个线程并发访问某一资源。MySQL的**锁类型**分为表锁和行锁，表示对整张表加锁，主要用在DDL场景中，也可以由用户指定，主要由server层负责管理；
>
> * 而行锁指的是锁定某一行或几行，或者是行与行之间的间隙，行锁由存储引擎管理，例如最常使用的InnoDB。
> * 表锁占用系统资源小，实现简单，但锁定粒度大，发生锁冲突概率高，并发度比较低。行锁占用系统资源大，锁定粒度小，发生锁冲突概率低，并发度比较高。

* InnoDB将锁分为**锁类型**和**锁模式**两类。锁类型包括表锁和行锁，而行锁还细分为记录锁、间隙锁、插入意向锁、Next-Key等更细的子类型。锁模式描述的是加什么锁,例如读锁和写锁

#### 锁模式

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

### 锁类型

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

### 显式锁和隐式锁

#### 显式锁(`Explicit Lock`)

* 显式锁(`Explicit Lock`)显式的加锁，在`SHOW ENGINE INNODB STATUS`中能够看到,会在内存中产生对象,占用内存.

  ```mysql
  SELECT ... FOR UPDATE;
  SELECT ... LOCK IN SHARE MODE;
  ```

#### 隐式锁(`Implicit Lock`)

* 当事务需要加锁的时，如果这个锁不可能发生冲突，InnoDB会跳过加锁环节，这种机制称为隐式锁。隐式锁是InnoDB实现的一种延迟加锁机制，其特点是只有在可能发生冲突时才加锁，从而减少了锁的数量，提高了系统整体性能。另外，隐式锁是针对被修改的`B+ Tree`记录，因此都是记录类型的锁，不可能是间隙锁或Next-Key类型。
* 隐式锁主要用在插入场景中。在Insert语句执行过程中，必须检查两种情况，
  * 一种是如果记录之间加有间隙锁，为了避免幻读，此时是不能插入记录的，
  * 另一种情况如果Insert的记录和已有记录存在唯一键冲突，此时也不能插入记录。
  * 除此之外，insert语句的锁都是隐式锁，但跟踪代码发现，insert时并没有调用`lock_rec_add_to_queue`函数进行加锁, 其实所谓隐式锁就是在Insert过程中不加锁。

* 隐式锁(`Implicit Lock`)是在索引中对记录逻辑的加锁，但是实际上不产生锁对象(不需要创建锁结构),不占用内存空间 

  ```mysql
  INSERT INTO xx VALUES( xx );
  -- 会对辅助索引加implicit lock
  UPDATE xx SET t = t + 1 WHERE id = 1;
  ```

##### 如何判断隐式锁是否存在

* InnoDB的每条记录中都一个隐含的`trx_id`字段，这个字段存在于聚集索引的`B+Tree`中。假设只有主键索引，则在进行插入时，行数据的`trx_id`被设置为当前事务id；假设存在二级索引，则在对二级索引进行插入时，需要更新所在`page`的`max_trx_id`。
* 因此对于主键，只需要通过查看记录隐藏列trx_id是否是活跃事务就可以判断隐式锁是否存在。 对于对于二级索引会相对比较麻烦，先通过二级索引页上的max_trx_id进行过滤，如果无法判断是否活跃则需要通过应用undo日志回溯老版本数据，才能进行准确的判断。

#### 隐式锁,显式锁转换

* 只有在特殊情况下，才会将隐式锁转换为显示锁。**这个转换动作并不是加隐式锁的线程自发去做的，而是其他存在行数据冲突的线程去做的**。例如事务1插入记录且未提交，此时事务2尝试对该记录加锁，那么事务2必须先判断记录上保存的事务id是否活跃，如果活跃则帮助事务1建立一个锁对象，而事务2自身进入等待事务1的状态，

* `Implicit Lock `转换成 `Explicit Lock`的时机:

  * 只有`Implicit Lock `产生冲突的时候，会自动转换成`Explicit Lock`,这样做的好处就是降低锁的开销.

    > 比如：我插入了一条记录10，本身这个记录加上implicit lock，如果这时候有人再去更新这条10的记录，那么就会自动转换成explicit lock

* 将记录上的隐式锁转换为显示锁是由文件`storage/innobase/lock/lock0lock.cc`中的函数`lock_rec_convert_impl_to_expl`完成的.

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

