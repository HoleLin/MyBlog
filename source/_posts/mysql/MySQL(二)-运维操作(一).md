---
title: MySQL(二)-运维操作(一)
date: 2021-06-12 16:07:22
cover: /img/cover/MySQL.jpg
tags:
- 运维操作
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

#### 客户端操作

* 展示告警信息`SHOW  WARNINGS\G`

* 展示当前进程列表`SHOW PROCESSLIST;`或者`SELECT *  FROM information_schema.PROCESSLIST;`

  ```mysql
  mysql> SHOW PROCESSLIST;
  +----+-----------------+--------------------+------+---------+------+------------------------+------------------+
  | Id | User            | Host               | db   | Command | Time | State                  | Info             |
  +----+-----------------+--------------------+------+---------+------+------------------------+------------------+
  |  5 | event_scheduler | localhost           | NULL | Daemon  | 6854 | Waiting on empty queue | NULL             |
  |  9 | root            | 158.33.120.50:56237 | test | Sleep   | 5753 |                        | NULL             |
  | 10 | root            | 158.33.120.50:56291 | test | Sleep   | 5753 |                        | NULL             |
  | 11 | root            | 158.33.120.50:56313 | test | Sleep   | 6533 |                        | NULL             |
  | 13 | root            | 158.33.120.50:58336 | test | Sleep   | 1164 |                        | NULL             |
  | 14 | root            | 158.33.120.50:58337 | test | Sleep   |  579 |                        | NULL             |
  | 16 | root            | 158.33.120.50:58542 | test | Sleep   |  526 |                        | NULL             |
  | 17 | root            | 158.33.120.50:58553 | test | Query   |    0 | init                   | SHOW PROCESSLIST |
  +----+-----------------+--------------------+------+---------+------+------------------------+------------------+
  8 rows in set (0.02 sec)
  -- Info 表明当前正在进行的工作.如果展示的是查询语句,表明该语句正在执行;如果值是NULL,表明线程正在休眠,并等待下一条用户命令.
  
  mysql> SELECT *  FROM information_schema.PROCESSLIST;
  +----+-----------------+--------------------+------+---------+------+------------------------+----------------------------------------------+
  | ID | USER            | HOST               | DB   | COMMAND | TIME | STATE                  | INFO                                         |
  +----+-----------------+--------------------+------+---------+------+------------------------+----------------------------------------------+
  | 16 | root            | 58.33.120.50:58542 | test | Sleep   |  768 |                        | NULL                                         |
  |  9 | root            | 58.33.120.50:56237 | test | Sleep   | 5995 |                        | NULL                                         |
  | 17 | root            | 58.33.120.50:58553 | test | Query   |    0 | executing              | select * from information_schema.processlist |
  | 10 | root            | 58.33.120.50:56291 | test | Sleep   | 5995 |                        | NULL                                         |
  | 11 | root            | 58.33.120.50:56313 | test | Sleep   | 6775 |                        | NULL                                         |
  | 13 | root            | 58.33.120.50:58336 | test | Sleep   | 1406 |                        | NULL                                         |
  |  5 | event_scheduler | localhost          | NULL | Daemon  | 7096 | Waiting on empty queue | NULL                                         |
  | 14 | root            | 58.33.120.50:58337 | test | Sleep   |  821 |                        | NULL                                         |
  +----+-----------------+--------------------+------+---------+------+------------------------+----------------------------------------------+
  8 rows in set (0.06 sec)
  ```

* 展示MySQL服务器启动了多长时间,单位秒

  ```mysql
  mysql> SHOW GLOBAL STATUS LIKE 'uptime';
  +---------------+-------+
  | Variable_name | Value |
  +---------------+-------+
  | Uptime        | 6343  |
  +---------------+-------+
  1 row in set (10.45 sec)
  ```

* 监控InnoDB

  ```mysql
  mysql> SHOW ENGINE INNODB STATUS \G
  *************************** 1. row ***************************
    Type: InnoDB
    Name: 
  Status: 
  =====================================
  2021-09-09 08:17:36 0x7fe5100d8700 INNODB MONITOR OUTPUT
  =====================================
  Per second averages calculated from the last 36 seconds
  -----------------
  BACKGROUND THREAD
  -----------------
  srv_master_thread loops: 488 srv_active, 0 srv_shutdown, 6742 srv_idle
  srv_master_thread log flush and writes: 0
  ----------
  SEMAPHORES
  ----------
  -- SEMAPHORES部分包含了线程等待互斥锁或读写锁的信息.这里需要注意的是等待线程的数量或长时间等待的线程.
  OS WAIT ARRAY INFO: reservation count 108
  OS WAIT ARRAY INFO: signal count 104
  RW-shared spins 2, rounds 3, OS waits 1
  RW-excl spins 14, rounds 395, OS waits 12
  RW-sx spins 0, rounds 0, OS waits 0
  Spin rounds per wait: 1.50 RW-shared, 28.21 RW-excl, 0.00 RW-sx
  ------------
  TRANSACTIONS
  ------------
  -- TRANSACTIONS部分包含所有当期正在执行的事务的信息.
  Trx id counter 149616
  Purge done for trx's n:o < 149615 undo n:o < 0 state: running but idle
  History list length 0
  LIST OF TRANSACTIONS FOR EACH SESSION:
  ---TRANSACTION 422096949945600, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949944744, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949939608, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949943888, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949943032, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949942176, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949941320, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949940464, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949938752, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 422096949937896, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  --------
  FILE I/O
  --------
  I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
  I/O thread 1 state: waiting for completed aio requests (log thread)
  I/O thread 2 state: waiting for completed aio requests (read thread)
  I/O thread 3 state: waiting for completed aio requests (read thread)
  I/O thread 4 state: waiting for completed aio requests (read thread)
  I/O thread 5 state: waiting for completed aio requests (read thread)
  I/O thread 6 state: waiting for completed aio requests (write thread)
  I/O thread 7 state: waiting for completed aio requests (write thread)
  I/O thread 8 state: waiting for completed aio requests (write thread)
  I/O thread 9 state: waiting for completed aio requests (write thread)
  Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
   ibuf aio reads:, log i/o's:, sync i/o's:
  Pending flushes (fsync) log: 0; buffer pool: 0
  1466 OS file reads, 253660 OS file writes, 167430 OS fsyncs
  0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
  -------------------------------------
  INSERT BUFFER AND ADAPTIVE HASH INDEX
  -------------------------------------
  Ibuf: size 1, free list len 0, seg size 2, 0 merges
  merged operations:
   insert 0, delete mark 0, delete 0
  discarded operations:
   insert 0, delete mark 0, delete 0
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 0 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 2 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 5 buffer(s)
  0.00 hash searches/s, 0.00 non-hash searches/s
  ---
  LOG
  ---
  Log sequence number          70573738
  Log buffer assigned up to    70573738
  Log buffer completed up to   70573738
  Log written up to            70573738
  Log flushed up to            70573738
  Added dirty pages up to      70573738
  Pages flushed up to          70573738
  Last checkpoint at           70573738
  248902 log i/o's done, 0.00 log i/o's/second
  ----------------------
  BUFFER POOL AND MEMORY
  ----------------------
  Total large memory allocated 137363456
  Dictionary memory allocated 463338
  Buffer pool size   8192
  Free buffers       6780
  Database pages     1400
  Old database pages 496
  Modified db pages  0
  Pending reads      0
  Pending writes: LRU 0, flush list 0, single page 0
  Pages made young 190, not young 47
  0.00 youngs/s, 0.00 non-youngs/s
  Pages read 914, created 577, written 3459
  0.00 reads/s, 0.00 creates/s, 0.00 writes/s
  No buffer pool page gets since the last printout
  Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
  LRU len: 1400, unzip_LRU len: 0
  I/O sum[0]:cur[0], unzip sum[0]:cur[0]
  --------------
  ROW OPERATIONS
  --------------
  0 queries inside InnoDB, 0 queries in queue
  0 read views open inside InnoDB
  Process ID=1, Main thread ID=140621479102208 , state=sleeping
  Number of rows inserted 146892, updated 0, deleted 0, read 350581
  0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
  Number of system rows inserted 37, updated 335, deleted 13, read 20235
  0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
  ----------------------------
  END OF INNODB MONITOR OUTPUT
  ============================
  
  1 row in set (0.18 sec)
  ```

  * **innodb_locks表在8.0.13版本中由performance_schema.data_locks表所代替,innodb_lock_waits表则由performance_schema.data_lock_waits表代替。(保存以获取的锁和等待的锁的信息)**
  * Innodb_trx(保存正在执行的事务的信息)

#### `information_schema`中的表

| 表名         | 作用                         |
| ------------ | ---------------------------- |
| `INNODB_TRX` | 包含当前运行的所有事务的列表 |

#### `performance_schema`中的表

| 表名                | 作用                                                     |
| ------------------- | -------------------------------------------------------- |
| `INNODB_LOCKS`      | 包含事务持有的当前锁的相关信息以及每个事务等待的锁的信息 |
| `INNODB_LOCK_WAITS` | 包含事务正在等待的锁的信息                               |

* 调试并发问题时有用的典型的`information_schema`表查询

  * 关于事务正在等待的所有锁的信息`SELECT * FROM INNODB_LOCK_WAITS;`

  * 阻塞的事务列表

    ```mysql
    SELECT * FROM performance_schema.data_locks WHERE ENGINE_LOCK_ID IN (SELECT BLOCKING_ENGINE_LOCK_ID FROM performance_schema.data_lock_waits);
    ```

  * 特定表上的锁的列表

    ```mysql
    SELECT * FROM performance_schema.data_locks WHERE OBJECT_NAME='table_name';
    
    +--------+----------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
    | ENGINE | ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | EVENT_ID | OBJECT_SCHEMA | OBJECT_NAME | PARTITION_NAME | SUBPARTITION_NAME | INDEX_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
    +--------+----------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
    | INNODB | 140506252934808:1092:140506164241328   |                  4396 |       713 |       18 | xxxx          | table_name   | NULL           | NULL              | NULL       |       140506164241328 | TABLE     | IX            | GRANTED     | NULL      |
    | INNODB | 140506252928872:1092:140506164198896   |                  4395 |       712 |       38 | xxxx          | table_name   | NULL           | NULL              | NULL       |       140506164198896 | TABLE     | IX            | GRANTED     | NULL      |
    | INNODB | 140506252933960:1092:140506164235296   |                  4394 |       716 |       18 | xxxx          | table_name   | NULL           | NULL              | NULL       |       140506164235296 | TABLE     | IX            | GRANTED     | NULL      |
    | INNODB | 140506252932264:1092:140506164223232   |                  4393 |       715 |       18 | xxxx          | table_name   | NULL           | NULL              | NULL       |       140506164223232 | TABLE     | IX            | GRANTED     | NULL      |
    | INNODB | 140506252933112:1092:140506164229264   |                  4392 |       714 |       18 | lack          | table_name   | NULL           | NULL              | NULL       |       140506164229264 | TABLE     | IX            | GRANTED     | NULL      |
    | INNODB | 140506252933112:31:4:2:140506164226352 |                  4392 |       714 |       18 | lack          | table_name   | NULL           | NULL              | PRIMARY    |       140506164226352 | RECORD    | X,REC_NOT_GAP | GRANTED     | 1         |
    +--------+----------------------------------------+-----------------------+-----------+----------+---------------+-------------+----------------+-------------------+------------+-----------------------+-----------+---------------+-------------+-----------+
    6 rows in set (0.02 sec)
    ```

  * 等待锁的事务列表:

    ```mysql
    SELECT TRX_ID,TRX_REQUESTED_LOCK_ID,TRX_MYSQL_THREAD_ID,TRX_QUERY
    FROM information_schema.INNODB_TRX
    WHERE TRX_STATE='LOCK WAIT';
    ```

  * 找出事务正在等待哪种类型的锁

    ```mysql
    SELECT THREAD_ID,EVENT_NAME,SOURCE,OPERATION,PROCESSLIST_ID
    FROM events_waits_current JOIN threads USING (THREAD_ID)
    WHERE PROCESSLIST_ID
    ```

#### 设置变量

* MySQL支持两种形式的变量:`SESSION`以及`GLOBAL`

  * 回话级别的变量设置只会对当前连接生效,而不会影响其他连接

    ```
    set [session] var_name = value
    ```

  * `GLOBAL`变量配置后将应用此后创建的所有连接.但设置一个`GLOBAL`变量,并不会影响当前连接.

    ```
    set gloabl var_name = value
    ```

  * 所以如果需要在当前连接使用一个新的变量值,那么应该同时设置`SESSION`变量和`GLOBAL`变量

  * 变量值还原为默认值:`set [session] var_name = DEFAULT`

#### 导出数据

```mysql
-- 导出表数据
select * into outfile 文件地址 [控制格式] from 表名; 
-- 导入数据
load data [local] infile 文件地址 [replace|ignore] into table 表名 [控制格式]; 
    生成的数据默认的分隔符是制表符
    local未指定，则数据文件必须在服务器上
    replace 和 ignore 关键词控制对现有的唯一键记录的重复的处理
-- 控制格式
fields  控制字段格式
默认：fields terminated by '	' enclosed by '' escaped by '\'
    terminated by 'string'  -- 终止
    enclosed by 'char'      -- 包裹
    escaped by 'char'       -- 转义
-- 示例：
  SELECT a,b,a+b INTO OUTFILE '/tmp/result.text'
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
  LINES TERMINATED BY ''
  FROM test_table;
lines   控制行格式
默认：lines terminated by ''
    terminated by 'string'  -- 终止
```

#### 备份与还原

```mysql
备份，将数据的结构与表内数据保存起来。
利用 mysqldump 指令完成。
-- 导出
mysqldump [options] db_name [tables]
mysqldump [options] ---database DB1 [DB2 DB3...]
mysqldump [options] --all--database
1. 导出一张表
　　mysqldump -u用户名 -p密码 库名 表名 > 文件名(D:/a.sql)
2. 导出多张表
　　mysqldump -u用户名 -p密码 库名 表1 表2 表3 > 文件名(D:/a.sql)
3. 导出所有表
　　mysqldump -u用户名 -p密码 库名 > 文件名(D:/a.sql)
4. 导出一个库
　　mysqldump -u用户名 -p密码 --lock-all-tables --database 库名 > 文件名(D:/a.sql)
可以-w携带WHERE条件
-- 导入
1. 在登录mysql的情况下：
　　source  备份文件
2. 在不登录的情况下
　　mysql -u用户名 -p密码 库名 < 备份文件
```

#### 用户和权限管理

```mysql
-- root密码重置
1. 停止MySQL服务
2.  [Linux] /usr/local/mysql/bin/safe_mysqld --skip-grant-tables &
    [Windows] mysqld --skip-grant-tables
3. use mysql;
4. UPDATE `user` SET PASSWORD=PASSWORD("密码") WHERE `user` = "root";
5. FLUSH PRIVILEGES;
用户信息表：mysql.user
-- 刷新权限
FLUSH PRIVILEGES;
-- 增加用户
CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
    - 必须拥有mysql数据库的全局CREATE USER权限，或拥有INSERT权限。
    - 只能创建用户，不能赋予权限。
    - 用户名，注意引号：如 'user_name'@'192.168.1.1'
    - 密码也需引号，纯数字密码也要加引号
    - 要在纯文本中指定密码，需忽略PASSWORD关键词。要把密码指定为由PASSWORD()函数返回的混编值，需包含关键字PASSWORD
-- 重命名用户
RENAME USER old_user TO new_user
-- 设置密码
SET PASSWORD = PASSWORD('密码')  -- 为当前用户设置密码
SET PASSWORD FOR 用户名 = PASSWORD('密码') -- 为指定用户设置密码
-- 删除用户
DROP USER 用户名
-- 分配权限/添加用户
GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
    - all privileges 表示所有权限
    - *.* 表示所有库的所有表
    - 库名.表名 表示某库下面的某表
    GRANT ALL PRIVILEGES ON `pms`.* TO 'pms'@'%' IDENTIFIED BY 'pms0817';
-- 查看权限
SHOW GRANTS FOR 用户名

-- 查看当前用户权限
SHOW GRANTS; 
SHOW GRANTS FOR CURRENT_USER;  
SHOW GRANTS FOR CURRENT_USER();
-- 撤消权限
REVOKE 权限列表 ON 表名 FROM 用户名
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名   -- 撤销所有权限
-- 权限层级
-- 要使用GRANT或REVOKE，您必须拥有GRANT OPTION权限，并且您必须用于您正在授予或撤销的权限。
全局层级：全局权限适用于一个给定服务器中的所有数据库，mysql.user
    GRANT ALL ON *.*和 REVOKE ALL ON *.*只授予和撤销全局权限。
数据库层级：数据库权限适用于一个给定数据库中的所有目标，mysql.db, mysql.host
    GRANT ALL ON db_name.*和REVOKE ALL ON db_name.*只授予和撤销数据库权限。
表层级：表权限适用于一个给定表中的所有列，mysql.talbes_priv
    GRANT ALL ON db_name.tbl_name和REVOKE ALL ON db_name.tbl_name只授予和撤销表权限。
列层级：列权限适用于一个给定表中的单一列，mysql.columns_priv
    当使用REVOKE时，您必须指定与被授权列相同的列。
    
-- 权限列表
ALL [PRIVILEGES]    -- 设置除GRANT OPTION之外的所有简单权限
ALTER   -- 允许使用ALTER TABLE
ALTER ROUTINE   -- 更改或取消已存储的子程序

CREATE  -- 允许使用CREATE TABLE
CREATE ROUTINE  -- 创建已存储的子程序
CREATE TEMPORARY TABLES     -- 允许使用CREATE TEMPORARY TABLE
CREATE USER     -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
CREATE VIEW     -- 允许使用CREATE VIEW

DELETE  -- 允许使用DELETE
DROP    -- 允许使用DROP TABLE
EXECUTE     -- 允许用户运行已存储的子程序
FILE    -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
INDEX   -- 允许使用CREATE INDEX和DROP INDEX
INSERT  -- 允许使用INSERT
LOCK TABLES     -- 允许对您拥有SELECT权限的表使用LOCK TABLES
PROCESS     -- 允许使用SHOW FULL PROCESSLIST
REFERENCES  -- 未被实施
RELOAD  -- 允许使用FLUSH
REPLICATION CLIENT  -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE   -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
SELECT  -- 允许使用SELECT
SHOW DATABASES  -- 显示所有数据库
SHOW VIEW   -- 允许使用SHOW CREATE VIEW
SHUTDOWN    -- 允许使用mysqladmin shutdown
SUPER   -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections。
UPDATE  -- 允许使用UPDATE
USAGE   -- “无权限”的同义词
GRANT OPTION    -- 允许授予权限
```

#### 字符集编码

* 当在排序或者比较过程中遇到问题时,应当检查字符集选项与表的定义.

* 显示字符集变量值

  ```mysql
  SHOW VARIABLES LIKE 'character_set_%'   -- 查看所有字符集编码项
  SHOW VARIABLES LIKE '%%coll%';
  ```

  * `character_set_client`      客户端向服务器发送数据时使用的编码
  * `character_set_results`       服务器端将结果返回给客户端所使用的编码
  * `character_set_connection`    连接层编码

* 设置字符集集

  ```mysql
  SET 变量名 = 变量值
      SET character_set_client = gbk;
      SET character_set_results = gbk;
      SET character_set_connection = gbk;
  SET NAMES GBK;  -- 相当于完成以上三个设置
  ```

* 校对集

  ```mysql
  -- 校对集用以排序
  SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']   查看所有字符集
  SHOW COLLATION [LIKE 'pattern']     查看所有校对集
  CHARSET 字符集编码     设置字符集编码
  COLLATE 校对集编码     设置校对集编码
  ```



