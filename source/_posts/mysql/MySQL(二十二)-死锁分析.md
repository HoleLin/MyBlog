---
title: MySQL(二十二)-死锁分析
date: 2022-04-04 22:19:50
cover: /img/cover/MySQL.jpg
tags:
categories:
- MySQL
- 死锁分析
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

* 何登成--管中窥豹——MySQL(InnoDB)死锁分析之道
* [手把手教你解数据库死锁](https://juejin.cn/post/6944615453700390919)
* [Mysql并发时经典常见的死锁原因，Mysql死锁问题分析及解决方法](https://juejin.cn/post/6970589814051586062)
* [并发insert死锁验证](https://juejin.cn/post/6894624297961127944)

### 死锁和死锁检测

* 当**并发系统**在不同现场出现**循环资源依赖**,涉及的线程都在等待别的线程释放资源时,就会导致这几个线程都进入无限等待状态,称为死锁.

#### 死锁出现的条件

* 多个并发事务(两个或者两个以上的事务);

* 每个事务都持有锁(或者是已经在等待锁);
* 每个事务都需要再继续持有锁(为了完成事务逻辑,还必须更新更多的行);
* 事务之间产生加锁的循环等待,形成死锁.
  * 若A事务需要B的资源,B事务需要A的资源,这就是典型的AB-BA死锁.

### 死锁的危害

* 死锁，即表明有多个事务之间需要互相争夺资源而互相等待。
* 如果没有死锁检测，那么就会互相卡死，一直hang死
* 如果有死锁检测机制，那么数据库会自动根据代价来评估出哪些事务可以被回滚掉，用来打破这个僵局
* 所以说：死锁并没有啥坏处，反而可以保护数据库和应用
* 那么出现死锁，而且非常频繁，我们应该调整业务逻辑，让其避免产生死锁方为上策

#### MySQL出现死锁后的处理策略

* 第一种:直接进入等待,直到超时,这个超时时间可以通过参数`innodb_lock_wait_timeout`来设置.
* 第二种:发起死锁检测后,发现死锁后,主动回滚死锁链条中的某个事务,让其他事务得以继续执行.将参数`innodb_deadlock_detect`设置`on`,表示开启这个逻辑.

* 在InnoDB中,`innodb_lock_wait_timeout`的默认值为50s,意味着若采用第一种策略,当出现死锁以后,第一个被锁住的线程要过50s才超时退出,然后其他线程才有可能继续执行.对于在线服务来说,这个等待时间往往是无法接受的.因此大部分情况都采用第二种,在MySQL中`innodb_deadlock_detect`默认为`on`.

  ```mysql
  1213 - Deadlock found when trying to get lock; try restarting transaction, Time: 0.172000s
  ```

  * 主动死锁检测在发生死锁的时候,是能够快速发现并进行处理的,但是它也有额外负担的.每当一个事务被锁的时候,就要看看它所依赖的线程有没有被别人锁住,如此循环,最后判断是否出现了循环等待,也就是死锁.
  * 若所有的事务都要更新同一行的数据,每个新来的被堵住的线程,都要判断会不会由于自己的加入导致死锁,这个是一个时间复杂度是O(n)的操作.若有1000个并发线程要同时更新同一行,那么死锁检测操作就是100万的数量级.虽然最终检测的结果是没有死锁的,但是这期间要消耗大量的CPU资源.因此就会看到CPU利用率很高,但是每秒却执行不了几个事务.

* **怎么解决由这种热点行更新的性能问题呢?**

  * 直接的方法:若确保这个业务一定不会出现死锁,可以临时吧死锁检测关掉.
  * 控制并发度来降低死锁检测的成本.
    * 这个并发度控制要做在数据库服务端.可以选择修改MySQL源码.基本思路就是,对于相同行的更新,在进入引擎之前排队.

### MySQL死锁相关的参数

| 参数名                       | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `innodb_print_all_deadlocks` | `innodb_print_all_deadlocks = ON`如果这个参数打开，那么死锁相关的信息都会打印输出到error log |
| `innodb_lock_wait_timeout`   | 单位为秒<br/>当MySQL获取row lock的时候，如果wait了`innodb_lock_wait_timeout=N`的时间，会报以下错误  `ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction` |
| `innodb_deadlock_detect`     | 值有`OFF`和`ON`<br/>`OFF`: 可以关闭掉死锁检测，那么就发生死锁的时候，用锁超时来处理。<br/>`ON`:（默认选项）开启死锁检测，数据库自动回滚 |
| `innodb_status_output_locks` | `innodb_status_output_locks`是**动态参数**，默认是关闭的，如果想看更细粒度的锁信息，需要将此参数打开。 （注意`innodb_status_output_locks=off`时，`show engine innodb status`中仍然有锁信息，只是没有那么详细） |

* MySQL的死锁检测回滚事务,会回滚事务影响行数较小的事务.**即可以推测出MySQL死锁检测回滚事务具有不确定性.**

  > When deadlock detection is enabled (the default), InnoDB automatically detects transaction deadlocks and rolls back a transaction or transactions to break the deadlock. InnoDB tries to pick small transactions to roll back, where the size of a transaction is determined by the number of rows inserted, updated, or deleted.

### **死锁排查步骤**

  * 关注`SHOW ENGINE INNODB STATUS`命令`TRANSACTIONS`部分;

  * 若无异常信息则需要检查`performance_schema.mutex_instances`表,该表会列出自服务器启动以来所有的冲突.

    ```mysql
    SELECT * FROM mutex_instances WHERE LOCKED_BY_THREAD_ID IS NOT NULL
    ```

  * 要找出谁在等待这些冲突,可以查询`performance_schema.events_waits_current`表

    ```mysql
    SELECT THREAD_ID,EVENT_ID,EVENT_NAME,SOURCE,TIMER_START,OBJECT_INSTANCE_BEGIN,OPERATION
    FROM events_waits_current
    WHERE THREAD_id IN(SELECT LOCKED_BY_THREAD_ID FROM mutex_instances WHERE LOCKED_BY_THREAD_ID IS NOT NULL);
    ```

  * 输出中的`THREAD_ID`,是内部`mysqld`分配给线程的实际编号,而不是帮助找到产生死锁原因的连接线程的编号.要找到连接线程的编号,可以查询`performance_schema.threads`表.

    ```mysql
    SELECT * FROM threads;
    ```

  * 选择一个连接终止来解决死锁.

### 典型的死锁案例剖析

#### 案例环境

* MySQL: 8.0.26
* Transaction Isolation: `REPEATABLE-READ`(**RR**)
* `innodb_deadlock_detect`为`ON`
* Engine: InnoDB

#### 典型的AB-BA死锁

| sessionA                                      | sessionB                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| `START TRANSACTION WITH CONSISTENT SNAPSHOT;` | `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                |
| `select * from tb_b where id = 1 for update;` |                                                              |
|                                               | `select * from tb_a where id = 2 for update;`                |
| `select * from tb_a where id = 2 for update;` |                                                              |
|                                               | `select * from tb_b where id = 1 for update;`                |
|                                               | `ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction` |

```mysql
------------------------
LATEST DETECTED DEADLOCK
------------------------
2022-04-05 15:31:23 0x16e617000
*** (1) TRANSACTION:
TRANSACTION 383307, ACTIVE 22 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 1121, OS thread handle 6157840384, query id 9315 localhost 127.0.0.1 root statistics
select * from tb_a where id = 2 for update

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 3789 page no 4 n bits 72 index PRIMARY of table `test_data`.`tb_b` trx id 383307 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d93d; asc      =;;
 2: len 7; hex 81000000f90110; asc        ;;
 3: len 2; hex 6231; asc b1;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3788 page no 4 n bits 72 index PRIMARY of table `test_data`.`tb_a` trx id 383307 lock_mode X locks rec but not gap waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 00000005d947; asc      G;;
 2: len 7; hex 010000015910be; asc     Y  ;;
 3: len 2; hex 6132; asc a2;;


*** (2) TRANSACTION:
TRANSACTION 383308, ACTIVE 17 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 1120, OS thread handle 6160003072, query id 9319 localhost 127.0.0.1 root statistics
select * from tb_b where id = 1 for update

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 3788 page no 4 n bits 72 index PRIMARY of table `test_data`.`tb_a` trx id 383308 lock_mode X locks rec but not gap
Record lock, heap no 5 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 00000005d947; asc      G;;
 2: len 7; hex 010000015910be; asc     Y  ;;
 3: len 2; hex 6132; asc a2;;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3789 page no 4 n bits 72 index PRIMARY of table `test_data`.`tb_b` trx id 383308 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000005d93d; asc      =;;
 2: len 7; hex 81000000f90110; asc        ;;
 3: len 2; hex 6231; asc b1;;

*** WE ROLL BACK TRANSACTION (2)
```

* 死锁路径: [sessionA->sessionB, sessionB->sessionA]

  >  **死锁,永远跟加锁顺序相关.事务以相反的顺序操作数据,才会产生死锁.分析死锁就是挖掘出这个相反的顺序在哪.**

#### 同一个事务中，S-lock 升级为 X-lock 不能直接继承

* 前置准备

  ```mysql
  CREATE TABLE t (i INT) ENGINE = InnoDB;
  INSERT INTO t (i) VALUES(1);
  ```

| seesionA                                                     | sessionB                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                | `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                |
| -- 获取S-lock<br>`SELECT * FROM t WHERE i = 1 LOCK IN SHARE MODE; ` |                                                              |
|                                                              | -- 想要获取X-lock，但是被sessionA的S-lock 卡住，目前处于waiting lock阶段<br/>`DELETE FROM t WHERE i = 1; ` |
| -- 想要获取X-lock，sessionA本身拥有S-lock，但是由于session 2 获取X-lock之前，所以sessionA不能够从S-lock 提升到 X-lock，需要等待sessionB 释放才可以获取，所以造成死锁<br>` DELETE FROM t WHERE i = 1; ` |                                                              |
|                                                              | `ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction` |

```mysql
------------------------
LATEST DETECTED DEADLOCK
------------------------
2022-04-05 15:54:02 0x16e617000
*** (1) TRANSACTION:
TRANSACTION 383322, ACTIVE 19 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1128, 1 row lock(s)
MySQL thread id 1137, OS thread handle 6161444864, query id 9479 localhost root updating
DELETE FROM t WHERE i = 1

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 3790 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test_data`.`t` trx id 383322 lock_mode X waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000001000; asc       ;;
 1: len 6; hex 00000005d959; asc      Y;;
 2: len 7; hex 81000000fd0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3790 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test_data`.`t` trx id 383322 lock_mode X waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000001000; asc       ;;
 1: len 6; hex 00000005d959; asc      Y;;
 2: len 7; hex 81000000fd0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;


*** (2) TRANSACTION:
TRANSACTION 383323, ACTIVE 21 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 3 row lock(s)
MySQL thread id 1125, OS thread handle 6161084416, query id 9480 localhost root updating
DELETE FROM t WHERE i = 1

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 3790 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test_data`.`t` trx id 383323 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000001000; asc       ;;
 1: len 6; hex 00000005d959; asc      Y;;
 2: len 7; hex 81000000fd0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 3790 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test_data`.`t` trx id 383323 lock_mode X waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000001000; asc       ;;
 1: len 6; hex 00000005d959; asc      Y;;
 2: len 7; hex 81000000fd0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;

*** WE ROLL BACK TRANSACTION (1)
```

* 死锁路径: [sessionB -> sessionA , sessionA -> sessionB]

#### 唯一键死锁 （Delete + Insert）

* 关键点在于：S-lock

* 前置准备

  ```mysql
  CREATE TABLE `uk` (
    `a` int(11) NOT NULL,
    UNIQUE KEY `uniq_a` (`a`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  insert into `uk` values(1);
  ```

| sessionA                                                     | sessionB                                                     | sessionC                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                | `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                | `START TRANSACTION WITH CONSISTENT SNAPSHOT;`                |
| `delete from uk where a=1;`                                  |                                                              |                                                              |
|                                                              | -- wait lock(想要加S-lock，却被sessonA的X-lock卡住)<br/>`insert into uk values(1);` |                                                              |
|                                                              |                                                              | -- wait lock(想要加S-lock，却被sessonA的X-lock卡住)<br/>`insert into uk values(1);` |
| --sessionB和sessionC 都获得了S-lock，然后都想要去给记录1 加上X-lock，却互相被对方的S-lock卡住，死锁产生<br/>`commit;` |                                                              |                                                              |
|                                                              | `Query OK, 1 row affected (20.80 sec)`                       | `ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction` |

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

#### 主键和二级索引的死锁

* 前置准备

  ```mysql
  drop table if exists `index_test`;
  create table `index_test`(
  	`id` int auto_increment,
  	`second_id` int,
  	`third_id` int,
  	primary key(`id`),
  	key idx_1(`second_id`),
    key idx_2(`third_id`)
  );
  insert into index_test(`second_id`,`third_id`) values (10,100),(30,200),(20,300),(40,400);
  ```

```mysql
mysql> select * from index_test;
+----+-----------+----------+
| id | second_id | third_id |
+----+-----------+----------+
|  1 |        10 |      100 |
|  2 |        30 |      200 |
|  3 |        20 |      300 |
|  4 |        40 |      400 |
+----+-----------+----------+
4 rows in set (0.00 sec)
```

* 主键索引结构如下:

  ```
  * primary key
  1		2		3		4	   -- primary key
  10	30	20	40 	 -- idx_1 
  100	200	300	400	 -- idx_2
  ```

* `idx_1`二级索引结构如下:

  ```
  * idx_1
  10	20	30	40	
  1		3		2		4		
  ```

* `idx_2`二级索引结构如下:

  ```
  * idx_2
  100	200	300	400	
  1		2		3		4		
  ```

> select * from index_test where second_id > 10;: 锁二级索引顺序为：20 =>>30 ， 对应锁主键的顺序为：3 =>>2
>
> select * from index_test where third_id > 100：锁二级索引顺序为：200 =>>300 ， 对应锁主键的顺序为：2 =>>3
>
> 死锁路径：
> 	由于二级索引引起的主键加锁顺序： 3 =>>2
> 	由于二级索引引起的主键加锁顺序： 2 =>>3
>
> 这个要求并发，且刚好
>
> session 1 加锁3的时候 session 2 要加锁2.
> session 1 加锁2的时候 session 3 要加锁3.
>
> 这样就产生了 AB-BA 死锁
>
> **在MySQL中,以不同索引的过滤条件,来操作相同的记录(Delete/Update),很容易产生死锁**.

#### purge + unique key 引发的死锁

* 前置准备

  ```mysql
  drop table if exists `index_test_unique`;
  create table `index_test_unique`(
  	`id` int,
  	primary key(`id`)
  );
  insert into index_test_unique values(1),(10),(20),(30),(40),(50);
  ```

* 在并发情况下进行以下测试

| sessionA                                      | sessionB                                       |
  | --------------------------------------------- | ---------------------------------------------- |
  | `START TRANSACTION WITH CONSISTENT SNAPSHOT;` | `START TRANSACTION WITH CONSISTENT SNAPSHOT;`  |
  | `delete from index_test_unique where id =1; ` |                                                |
  |                                               | `delete from index_test_unique where id =10; ` |
  | `insert into index_test_unique values(10)`;   |                                                |
  |                                               | `insert into index_test_unique values(1)`;     |

#### REPLACE INTO问题

* Replace into操作可以算是比较常用的操作类型之一，当我们不确定即将插入的记录是否存在唯一性冲突时，可以通过Replace into的方式让MySQL自动处理：当存在冲突时，会把旧记录替换成新的记录。

### 总结

> * 要分析一个死锁,必须深入业务,了解整个事务的逻辑.
> * GAP锁很复杂,为了减少GAP锁,减少GAP导致的死锁,尽量选择`READ COMMITTED`隔离级别.
> * 适当的减少`UNIQUE`索引,能够减少GAP锁导致的死锁.
> * 在MySQL中,以不同索引的过滤条件,来操作相同的记录(Delete/Update),很容易产生死锁.
> * RC隔离级别下,如果死锁中出现Next-Key(Gap锁),说明表中一定存在`UNIQUE`索引.
> * 多语句事务产生的死锁,确保每条语句操作记录的顺序性,能够极大减少死锁.
> * 尽量不用`replace into`，用`insert into ... on duplicate key update `代替
