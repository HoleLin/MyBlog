---
title: MySQL语句
mermaid: true
date: 2021-06-12 19:43:29
cover: /img/cover/MySQL.jpg
tags:
- 语句
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

* [不藏了，我的一千行 MySQL 学习笔记（2万字长文）](https://mp.weixin.qq.com/s/gAoosK9vAGxeCB7rnik1GA)

### MySQL客户端

* MySQL登录:`mysql 参数`

  | 参数                 | 描述                                                         |
  | -------------------- | ------------------------------------------------------------ |
  | -D,--database = name | 打开指定的数据库                                             |
  | --delimiter = name   | 指定分割符                                                   |
  | -h,--host = name     | 服务器名称                                                   |
  | -p,--port = #        | 端口号                                                       |
  | --prompt = name      | 设置提示符,[\D:完整的日期;\d: 当前数据库;\h:服务器名称;\u: 当前用户] |
  | -u,--user = name     | 用户名                                                       |
  | -V,--version         | 输出版本信息并且推出                                         |

* MySQL退出: `exit/quit/\q`

* 显示数据库信息:`\s`

  ```sql
  
  Sun Jun 13 16:30:30 2021@localhost@holelin@(none) >\s
  --------------
  mysql  Ver 8.0.24 for Linux on x86_64 (MySQL Community Server - GPL)
  
  Connection id:          284
  Current database:
  Current user:           holelin@localhost
  SSL:                    Not in use
  Current pager:          stdout
  Using outfile:          ''
  Using delimiter:        ;
  Server version:         8.0.24 MySQL Community Server - GPL
  Protocol version:       10
  Connection:             Localhost via UNIX socket
  Server characterset:    utf8mb4
  Db     characterset:    utf8mb4
  Client characterset:    utf8mb4
  Conn.  characterset:    utf8mb4
  UNIX socket:            /var/lib/mysql/mysql.sock
  Binary data as:         Hexadecimal
  Uptime:                 37 days 20 hours 29 min 15 sec
  
  Threads: 2  Questions: 315  Slow queries: 0  Opens: 301  Flush tables: 6  Open tables: 128  Queries per second avg: 0.000
  --------------
  
  ```

* 查看MySQL默认读取`my.cnf`的目录

  * 若没有设置使用指定目录的`my.cnf`,MySQL启动是会读取安装目录及默认目录下的`my.conf`,查看MySQL默认读取`my.conf的目录

    ```sh
    [root@holelin docs]# mysql --help |grep 'my.cnf'
                          order of preference, my.cnf, $MYSQL_TCP_PORT,
    /etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
    ```
  
* 显示哪些线程正在运行: `SHOW PROCESSLIST;`

* 显示系统变量信息: `SHOW VARIABLES;`

* 在执行`explain`后面显示告警

  * 开启:`\W`
  * 关闭:`\w`

  ```mysql
  mysql> \W
  Show warnings enabled.
  mysql> explain select * from user\G
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: user
     partitions: NULL
           type: ALL
  possible_keys: NULL
            key: NULL
        key_len: NULL
            ref: NULL
           rows: 1
       filtered: 100.00
          Extra: NULL
  1 row in set, 1 warning (0.00 sec)
  
  Note (Code 1003): /* select#1 */ select `auth`.`user`.`u_id` AS `u_id`,`auth`.`user`.`name` AS `name`,`auth`.`user`.`nick_name` AS `nick_name`,`auth`.`user`.`password` AS `password`,`auth`.`user`.`delete_flag` AS `delete_flag` from `auth`.`user`
  
  mysql> \w
  Show warnings disabled.
  mysql> explain select * from user\G
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: user
     partitions: NULL
           type: ALL
  possible_keys: NULL
            key: NULL
        key_len: NULL
            ref: NULL
           rows: 1
       filtered: 100.00
          Extra: NULL
  1 row in set, 1 warning (0.00 sec)
  ```

* 查看服务器已启动的时间,单位是秒:`show global status like 'uptime';`

  ```mysql
  mysql> show global status like 'uptime';
  +---------------+---------+
  | Variable_name | Value   |
  +---------------+---------+
  | Uptime        | 1911726 |
  +---------------+---------+
  1 row in set (0.00 sec)
  ```

* InnoDB监控器:`show engine innodb status\G`

  ```mysql
  mysql> show engine innodb status\G
  *************************** 1. row ***************************
    Type: InnoDB
    Name: 
  Status: 
  =====================================
  2021-08-27 18:05:50 140320345196288 INNODB MONITOR OUTPUT
  =====================================
  Per second averages calculated from the last 10 seconds
  -----------------
  BACKGROUND THREAD
  -----------------
  srv_master_thread loops: 188 srv_active, 0 srv_shutdown, 1914135 srv_idle
  srv_master_thread log flush and writes: 0
  ----------
  SEMAPHORES
  ----------
  OS WAIT ARRAY INFO: reservation count 139
  OS WAIT ARRAY INFO: signal count 138
  RW-shared spins 0, rounds 0, OS waits 0
  RW-excl spins 0, rounds 0, OS waits 0
  RW-sx spins 0, rounds 0, OS waits 0
  Spin rounds per wait: 0.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
  ------------
  TRANSACTIONS
  ------------
  Trx id counter 6900
  Purge done for trx's n:o < 6897 undo n:o < 0 state: running but idle
  History list length 0
  LIST OF TRANSACTIONS FOR EACH SESSION:
  ---TRANSACTION 421795506855320, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 421795506856176, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 421795506854464, not started
  0 lock struct(s), heap size 1136, 0 row lock(s)
  ---TRANSACTION 421795506853608, not started
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
  1438 OS file reads, 9113 OS file writes, 5559 OS fsyncs
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
  Hash table size 34679, node heap has 0 buffer(s)
  Hash table size 34679, node heap has 0 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 1 buffer(s)
  Hash table size 34679, node heap has 2 buffer(s)
  Hash table size 34679, node heap has 4 buffer(s)
  0.00 hash searches/s, 0.00 non-hash searches/s
  ---
  LOG
  ---
  Log sequence number          39315854
  Log buffer assigned up to    39315854
  Log buffer completed up to   39315854
  Log written up to            39315854
  Log flushed up to            39315854
  Added dirty pages up to      39315854
  Pages flushed up to          39315854
  Last checkpoint at           39315854
  2487 log i/o's done, 0.00 log i/o's/second
  ----------------------
  BUFFER POOL AND MEMORY
  ----------------------
  Total large memory allocated 137035776
  Dictionary memory allocated 848877
  Buffer pool size   8192
  Free buffers       6527
  Database pages     1655
  Old database pages 590
  Modified db pages  0
  Pending reads      0
  Pending writes: LRU 0, flush list 0, single page 0
  Pages made young 1190, not young 797
  0.00 youngs/s, 0.00 non-youngs/s
  Pages read 1128, created 539, written 4853
  0.00 reads/s, 0.00 creates/s, 0.00 writes/s
  No buffer pool page gets since the last printout
  Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
  LRU len: 1655, unzip_LRU len: 0
  I/O sum[0]:cur[0], unzip sum[0]:cur[0]
  --------------
  ROW OPERATIONS
  --------------
  0 queries inside InnoDB, 0 queries in queue
  0 read views open inside InnoDB
  Process ID=25352, Main thread ID=140320034805504 , state=sleeping
  Number of rows inserted 430, updated 24, deleted 3, read 5411
  0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
  Number of system rows inserted 2076, updated 1679, deleted 944, read 41722
  0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
  ----------------------------
  END OF INNODB MONITOR OUTPUT
  ============================
  
  1 row in set (0.11 sec)
  ```

##### 使用`perror`工具查询MySQL错误编号

```sh
[root@holelin ~]# perror 150
MySQL error code MY-000150 (handler): Foreign key constraint is incorrectly formed
[root@holelin ~]# perror 1064
MySQL error code MY-001064 (ER_PARSE_ERROR): %s near '%-.80s' at line %d
```

### 操作数据库

#### 显示所有数据库

```sql
SHOW DATABASES;
```

#### 选择(打开)数据库:

```sql
USE db_name;
```

#### 创建数据库

```sql
-- 格式
CREATE {DATABASE|SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name;

-- 示例
mysql> create database if not exists holelin_test character set  = utf8mb4;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| holein_test        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

```

#### 查看当前服务器下的数据库列表

```sql
SHOW {DATABASES|SCHEMAS} [LIKE 'pattern' | WHERE expr]; 

mysql> show schemas like 'hole%';
+------------------+
| Database (hole%) |
+------------------+
| holelin_test      |
| holelin          |
+------------------+
2 rows in set (0.00 sec)

```

#### 查看警告信息

```sql
SHOW WARNINGS;
```

#### 查看数据库创建时使用的指令信息

```sql
SHOW CREATE DATABASE database_name;
mysql> show create database holelin_test;
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+
| Database     | Create Database                                                                                                                        |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+
| holelin_test | CREATE DATABASE `holelin_test` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */ |
+--------------+----------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

#### 修改数据库编码方式

```sql
ALTER {DATABASE|SCHEMA} db_name [DEFAULT] CHARACTER SET [=] charset_name;

mysql> alter database holelin_test character set utf8;
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> show create database holelin_test;
+--------------+----------------------------------------------------------------------------------------------------------+
| Database     | Create Database                                                                                          |
+--------------+----------------------------------------------------------------------------------------------------------+
| holelin_test | CREATE DATABASE `holelin_test` /*!40100 DEFAULT CHARACTER SET utf8 */ /*!80016 DEFAULT ENCRYPTION='N' */ |
+--------------+----------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

```

#### 删除数据库

```sql
DROP {DATABASE|SCHEMA} [IF EXISTS] db_name;
```

#### 显示当前数据库

```sql
SELECT DATABASE();
Mysql> use holelin_test;
Database changed
mysql> select database();
+--------------+
| database()   |
+--------------+
| holelin_test |
+--------------+
1 row in set (0.00 sec)
```

### 操作表

#### 创建表

```sql
-- TEMPORARY 临时表，会话结束时表自动消失
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] [库名.]table_name(
	column_name data_type;
    -- 字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
	....
) [ 表选项];
-- 表选项
    -- 字符集
        CHARSET = charset_name
        如果表没有设定，则使用数据库字符集
    -- 存储引擎
        ENGINE = engine_name
        表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
        常见的引擎：InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
        不同的引擎在保存表的结构和数据时采用不同的方式
        MyISAM表文件含义：.frm表定义，.MYD表数据，.MYI表索引
        InnoDB表文件含义：.frm表定义，表空间数据和日志文件
        SHOW ENGINES -- 显示存储引擎的状态信息
        SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
    -- 自增起始数
        AUTO_INCREMENT = 行数
    -- 数据文件目录
        DATA DIRECTORY = '目录'
    -- 索引文件目录
        INDEX DIRECTORY = '目录'
    -- 表注释
        COMMENT = 'string'
    -- 分区选项
        PARTITION BY ... 
        
-- CREATE/SELECT
CREATE TABLE [IF NOT EXISTS] table_name(
	column_name data_type;
	....
) SELECT ...;
```

#### 查看数据库中数据表列表

```sql
SHOW TABLES [FROM db_name] [LIKE 'patter'|WHERE expr];
```

#### 查看数据库表结构

```sql
SHOW COLUMNS FROM table_name;

DESC table_name;

DESCRIBE table_name; 

EXPLAIN table_name; 

SHOW COLUMNS FROM table_name [LIKE 'PATTERN']

SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

mysql> show columns from innodb1;
+-------------+-------------+------+-----+---------+----------------+
| Field       | Type        | Null | Key | Default | Extra          |
+-------------+-------------+------+-----+---------+----------------+
| id          | int         | NO   | PRI | NULL    | auto_increment |
| first_name  | varchar(16) | YES  | MUL | NULL    |                |
| last_name   | varchar(16) | YES  |     | NULL    |                |
| id_card     | varchar(18) | YES  | UNI | NULL    |                |
| information | text        | YES  | MUL | NULL    |                |
+-------------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)

mysql> desc innodb1;
+-------------+-------------+------+-----+---------+----------------+
| Field       | Type        | Null | Key | Default | Extra          |
+-------------+-------------+------+-----+---------+----------------+
| id          | int         | NO   | PRI | NULL    | auto_increment |
| first_name  | varchar(16) | YES  | MUL | NULL    |                |
| last_name   | varchar(16) | YES  |     | NULL    |                |
| id_card     | varchar(18) | YES  | UNI | NULL    |                |
| information | text        | YES  | MUL | NULL    |                |
+-------------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

#### 编辑数据表的默认存储引擎

* 修改MySQL配置文件

  ```sql
  default-storage-engine=INNODB
  ```

#### 添加字段

```sql
-- 添加单列
-- FIRST 添加在首位
-- AFTER 添加在末位
ALTER TABLE table_name 
	ADD [COLUMN] column_name column_definition [FIRST|AFTER column_name];

-- 添加多列
ALTER TABLE table_name 
	ADD [COLUMN] (column_name column_defintion,......);
```

#### 删除字段

```sql
ALTER TABLE table_name 
	DROP [COLUMN] column_name;
```

#### 添加/删除约束

```sql
-- 添加主键约束
-- symbol 约束名称
ALTER TABLE table_name 
	ADD [CONSTRAINT] [symbol] PRIMARY KEY [index_name][index_type] (index_column_name,....);

-- 添加唯一约束
ALTER TABLE table_name 
	ADD [CONSTRAINT] [symbol] UNIQUE [INDEX|KEY] [index_name] [index_type] (index_column_name,....);

-- 添加外键约束
ALTER TABLE table_name 
	ADD [CONSTRAINT] [symbol] FOREIGN KEY [index_name] [index_type] (index_column_name,....) REFERENCE table_name(column);

-- 添加/删除默认约束
-- literal 默认值
ALTER TABLE table_name
	ALTER [COLUMN] column_name {SET DEFAULT literal|DROP DEFAULT};

-- 删除主键约束
ALTER TABLE table_name 
	DROP PRIMARY KEY;

-- 删除唯一约束
ALTER TABLE table_name 
	DROP {INDEX|KEY} index_name;

-- 删除外键约束
-- fk_symbol 约束名称
ALTER TABLE table_name 
	DROP FOREIGN KEY fk_symbol;
```

#### 修改列定义

```sql
ALTER TABLE table_name
	MODIFY [COLUMN] column_name column_definiton [FIRST|AFTER column_name];
ALTER TABLE table_name
	CHANGE [COLUMN] old_column_name new_col_name column_definition [FIRST|AFTER column_name];
```

#### 数据表名称更改

```sql
ALTER TABLE table_name RENAME [TO|AS] new_table_name;
-- 批量修改
RENAME TABLE table_name TO new_table_name[,table_name2 TO new_table_name2]
RENAME TABLE 原表名 TO 库名.表名 （可将表移动到另一个数据库）
```

#### 插入数据

```sql
-- 自增字段可以使NULL/DEFAULT设置
INSERT [INTO] table_name [(columns_name,....)] {VALUES|VALUE} ({expr|DEFAULT},....),[({expr|DEFAULT},....)];

-- INSERT/SET语句
INSERT [INTO] table_name SET column_name={expr|DEFAULT},....;

-- INSERT/SELECT语句 此方法将查询结果插入到指定数据表中
INSERT [INTO] table_name [column_name,....] SELECT ...
```

#### 更新数据

```sql
-- 单表更新
-- 不加WHERE即对整个表进行操作
UPDATE [LOW_PRIORITY][IGNORE] table_name SET column_name ={expr|DEFAULT}[,column_name2 ={expr|DEFAULT}..] 
[WHERE where_condition]

-- 多表更新
UPDATE [LOW_PRIORITY][IGNORE] table_name SET column_name ={expr|DEFAULT}[,column_name2 ={expr|DEFAULT}..] 
[WHERE where_condition]
{[INNER|CROSS] JOIN | {LEFT|RIGHT} {OUTER} JOIN}
table_reference ON conditional_expr


select语句获得的数据可以用insert插入。
可以省略对列的指定，要求 values () 括号内，提供给了按照列顺序出现的所有字段的值。
    或者使用set语法。
    INSERT INTO tbl_name SET field=value,...；
可以一次性使用多个值，采用(), (), ();的形式。
    INSERT INTO tbl_name VALUES (), (), ();
可以在列值指定时，使用表达式。
    INSERT INTO tbl_name VALUES (field_value, 10+10, now());
可以使用一个特殊值 DEFAULT，表示该列使用默认值。
    INSERT INTO tbl_name VALUES (field_value, DEFAULT);
可以通过一个查询的结果，作为需要插入的值。
    INSERT INTO tbl_name SELECT ...;
可以指定在插入的值出现主键（或唯一索引）冲突时，更新其他非主键列的信息。
    INSERT INTO tbl_name VALUES/SET/SELECT ON DUPLICATE KEY UPDATE 字段=值, …;
```

#### 删除记录

```sql
DELETE FROM table_name 
[WHERE where_condition]
-- 多表删除
DELETE FROM table_name[.*],[,table_name[.*]] 
[WHERE where_condition]

DELETE FROM tbl_name [WHERE where_definition] [ORDER BY ...] [LIMIT row_count]
-- 按照条件删除。where
-- 指定删除的最多记录数。limit
-- 可以通过排序条件删除。order by + limit
-- 支持多表删除，使用类似连接语法。
delete from 需要删除数据多表1，表2 using 表连接操作 条件。
```

#### 清空表数据

```sql
TRUNCATE [TABLE] 表名
```

* **清空数据(DELETE)和删除重建表区别**
  * truncate 是删除表再创建，delete 是逐条删除
  * truncate 重置auto_increment的值。而delete不会
  * truncate 不知道删除了几条，而delete知道。
  * 当被用于带分区的表时，truncate 会保留分区

#### 复制表结构

```sql
-- 复制表结构
CREATE TABLE 表名 LIKE 要复制的表名
-- 复制表结构和数据
CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名
```

#### 表维护

```sql
-- 检查表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] ...
-- 优化表
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
-- 修复表
REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]
-- 分析表
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

### 字符集编码

```sql
-- MySQL、数据库、表、字段均可设置编码
-- 数据编码与客户端编码不需一致
SHOW VARIABLES LIKE 'character_set_%'   -- 查看所有字符集编码项
    character_set_client        客户端向服务器发送数据时使用的编码
    character_set_results       服务器端将结果返回给客户端所使用的编码
    character_set_connection    连接层编码
SET 变量名 = 变量值
    SET character_set_client = gbk;
    SET character_set_results = gbk;
    SET character_set_connection = gbk;
SET NAMES GBK;  -- 相当于完成以上三个设置
-- 校对集
    校对集用以排序
    SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']   查看所有字符集
    SHOW COLLATION [LIKE 'pattern']     查看所有校对集
    CHARSET 字符集编码     设置字符集编码
    COLLATE 校对集编码     设置校对集编码
```

### 查询

```sql
-- DISTINCT 需要放到所有列名的前面
-- DISTINCT 是对所有列名的组合进行去重
SELECT [DISTINCT] select_expr[,select_expr.....]
[
	FROM table_name
    [JOIN [right_table] ON [join_condition]]
	[WHERE where_condition]
    -- ASC 升序(默认)
    -- DESC 降序
	[GROUP BY {column_name|position} [ASC|DESC],...]
	[HAVING having_coditon]
  -- ORDER BY可以使用非选择列进行排序（即SELECT后面没有这个列名）
	[ORDER BY {column_name|expr|position} [ASC|DESC],...]
    -- offset 从0开始
	[LIMIT {[offset,] row_count|`row_count OFFSET offset`}]
]

SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [合计函数] -> HAVING -> ORDER BY -> LIMIT
a. select_expr
    -- 可以用 * 表示所有字段。
        select * from tb;
    -- 可以使用表达式（计算公式、函数调用、字段也是个表达式）
        select stu, 29+25, now() from tb;
    -- 可以为每个列使用别名。适用于简化列标识，避免多个列标识符重复。
        - 使用 as 关键字，也可省略 as.
        select stu+10 as add10 from tb;
b. FROM 子句
    用于标识查询来源。
    -- 可以为表起别名。使用as关键字。
        SELECT * FROM tb1 AS tt, tb2 AS bb;
    -- from子句后，可以同时出现多个表。
        -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
        SELECT * FROM tb1, tb2;
    -- 向优化符提示如何选择索引
        USE INDEX、IGNORE INDEX、FORCE INDEX
        SELECT * FROM table1 USE INDEX (key1,key2) WHERE key1=1 AND key2=2 AND key3=3;
        SELECT * FROM table1 IGNORE INDEX (key3) WHERE key1=1 AND key2=2 AND key3=3;
c. WHERE 子句
    -- 从from获得的数据源中进行筛选。
    -- 整型1表示真，0表示假。
    -- 表达式由运算符和运算数组成。
        -- 运算数：变量（字段）、值、函数返回值
        -- 运算符：
            =, <=>, <>, !=, <=, <, >=, >, !, &&, ||,
            in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
            is/is not 加上ture/false/unknown，检验某个值的真假
            <=>与<>功能相同，<=>可用于null比较
d. GROUP BY 子句, 分组子句
    GROUP BY 字段/别名 [排序方式]
    分组后会进行排序。升序：ASC，降序：DESC
    以下[合计函数]需配合 GROUP BY 使用：
    count 返回不同的非NULL值数目  count(*)、count(字段)
    sum 求和
    max 求最大值
    min 求最小值
    avg 求平均值
    group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接。
e. HAVING 子句，条件子句
    与 where 功能、用法相同，执行时机不同。
    where 在开始时执行检测数据，对原数据进行过滤。
    having 对筛选出的结果再次进行过滤。
    having 字段必须是查询出来的，where 字段必须是数据表存在的。
    where 不可以使用字段的别名，having 可以。因为执行WHERE代码时，可能尚未确定列值。
    where 不可以使用合计函数。一般需用合计函数才会用 having
    SQL标准要求HAVING必须引用GROUP BY子句中的列或用于合计函数中的列。
f. ORDER BY 子句，排序子句
    order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
    升序：ASC，降序：DESC
    支持多个字段的排序。
g. LIMIT 子句，限制结果数量子句
    仅对处理好的结果进行数量限制。将处理好的结果的看作是一个集合，按照记录出现的顺序，索引从0开始。
    limit 起始位置, 获取条数
    省略第一个参数，表示从索引0开始。limit 获取条数
h. DISTINCT, ALL 选项
    distinct 去除重复记录
    默认为 all, 全部记录
```

<img src="https://www.holelin.cn/img/mysql/MySQL%E5%AD%90%E5%8F%A5%E8%A7%A3%E6%9E%90%E9%A1%BA%E5%BA%8F.png" alt="img" style="zoom:80%;" />

* **ORDER BY子句**

  * 排序的列名: ORDER BY后面可以有一个或者多个列名,多个列排序会逐次进行排序;
  * 排序的顺序:  ASC(递增排序) DESC(递减排序) 默认ASC递增排序;
  * 非选择列排序: ORDER BY可以使用非选择列进行排序,即SELECT 后面没有这个列名,同样可以使用ORDER BY排序;
  * ORDER BY的位置: ORDER BY 子句通常位于SELECT语句的最后一条子句,否则会报错;

* **LIMIT子句**

  ```sql
  SELECT * 
  FROM table
  WHERE  condition1 = 0 AND condition2 = 2 AND condition3 = 3
  ORDER BY id DESC
  LIMIT 2000 OFFSET 50000
  ```

  * LIMIT子句可以被用于强制SELECT语句返回指定的记录数,LIMIT接受一个或两个数字参数,参数必须是一个整数常量,如果给定两个参数,第一个参数指定第一个返回记录行的偏移量,第二个参数指定返回记录行的最大数目,初始记录行的偏移量是0而不是1,为了与`PostgreSQL`兼容MySQL也支持`LIMIT # OFFSET #`

    ```sql
    -- 查询记录行6-15 即从6开始往后查10条记录
    SELECT * FROM table LIMIT 5,10; 
    
    -- 查询记录行95~last,从95开始查到最后
    SELECT * FROM table LIMIT 95,-1;
    
    SELECT * FROM table LIMIT 5; -- 等价于 SELECT * FROM table LIMIT 0,5;
    ```

  * LIMIT的作用: 分页查询

    * 最基本的分页方式: `SELECT ...FROM...WHERE...ORDER BY ...LIMIT....`,适用与数据量较小的场景

      ```sql
      SELECT * 
      FROM table 
      WHERE c_id = 123
      ORDER BY id 
      LIMIT 50,10;
      ```

    * 子查询的分页查询:

      ```sql
      SELECT * 
      FROM table 
      WHERE id > (
      	SELECT id 
      	FROM table 
      	WHER c_id =123
      	ORDER BY id
      	LIMIT 100000,1
      )
      LIMIT 10;
      ```

    * JOIN分页方式

      ```sql
      SELECT * 
      FROM table AS t1
      JOIN (
      	SELECT id
      	FROM table 
      	ORDER BY id DESC LIMIT ".($page-1)*$pahesize.",1
      ) AS t2
      WHERE t1,id<=t2.id
      ORDER BY id DESC
      LIMIT $pagesize;
      ```

#### 子查询

> 子查询指嵌套在查询内部且必须始终在圆括号内;
>
> 子查询可以包含多个关键字或条件,如DISTINCT,GROUP BY,ORDER BY ,函数等;
>
> 子查询的外层查询可以是:SELECT,INSERT,UPDATE,SET或DO
>
> 子查询的返回值可以是标量,一行,一列,或子查询

```sql
    - 子查询需用括号包裹。
-- from型
    from后要求是一个表，必须给子查询结果取个别名。
    - 简化每个查询内的条件。
    - from型需将结果生成一个临时表格，可用以原表的锁定的释放。
    - 子查询返回一个表，表型子查询。
    select * from (select * from tb where id>0) as subfrom where id>1;
-- where型
    - 子查询返回一个值，标量子查询。
    - 不需要给子查询取别名。
    - where子查询内的表，不能直接用以更新。
    select * from tb where money = (select max(money) from tb);
    -- 列子查询
        如果子查询结果返回的是一列。
        使用 in 或 not in 完成查询
        exists 和 not exists 条件
            如果子查询返回数据，则返回1或0。常用于判断条件。
            select column1 from t1 where exists (select * from t2);
    -- 行子查询
        查询条件是一个行。
        select * from t1 where (id, gender) in (select id, gender from t2);
        行构造符：(col1, col2, ...) 或 ROW(col1, col2, ...)
        行构造符通常用于与对能返回两个或两个以上列的子查询进行比较。
    -- 特殊运算符
    != all()    相当于 not in
    = some()    相当于 in。any 是 some 的别名
    != some()   不等同于 not in，不等于其中某一个。
    all, some 可以配合其他运算符一起使用。
```

##### 使用比较运算符的子查询

> =,>,<,>=,<=,<>,!=,<=>

* 语法 `operand comparison_operator (subQuery)`

|       | ANY    | SOME   | ALL    |
| ----- | ------ | ------ | ------ |
| >,>=  | 最小值 | 最小值 | 最大值 |
| <,<=  | 最大值 | 最大值 | 最小值 |
| =     | 任意值 | 任意值 |        |
| <>,!= |        |        | 任意值 |

* 使用`[NOT] IN`的子查询
  * 语法: `operand comparison_operator [not] IN (subQuery)` 
  * `= ANY` 运算符与`IN`等效
  * `!=ALL`运算符与`NOT IN`等效
* 使用`[NOT] EXISTS`子查询
  * 如果子查询返回任何行,EXISTS将会返回TRUE;否则为FALSE;

#### 联合查询UNION

```sql
-- 将多个select查询的结果组合成一个结果集合。
SELECT ... UNION [ALL|DISTINCT] SELECT ...
-- 默认 DISTINCT 方式，即所有返回的行都是唯一的
-- 建议，对每个SELECT查询加上小括号包裹。
ORDER BY 排序时，需加上 LIMIT 进行结合。
-- 需要各select查询的字段数量一样。
-- 每个select查询的字段列表(数量、类型)应一致，因为结果中的字段名以第一条select语句为准。
```

### 索引

#### 创建索引

* 在创建表的时候对字段进行指定索引

  ```sql
  CREATE TABLE user_index (
  	id INT auto_increment PRIMARY KEY,
  	first_name VARCHAR ( 16 ),
  	last_name VARCHAR ( 16 ),
  	uuid VARCHAR ( 20 ) UNIQUE,
  	id_card VARCHAR ( 18 ),
  	information text 
  );
  ```

* 通过`alter table 表名 add 索引`更改表结构来添加索引

  ```sql
  -- 创建一个first_name和last_name的复合索引，并命名为name
  alter table user_index add key name (first_name,last_name);
  -- 创建一个id_card的唯一索引，默认以字段名作为索引名
  alter table user_index add UNIQUE KEY (id_card);
  -- 鸡肋，全文索引不支持中文
  alter table user_index add FULLTEXT KEY (information);
  ```

#### 查看索引

* 通过`desc 表名`展示表结构来查看索引信息

  ```sql
  mysql> desc user_index;
  +-------------+-------------+------+-----+---------+----------------+
  | Field       | Type        | Null | Key | Default | Extra          |
  +-------------+-------------+------+-----+---------+----------------+
  | id          | int         | NO   | PRI | NULL    | auto_increment |
  | first_name  | varchar(16) | YES  | MUL | NULL    |                |
  | last_name   | varchar(16) | YES  |     | NULL    |                |
  | uuid        | varchar(20) | YES  | UNI | NULL    |                |
  | id_card     | varchar(18) | YES  | UNI | NULL    |                |
  | information | text        | YES  | MUL | NULL    |                |
  +-------------+-------------+------+-----+---------+----------------+
  6 rows in set (0.07 sec)
  ```

*  通过`show create table 表名`展示建表语句来查看索引信息

  ```sql
  mysql> show create table user_index;
  +------------+--------------------------------------------------------------------------------------------------------------+
  | Table      | Create Table                                                                                                 |
  +------------+--------------------------------------------------------------------------------------------------------------+
  | user_index | CREATE TABLE `user_index` (
    `id` int NOT NULL AUTO_INCREMENT,
    `first_name` varchar(16) DEFAULT NULL,
    `last_name` varchar(16) DEFAULT NULL,
    `uuid` varchar(20) DEFAULT NULL,
    `id_card` varchar(18) DEFAULT NULL,
    `information` text,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uuid` (`uuid`),
    UNIQUE KEY `id_card` (`id_card`),
    KEY `name` (`first_name`,`last_name`),
    FULLTEXT KEY `information` (`information`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
  +------------+--------------------------------------------------------------------------------------------------------------+
  1 row in set (0.08 sec)
  ```
  
* 通过`SHOW INDEXS FROM table_name\G;`

#### 删除索引

* 通过`alter table 表名 drop key 索引名称`

  ```sql
  alter table user_index drop KEY name;
  alter table user_index drop KEY id_card;
  alter table user_index drop KEY information;
  ```

* 删除主键索引：`alter table 表名 drop primary key`（因为主键只有一个）。这里值得注意的是，如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引）

  ```sql
  mysql> alter table user_index drop primary key;
  1075 - Incorrect table definition; there can be only one auto column and it must be defined as a key
  ```

  * 若一定需要删除,则需要取消自增长在进行删除

  ```sql
  ALTER TABLE user_index MODIFY id INT;
  ALTER TABLE user_index DROP PRIMARY KEY;
  ```

### 导出

```sql
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

### 备份与还原

```sql
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

### 视图

```sql
什么是视图：
    视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
    视图具有表结构文件，但不存在数据文件。
    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。
    视图是存储在数据库中的查询的sql语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。
-- 创建视图
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
    - 视图名必须唯一，同时不能与表重名。
    - 视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    - 可以指定视图执行的算法，通过ALGORITHM指定。
    - column_list如果存在，则数目必须等于SELECT语句检索的列数
-- 查看结构
    SHOW CREATE VIEW view_name
-- 删除视图
    - 删除视图后，数据依然存在。
    - 可同时删除多个视图。
    DROP VIEW [IF EXISTS] view_name ...
-- 修改视图结构
    - 一般不修改视图，因为不是所有的更新视图都会映射到表上。
    ALTER VIEW view_name [(column_list)] AS select_statement
-- 视图作用
    1. 简化业务逻辑
    2. 对客户端隐藏真实的表结构
-- 视图算法(ALGORITHM)
    MERGE       合并
        将视图的查询语句，与外部查询需要先合并再执行！
    TEMPTABLE   临时表
        将视图执行完毕后，形成临时表，再做外层查询！
    UNDEFINED   未定义(默认)，指的是MySQL自主去选择相应的算法。
```

### 事务(transaction)

```sql
事务是指逻辑上的一组操作，组成这组操作的各个单元，要不全成功要不全失败。
    - 支持连续SQL的集体成功或集体撤销。
    - 事务是数据库在数据晚自习方面的一个功能。
    - 需要利用 InnoDB 或 BDB 存储引擎，对自动提交的特性支持完成。
    - InnoDB被称为事务安全型引擎。
-- 事务开启
    START TRANSACTION; 或者 BEGIN;
    开启事务后，所有被执行的SQL语句均被认作当前事务内的SQL语句。
-- 事务提交
    COMMIT;
-- 事务回滚
    ROLLBACK;
    如果部分操作发生问题，映射到事务开启前。
-- 事务的特性
    1. 原子性（Atomicity）
        事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
    2. 一致性（Consistency）
        事务前后数据的完整性必须保持一致。
        - 事务开始和结束时，外部数据一致
        - 在整个事务过程中，操作是连续的
    3. 隔离性（Isolation）
        多个用户并发访问数据库时，一个用户的事务不能被其它用户的事物所干扰，多个并发事务之间的数据要相互隔离。
    4. 持久性（Durability）
        一个事务一旦被提交，它对数据库中的数据改变就是永久性的。
-- 事务的实现
    1. 要求是事务支持的表类型
    2. 执行一组相关的操作前开启事务
    3. 整组操作完成后，都成功，则提交；如果存在失败，选择回滚，则会回到事务开始的备份点。
-- 事务的原理
    利用InnoDB的自动提交(autocommit)特性完成。
    普通的MySQL执行语句后，当前的数据提交操作均可被其他客户端可见。
    而事务是暂时关闭“自动提交”机制，需要commit提交持久化数据操作。
-- 注意
    1. 数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
    2. 事务不能被嵌套
-- 保存点
    SAVEPOINT 保存点名称 -- 设置一个事务保存点
    ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
    RELEASE SAVEPOINT 保存点名称 -- 删除保存点
-- InnoDB自动提交特性设置
    SET autocommit = 0|1;   0表示关闭自动提交，1表示开启自动提交。
    - 如果关闭了，那普通操作的结果对其他客户端也不可见，需要commit提交后才能持久化数据操作。
    - 也可以关闭自动提交来开启事务。但与START TRANSACTION不同的是，
        SET autocommit是永久改变服务器的设置，直到下次再次修改该设置。(针对当前连接)
        而START TRANSACTION记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(针对当前事务)
```

### 锁表

```sql
表锁定只用于防止其它客户端进行不正当地读取和写入
MyISAM 支持表锁，InnoDB 支持行锁
-- 锁定
    LOCK TABLES tbl_name [AS alias] [READ|WRITE]
-- 解锁
    UNLOCK TABLES
```

### 触发器

```sql
    触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象
    监听：记录的增加、修改、删除。
-- 创建触发器
CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
    参数：
    trigger_time是触发程序的动作时间。它可以是 before 或 after，以指明触发程序是在激活它的语句之前或之后触发。
    trigger_event指明了激活触发程序的语句的类型
        INSERT：将新行插入表时激活触发程序
        UPDATE：更改某一行时激活触发程序
        DELETE：从表中删除某一行时激活触发程序
    tbl_name：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
    trigger_stmt：当触发程序激活时执行的语句。执行多个语句，可使用BEGIN...END复合语句结构
-- 删除
DROP TRIGGER [schema_name.]trigger_name
可以使用old和new代替旧的和新的数据
    更新操作，更新前是old，更新后是new.
    删除操作，只有old.
    增加操作，只有new.
-- 注意
    1. 对于具有相同触发程序动作时间和事件的给定表，不能有两个触发程序。
```

### SQL编程

```sql
-- 修改最外层语句结束符
delimiter 自定义结束符号
    SQL语句
自定义结束符号
delimiter ;     -- 修改回原来的分号

-- 语句块包裹
begin
    语句块
end

-- 特殊的执行
1. 只要添加记录，就会触发程序。
2. Insert into on duplicate key update 语法会触发：
    如果没有重复记录，会触发 before insert, after insert;
    如果有重复记录并更新，会触发 before insert, before update, after update;
    如果有重复记录但是没有发生更新，则触发 before insert, before update
3. Replace 语法 如果有记录，则执行 before insert, before delete, after delete, after insert
```

```sql
--// 局部变量 ----------
-- 变量声明
    declare var_name[,...] type [default value]
    这个语句被用来声明局部变量。要给变量提供一个默认值，请包含一个default子句。值可以被指定为一个表达式，不需要为一个常数。如果没有default子句，初始值为null。
-- 赋值
    使用 set 和 select into 语句为变量赋值。
    - 注意：在函数内是可以使用全局变量（用户自定义的变量）


--// 全局变量 ----------
-- 定义、赋值
set 语句可以定义并为变量赋值。
set @var = value;
也可以使用select into语句为变量初始化并赋值。这样要求select语句只能返回一行，但是可以是多个字段，就意味着同时为多个变量进行赋值，变量的数量需要与查询的列数一致。
还可以把赋值语句看作一个表达式，通过select执行完成。此时为了避免=被当作关系运算符看待，使用:=代替。（set语句可以使用= 和 :=）。
select @var:=20;
select @v1:=id, @v2=name from t1 limit 1;
select * from tbl_name where @var:=30;
select into 可以将表中查询获得的数据赋给变量。
    -| select max(height) into @max_height from tb;
-- 自定义变量名
为了避免select语句中，用户自定义的变量与系统标识符（通常是字段名）冲突，用户自定义变量在变量名前使用@作为开始符号。
@var=10;
    - 变量被定义后，在整个会话周期都有效（登录到退出）



--// 控制结构 ----------
-- if语句
if search_condition then
    statement_list   
[elseif search_condition then
    statement_list]
...
[else
    statement_list]
end if;

-- case语句
CASE value WHEN [compare-value] THEN result
[WHEN [compare-value] THEN result ...]
[ELSE result]
END

-- while循环
[begin_label:] while search_condition do
    statement_list
end while [end_label];
- 如果需要在循环内提前终止 while循环，则需要使用标签；标签需要成对出现。
    -- 退出循环
        退出整个循环 leave
        退出当前循环 iterate
        通过退出的标签决定退出哪个循环

--// 存储函数，自定义函数 ----------
-- 新建
    CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
        函数体
    - 函数名，应该合法的标识符，并且不应该与已有的关键字冲突。
    - 一个函数应该属于某个数据库，可以使用db_name.funciton_name的形式执行当前函数所属数据库，否则为当前数据库。
    - 参数部分，由"参数名"和"参数类型"组成。多个参数用逗号隔开。
    - 函数体由多条可用的mysql语句，流程控制，变量声明等语句构成。
    - 多条语句应该使用 begin...end 语句块包含。
    - 一定要有 return 返回值语句。
-- 删除
    DROP FUNCTION [IF EXISTS] function_name;
-- 查看
    SHOW FUNCTION STATUS LIKE 'partten'
    SHOW CREATE FUNCTION function_name;
-- 修改
    ALTER FUNCTION function_name 函数选项

--// 存储过程，自定义功能 ----------
-- 定义
存储存储过程 是一段代码（过程），存储在数据库中的sql组成。
一个存储过程通常用于完成一段业务逻辑，例如报名，交班费，订单入库等。
而一个函数通常专注与某个功能，视为其他程序服务的，需要在其他语句中调用函数才可以，而存储过程不能被其他调用，是自己执行 通过call执行。
-- 创建
CREATE PROCEDURE sp_name (参数列表)
    过程体
参数列表：不同于函数的参数列表，需要指明参数类型
IN，表示输入型
OUT，表示输出型
INOUT，表示混合型
注意，没有返回值。


/* 存储过程 */ ------------------
存储过程是一段可执行性代码的集合。相比函数，更偏向于业务逻辑。
调用：CALL 过程名
-- 注意
- 没有返回值。
- 只能单独调用，不可夹杂在其他语句中
-- 参数
IN|OUT|INOUT 参数名 数据类型
IN      输入：在调用过程中，将数据输入到过程体内部的参数
OUT     输出：在调用过程中，将过程体处理完的结果返回到客户端
INOUT   输入输出：既可输入，也可输出
-- 语法
CREATE PROCEDURE 过程名 (参数列表)
BEGIN
    过程体
END
```

### 用户和权限管理

```sql
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



