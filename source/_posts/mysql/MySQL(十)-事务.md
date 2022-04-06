---
title: MySQL(十)-事务
mermaid: true
date: 2021-07-12 19:43:29
cover: /img/cover/MySQL.jpg
tags:
categories:
- MySQL
- 事务
updated: 2021-08-20 15:43:29
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

* 极客时间--MySQL实战45讲(林晓斌)
* 极客时间--SQL必知必会(陈旸)

#### 事务

* 事务是一个不可分割的数据库操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。事务是逻辑上的一组操作，要么都执行，要么都不执行(支持连续SQL的集体成功或集体撤销。);
* 事务支持在引擎层实现的;
* 需要利用 InnoDB 或 BDB 存储引擎，对自动提交的特性支持完成。
* InnoDB被称为事务安全型引擎。
* 注意:
  * 数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
  * 事务不能被嵌套.

#### 事务常用控制语句

* `SET autocommit = 0|1;`: 0表示关闭自动提交，1表示开启自动提交。
  * `SET autocommit`是永久改变服务器的设置，直到下次再次修改该设置。(**针对当前连接**)
  * 而`START TRANSACTION`记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(**针对当前事务**)

* `START TRANSACTION`或`BEGIN`:显示开启一个事务;

* `COMMIT` 提交事务。当提交事务后，对数据库的修改是永久性的

* `ROLLABACK`或者`ROLLBACK TO [SAVEPOINT]`为回滚，即撤销正在进行的所有没有提交的修改，或者将事务回滚到某个保存点;

* `SAVEPOINT`:再事务中创建保存点，方便后续针对保存点进行回滚。一个事务可以存在多个保存。

  ```mysql
  -- 设置一个事务保存点
  SAVEPOINT 保存点名称 
  -- 回滚到保存点
  ROLLBACK TO SAVEPOINT 保存点名称 
  -- 删除保存点
  RELEASE SAVEPOINT 保存点名称 
  ```

* `SET TRANSACTION`:设置事务的隔离级别

#### MySQL中的锁

> 数据库锁设计的初衷是处理并发问题.作为多用户共享的资源,当出现并发访问的时候,数据库需要合理地控制资源的访问规则.而锁就是用来实现这些访问规则的重要数据结构.
>
> 根据加锁的范围,MySQL里面的锁大致为**全局锁,表级锁和行锁**.

##### 语句

```mysql
-- 显示加读锁
LOCK TABLE ...READ
-- 显示加写锁
LOCK TABLE ...WRITE
```

##### 全局锁

* 全局锁就是对整个数据库实例加锁.MySQL提供了一个加全局读锁的方法,命令是`FLUSH TABLES WITH READ LOCK`(**FTWRL**),使用这个命令后,其他线程的一下语句会被阻塞:
  * **数据更新语句(数据的增删改);**
  * **数据定义语句(包括建表,修改表结构等);**
  * **更新类事务的提交事务;**
  
* 全局锁的典型使用场景是:**做全库逻辑备份**.也就是把整库每个表都`SELECT`出来存成文本;

* 通过FTWRL确定不会其他线程对数据库做更新,然后对整个库做备份.注意,在整个备份过程中整个库完全处于**只读状态**;让整个库都只读会导致:
  * 若在主库备份,那么备份期间都不能执行更新,业务基本上就得停摆;
  * 若在从库备份,那么备份期间从库不能执行主库同步过来的`binlog`会导致主从延迟;
  
* 不加锁的话,备份系统备份得到的库不是逻辑时间点,这个视图是逻辑不一致的.

* 官方自带的逻辑备份工具是`mysqldump`.当`mysqldump`使用参数`-single-transaction`的时候,导数据之前会启动一个事务,来保证拿到一致性视图.由于MVCC的支持,这个过程中数据是可以正常更新的.

  ```mysql
  -- mysqldump [选项] 数据库名 [表名] > 脚本名
  -- mysqldump [选项] --数据库名 [选项 表名] > 脚本名
  -- mysqldump [选项] --all-databases [选项] > 脚本名
  ```

  | 参数名                            | 缩写 | 含义                          |
  | --------------------------------- | ---- | ----------------------------- |
  | `--host`                          | -h   | 服务器IP地址                  |
  | `--port`                          | -P   | 服务器端口号                  |
  | `--user`                          | -u   | MySQL 用户名                  |
  | `--pasword`                       | -p   | MySQL 密码                    |
  | `--databases`                     |      | 指定要备份的数据库            |
  | `--all-databases`                 |      | 备份mysql服务器上的所有数据库 |
  | `--compact`                       |      | 压缩模式，产生更少的输出      |
  | `--comments`                      |      | 添加注释信息                  |
  | `--complete-insert`               |      | 输出完成的插入语句            |
  | `--lock-tables`                   |      | 备份前，锁定所有数据库表      |
  | `--no-create-db/--no-create-info` |      | 禁止生成创建数据库语句        |
  | `--force`                         |      | 当出现错误时仍然继续备份操作  |
  | `--default-character-set`         |      | 指定默认字符集                |
  | `--add-locks`                     |      | 备份数据库表时锁定数据库表    |

  * 备份示例

    ```mysql
    -- 备份所有数据库
    mysqldump -u root -p --all-databases > /backup/mysqldump/all.db
    -- 备份指定数据库
    mysqldump -u root -p test > /backup/mysqlump/test.db
    -- 备份指定数据库指定表(多个表以空格间隔)
    mysqldump -u root -p mysql table1 table2 > /backup/mysqldump/table1_and_table2.db
    -- 备份指定数据库排除某些表
    mysqldump -u root -p test --ignore-table=test.t1 --ignore-table=test.t2 > /backup/mysqlump/test2.db
    ```

  * 还原示例

    ```mysql
    -- 系统命令行直接还原 在导入备份数据库前,db_name如果没有,是需要创建的,而且与db_name.db中数据库名是一样的才可以导入
    mysqladmin -u root -p create db_name
    mysql -u root -p db_name < /back/mysqldump/db_name.db
    
    -- source方法
    mysql -u root -p 
    use db_name
    source /back/mysqldump/db_name.db
    ```

* **有了`mysqldump --single-transaction`以及`MVCC`的支持,为什么还需要`FTWRL`呢?**

  * 因为一致性读是好,但前提是引擎要支持这个隔离级别.MyISAM这种不支持事务的引擎,如果备份过程中有更新,总能读取到最新的数据,那么就破坏了备份的一致性.这是就需要使用`FTWRL`

* **所以`--single-transaction`方法只适用与所有的表使用了事务引擎的库.**

  * 如果有的表使用了不支持事务的引擎,那么备份只能通过`FTWRL`方法.

* **既然要全库只读,为什么不适用`set global readonly=true`的方式呢?**

  * 确实`readonly`方式也可以让全库进入只读状态,但是还是建议使用`FTWRL`方式,主要两个原因:
    * 在有些系统中,`readonly`的值会被用来做其他逻辑,比如用来判断一个库是主库还是从库.因此,修改`global`变量的方式影响面更大,不建议使用.
    * 在异常处理机制上有差异,如果执行`FTWRL`命令之后由于客户端发生异常断开,那么MySQL会自动释放这个全局锁.整个库回到可以正常更新的状态.而将整个库设置为`readonly`之后,如果客户端发生异常,则数据库就会一直保持`readonly`状态,这样会导致整个库长时间处于不可写状态,风险较高.

##### 表级锁

* 在MySQL里面表级别的锁有两种:一种是**表锁**,一种是**元数据锁(meta data lock,MDL)**;

  * 表锁语法是`lock tables ... read/write`.与`FTWRL`类似,可以用`unlock tables`主动释放锁,也可以在客户端断开的时候自动释放.需要注意,`lock tables`语法除了会限制别的线程的读写外,也限定了本线程接下来的操作对象.
    * 即在线程A中执行`lock tables t1 read,t2 write;`,则其他线程写他t1,读写t2的语句都会被阻塞.同时线程A在执行`unlock tables`之前,也只能执行读t1,读写t2.连写t1都不允许,自然也不能访问其他表.

* 另一类表级的锁是MDL(meta data lock),MDL不需要显示使用,在访问一个表的时候会被自动加上.MDL的作用是,保证读写的正确性,**防止DDL和DML并发冲突**;

  * 在MySQL5.5版本中引入MDL,当对一个表做增删改查操作的时候,加上MDL读锁;当要对表结构变更操作的时候,加MDL写锁.
  * **MDL会知道事务提交才释放,在做表结构变更的时候,需要小心不要导致锁住线上查询和更新.**
  * 在MySQL5.6版本引入`oneline DDL`,`oneline DDL`的过程大致如下:
    * 拿MDL写锁
    * 降级为MDL读锁
    * 真正做MDL
    * 升级成MDL写锁
    * 释放MDL锁

* **如何安全地给小表增加字段?**

  ```mysql
  -- command 1
  begin;
  select * from user limit 1;
  
  -- command 2 正常
  select * from user limit 1;
  
  -- command 3 Blocked
  alter table user add age int;
  
  -- commad 4 Blocked
  select * from user limit 1;
  ```

  * 首先要解决长事务,事务不提交,就会一直占着MDL锁.在MySQL的`information_schema`库的`innodb_trx`表中,可以查到当前执行中的事务.若要做DDL变更的表刚好有长事务在执行,要考虑暂停DDL,或者kill掉这个长事务.

  * 对于一个要变更的表是热点表,虽然数据量不大,但是上面的请求很频繁(开启事务,获取数据),并且不得不加字段该怎么操作?

    * 此时kill可能未必管用,因为新的请求马上就来了,比较理想的机制是,在`alter table`语句中设定等待时间,如果在这个指定的等待时间里面能够拿到MDL写锁最好,拿不到也不要阻塞后面的业务语句,先放弃.之后再重试命令重复这个过程.

  * `MariaDB`已经合并了`AliSQL`的这个功能,所以这两个开源分支目前都支持DDL `NOWAIT/WAIT n`这个语法

    ```mysql
    alter table table_name NOWAIT add column ...
    alter table table_name WAIT n add column ...
    ```

##### 行锁

* MySQL的行锁是在引擎层由各个引擎自己实现的.但并不是所有的引擎都支持行锁,比如MyISAM引擎就不支持行锁.不支持行锁意味着并发控制只能使用表锁,对于这种引擎的表,同一张表上任何时刻只能有一个更新在执行,这样就会影响业务并发度.

  * **如果事务中需要锁多个行,要把最可能造成锁冲突,最可能影响并发度的锁尽量往后放.**

    ```tcl
    1. 从顾客A账户余额中扣除电影票价
    2. 给影院B的账户余额增加这张电影票价
    3. 记录一条交易日志
    -------------------------
    同事顾客C也在影院B买票,语句2就可能会出现冲突,可以调整语句的顺序由[1->2->3]-->[3-->1-->2],减少2这行的锁时间,最大程度地减少事务之间的锁等待,提升并发度.(冲突的行之前有操作,在执行到冲突的行的时间可能会被错开)
    ```

###### 两阶段锁协议

* **在InnoDB事务中,行锁是在需要的时候才加上的,但并不是不需要了就立刻释放,而是要等到事务结束时才释放.这就是两阶段锁协议.**

#### 事务应该具有4个属性: ACID

* **Atomicity 原子性**

  > 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用;

* **Consistency 一致性**

  > 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的。

* **Isolation 隔离性**

  > 通常来说,一个事务所做的修改在最终提交以前,对其他事务是不可见的.
  >
  > 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的。

* **Durability 持久性**

  > 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

* **在这个四个特性中，原子性是基础，隔离性是手段，一致性是约束条件，而持久性是目的。**

#### 事务中常出现3个的问题

* **脏读(Dirty Read)**

  > 某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个事务RollBack了操作，则后一个事务所读取的数据就会是不正确的。**就是读到了别的事务回滚前的脏数据**,
  >
  > **当前事务读到的数据是别的事务想要修改成为的但是没有修改成功的数据。**

* **不可重复读(Non-repeatable Read)**

  > 在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。
  >
  > **当前事务先进行了一次数据读取，然后再次读取到的数据是别的事务修改成功的数据，导致两次读取到的数据不匹配.**
  >
  > 同一条记录的内容被修改了，重点在于UPDATE或DELETE

* **幻读(Phantom Read)**

  > 在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的
  >
  > **当前事务读第一次取到的数据比后来读取到数据条目不一致。**
  >
  > 查询某一个范围的数据行变多了或变少了，重点在与INSERT

#### SQL标准的事务隔离级别

![img](https://www.holelin.cn/img/mysql/%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.png)

* **读未提交(Read Uncommitted)**
  * 一个事务还没提交时,它做的变更就能被别的事务看到;
  
  * **别人改数据的事务尚未提交,我在我的事务中也能读到;**
  
  * 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
  
* **读提交(Read Committed)**

  * 一个事务提交之后,他的变更才能被其他事务看到;

  * **别人改数据的事务已提交,我在我的事务中才能读到;**
  * 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。

* **可重复读(Repeatable Read)**

  * 一个事务执行过程中看到的数据,总是跟这个事务在启动时看到的数据是一致.当然在可重复读隔离级别,未提交变更对其他事务也是不可见的;

  * **别人改数据的事务已经提交,我在我的事务中也不去读;**
  * 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

* **串行(Serializable)**
  * 对于同一行记录,"写"会加"写锁","读"会加"读锁".当出现读写锁冲突的时候,后访问的事务必须等前一个事务执行完成,才能继续执行;
  * **我的事务尚未提交,别人就不能改我的数据;**
  * 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

#### MySQL和Oracle默认隔离级别

* Oracle默认隔离级别为**Read Committed**隔离级别;
* MySQL 默认隔离级别为**Repeatable Read**隔离级别;

> 在实现上,数据库里面会创建一个视图,访问的时候以视图的逻辑结果为准.
>
> * 在"可重复读"隔离级别下,这个视图是在事务启动是创建的,整个事务存在期间都用这个视图.
> * 在"读提交"隔离级别下,这个视图是在每个SQL语句开始执行的时候创建的.
> * 这里需要注意的是,"读未提交"隔离级别下直接返回记录上的最新值,没有视图的概念;
> * 而"串行化"隔离别下直接用加锁的方式来避免并行访问.
>
> 事务隔离机制的实现基于锁机制和并发调度。其中并发调度使用的是MVVC（多版本并发控制），通过保存修改的旧版本信息来支持并发一致性读和回滚等特性。
>
> 因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是Read-Committed(读取提交内容)，但是你要知道的是InnoDB 存储引擎默认使用 **Repeatable-Read（可重读）**并不会有任何性能损失。
>
> InnoDB 存储引擎在分布式事务的情况下一般会用到**Serializable(可串行化)**隔离级别。

#### 设置事务隔离级别

* 参数设置: `trancsaction-isolation`设置为`READ-COMMITED`

##### 事务的启动: 

* 显示启动事务,`begin`或`start transaction`配套的提交语句`commit`,回滚语句`rollback`;
  * `begin/start transaction`命令并不是一个事务的起点,在执行到它们之后的第一个操作InnoDB表的语句,事务才真正启动;
    * `begin/start transaction`方式,一致性视图是在第一个快照读语句是创建的;
  * 若想要马上启动一个事务,可以使用`start transaction with consistent snapshot;`这个命令;
    * `start transaction with consistent snapshot;`方式,一致性视图是在执行`start transaction with consistent snapshot`时创建的.
    * `start transaction with consistent snapshot;`意思是从这个语句开始,创建一个持续整个事务的一致性快照.所以在读提交隔离级别下,这个用法就没意义了,等效普通的`start transaction`
* `set autocommit=0`,这个命令会将线程的自动提交关掉;

##### 查询MySQL全局事务隔离级别

```sql
-- 5.x
SELECT @@global.tx_isolation;
 -- 8.0
SELECT @@global.transaction_isolation;
```

##### 查询当前会话事务隔离级别

```mysql
 -- 5.x
 SELECT @@tx_isolation;
 -- 8.0
 SELECT @@transaction_isolation,
 SHOW VARIABLES LIKE 'transaction_isolation'
```

##### 设置当前会话的隔离级别

```mysql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

#### **隔离级别与锁的关系**

* 在`Read Uncommitted`级别下，读取数据不需要加共享锁，这样就不会跟被修改的数据上的排他锁冲突

* 在`Read Committed`级别下，读操作需要加共享锁，但是在语句执行完以后释放共享锁；

* 在`Repeatable Read`级别下，读操作需要加共享锁，但是在事务提交之前并不释放共享锁，也就是必须等待事务执行完毕以后才释放共享锁。

* `Serializable`是限制性最强的隔离级别，因为该级别锁定整个范围的键，并一直持有锁，直到事务完成。

### MVCC

* MVCC的英文全称为`Multiversion Concurrency Control `即**多版本并发控制技术**.MVCC是通过数据行的多个版本管理来实现数据库的并发控制,简单来说它的思想就是保存数据的历史版本,这样就可以通过比较版本号决定数据是否显示出来,读取数据的时候不需要加锁也可以保证事务的隔离效果.

#### MVCC可以解决哪些问题

* **读写之间阻塞的问题**,通过MVCC可以让读写互相不阻塞,即读不阻塞写,写不阻塞读,这样就可以提高事务的并发处理能力;
* **降低了死锁的概率**,这是因为MVCC采用了乐观锁的方式,读取数据并不需要加锁,对与写操作,也只锁定必要的行;
* **解决一致性读的问题**.一致性读也被称为快照读,当我们查询数据库在某个时间点的快照时,只能看到这个时间点之前的事务提交更新的结果,而不能看到这个时间点之后事务提交的更新结果.
  * **用于支持RC(`Read Committed`,读提交)和RR(`Repeatable Read`,可重复读)隔离级别的实现**

#### 快照读 当前读

* **快照读读取的是快照数据,不加锁的简单SELECT都属于快照读.**

  * `Read Committed`隔离级别：每次SELECT都生成一个快照读 
  * `Read Repeatable`隔离级别：开启事务后第一个SELECT语句才是快照读的地方，而不是一开启事务就快照读

* **当前读就是读取最新数据,而不是历史版本的数据**.

  * 加锁的SELECT或者对数据进行增删改都会进行当前读.
  
  ```mysql
  -- 读锁/共享锁(S锁)
  SELECT * FROM test LOCK IN SHARE MODE;
  -- 写锁/排他锁(X锁)
  SELECT * FROM test FOR UPDATE;
  INSERT INTO test VALUES ....;
  DELETE FROM test WHERE ...;
  -- 更新数据都是先读后写的
  UPDATE test SET ...;
  ```
  
  * **快照读就是普通的读操作,而当前读包括了加锁的读取和DML操作**

#### 事务的可重复读的能力是怎么实现的?

* **可重复读的核心是一致性读(consistent read);而事务更新数据的时候,只能用当前读,如果当前的记录的行锁被其他事务占用的话,就需要进入锁等待.**

#### InnoDb中MVCC是如何实现的

* InnoDB的MVCC是通过在每行记录后面保存两个隐藏的列来实现的.这两个列,一个保存了行的创建时间,一个保存了行的过期时间(或删除时间).实际上并不是存储真正的时间值而是**系统版本号(system version number)**.

  * 每开始一个新的事务,系统版本号都会自动递增.事务开始时刻的系统版本号会作为事务的版本号,用来和查询到的每行记录的版本号进行比较.

* 在RR(可重复读)隔离级别,MVCC针对`SELECT`,`INSERT`,`DELETE`,`UPDATE`具体操作

  ```
  SELECT:
  	InnoDB会根据以下两个条件检查每行记录:
  		InnoDB只查找早于当前版本的数据行(也就是行的系统版本号小于或等于事务的系统版本号),这样可以确保事务读取的行要么是在事务开始前已存在,要么是事务自身插入或修改过的.
  		行的删除版本要么未定义,要么大于当前事务版本号,这可以确保事务读取到的行,在事务开始之前未被删除.
  	只有满足上述两个条件的记录,才能返回作为查询结果.
  	
  INSERT: 
  	InnoDB为新插入的每一行保存当前系统版本号作为行版本号.
  DELETE:
  	InnoDB为删除的每一行保存当前系统版本号作为行删除标识.
  UPDATE:
  	InnoDB为插入一行新记录,保存当前系统版本号作为行版本号,同时保存当前系统版本号到原来的行作为行删除标识.
  ```

##### 事务版本号

* InnoDB里面每个事务有一个唯一的事务ID,叫做`transaction id`.它是在事务开始的时候向InnoDB的事务系统申请的,是按照申请顺序严格递增的.
* 每行数据也都是有多个版本的.每次事务更新数据的时候,都会生成一个新的数据版本,并且把`transaction id`赋值给这个版本的事务ID,记为`row_trx_id`.同时,旧的数据版本要保留,并且在新的数据版本中,能够有信息可以直接拿到它.也就是说,数据表中的一行记录,其实可以有多个版本(row),每个版本有自己的`row_trx_id`

##### 行记录的隐藏列

* InnoDB的叶子段存储了数据页,数据页中保存了行记录,而在行记录中有一些重要的隐藏字段
  * `db_row_id`:隐藏的行ID,用来生成默认聚集索引.若我们创建数据表的时候没有指定聚簇索引,这时InnoDB就会用这个隐藏ID来创建聚簇索引.采用聚簇索引的方式可以提升数据的查找效率;
  * `db_trx_id`:操作这个数据的事务ID,也就是最后一个对该数据进行插入或更新的事务ID;
  * `db_roll_prt`:回滚指针,也就是指向这个记录的`Undo Log`信息;
  
  <img src="https://www.holelin.cn/img/mysql/%E8%A1%8C%E8%AE%B0%E5%BD%95%E7%9A%84%E9%9A%90%E8%97%8F%E5%88%97.png" alt="img"  />

#### Undo Log

* InnoDB将行记录快照保存在Undo Log里,可以在回滚段中找到.

<img src="https://www.holelin.cn/img/mysql/UndoLog%E5%9B%9E%E6%BB%9A.png" alt="img" style="zoom:67%;" />

#### Read View是如何工作

* 在MVCC机制,多个事务对同一个行记录进行更新会产生多个历史快照,这些历史快照保存在Undo Log里.若一个事务想要查询这个行记录,想要读取哪个版本的行记录呢?

  * 这时需要用到ReadView ,它可以帮助我们解决行的可见性问题.**ReadView保存了当前事务开启时所有活跃(还没提交)的事务列表**,也可以理解为ReadView保存了不应该让这个事务看到的其他的事务ID列表.

* 在Read View中有几个重要的属性:

  * `trx_ids`:系统当前正在活跃的事务ID集合;
  * `low_limit_id`:活跃的事务中最大的事务ID;
  * `up_limit_id`:活跃的事务中最小的事务ID;
  * `creator_trx_id`:创建这个Read View的事务ID;

* 示例:如下图,`trx_ids`为`trx2`,`trx3`,`trx5`和`trx8`的集合,活跃的最大事务ID(`low_limit_id`)为`trx8`,活跃的最小的事务ID(`up_limit_id`)为`trx2`

  <img src="https://www.holelin.cn/img/mysql/MVCC%20Read%20View.png" alt="img" style="zoom:67%;" />
  
  ![img](https://www.holelin.cn/img/mysql/Read%20View.png)
  
* 假设当前有事务`creator_trx_id`想要读取某个行记录,这个行记录的事务ID为`trx_id`,那么会出现以下几种情况:

  * 如果`trx_id`<活跃的最小的事务ID(`up_limit_id`),也就是说这个行记录在这些活跃的事务创建之前就已经提交了,那么这个行记录对该事务是可见的.
  * 通过`trx_id`>活跃的最大的事务ID(`low_limit_id`),这说明该行记录在这些活跃的事务创建之后才创建,那么这个行记录对当前事务不可见.
  * 如果`up_limit_id<trx_id<low_limit_id`,说明该行记录所在事务`trx_id`在目前`creator_trx_id`这个事务创建的时候,可能还处于活跃的状态,因此需要在`trx_ids`集合中进行遍历,如果`trx_id`存在于`trx_ids`集合中,证明这个事务`trx_id`还处于活跃状态,不可见.否则,如果`trx_id`不存在与`trx_ids`集合中,证明事务`trx_id`已经提交了,该行记录可见.

* 查询一条记录的时候,系统如何通过多版本并发控制技术找到它:

  * 首先获取事务自己的版本号,也就是事务ID;
  * 获取Read View
  * 查询得到的数据,然后与Read View中的事务版本号进行比较;
  * 如果不符合Read View规则,就需要从Undo Log中获取历史快照;
  * 最后返回符合规则的数据;

* 在InnoDB中,MVCC是通过Undo Log+Read View进行数据读取,Undo Log保存了历史快照,而Read View规则帮助我们判断当前版本的数据是否可见.

* 需要说明的是,在隔离级别为读已提交(Read Commit)时,一个事务中的每一次SELECT查询都会获取依次Read View.

  * 在读已提交的隔离级别下,同时的查询语句都会重新获取一次Read View,这是如果Read View不同,就可能产生不可重复读或者幻读的情况.

* 当隔离级别为可重复读的时候,就避免了不可重复读,这是因为一个事务只在第一次的SELECT的时候获取依次ReadView,而在后面的所有SELECT都会复用这个ReadView.

##### InnoDB是如何解决幻读的

* 在可重复读的情况下,InnoDB可以通过Next-Key锁+MVCC来解决幻读问题.
* 在读已提交的情况下,及时采用了MVCC方式也会出先幻读.
  * 若同时开始事务A和事务B,现在事务A中进行某个条件范围的查询,读取的时候采用排它锁,在事务B中加一条符合条件范围的数据,并进行提交,然后我们在事务A中再次查询该条件范围的数据,就会发现结果集中多出了一个符合条件的数据,这样就出现了幻读.
* 出现幻读的原因是在读已提交的情况下,InnoDB只采用记录锁(Record Locking)
  * 记录锁:针对单个行记录添加锁
  * 间隙锁(Gap Locking): 可以帮我们锁住一个范围(索引之间的空隙),但不包括记录本身.采用间隙锁的方式可以防止幻读情况的产生.
  * Next-Key锁:帮我们锁住了一个范围,同时锁定记录本身,相当于间隙锁+记录锁,可以解决幻读的问题.
* 在隔离级别为可重复读时,InnoDB会采用Next-Key锁的机制,帮我们解决幻读问题.

