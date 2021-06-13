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
CREATE TABLE [IF NOT EXISTS] table_name(
	column_name data_type;
	....
);
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
ALTER TABLE table_name ADD [COLUMN] column_name column_definition [FIRST|AFTER column_name];

-- 添加多列
ALTER TABLE table_name ADD [COLUMN] (column_name column_defintion,......);

```

#### 删除字段

```sql
ALTER TABLE table_name DROP [COLUMN] column_name;
```

#### 添加/删除约束

```sql
-- 添加主键约束
-- symbol 约束名称
ALTER TABLE table_name ADD [CONSTRAINT] [symbol] PRIMARY KEY [index_name][index_type] (index_column_name,....);

-- 添加唯一约束
ALTER TABLE table_name ADD [CONSTRAINT] [symbol] UNIQUE [INDEX|KEY] [index_name] [index_type] (index_column_name,....);

-- 添加外键约束
ALTER TABLE table_name ADD [CONSTRAINT] [symbol] FOREIGN KEY [index_name] [index_type] (index_column_name,....) REFERENCE table_name(column);

-- 添加/删除默认约束
-- literal 默认值
ALTER TABLE table_name ALTER [COLUMN] column_name {SET DEFAULT literal|DROP DEFAULT};

-- 删除主键约束
ALTER TABLE table_name DROP PRIMARY KEY;

-- 删除唯一约束
ALTER TABLE table_name DROP {INDEX|KEY} index_name;

-- 删除外键约束
-- fk_symbol 约束名称
ALTER TABLE table_name DROP FOREIGN KEY fk_symbol;
```

#### 修改列定义

```sql
ALTER TABLE table_name MODIFY [COLUMN] column_name column_definiton [FIRST|AFTER column_name];
```

#### 修改列定义

```sql
ALTER TABLE table_name CHANGE [COLUMN] old_column_name new_col_name column_definition [FIRST|AFTER column_name];
```

#### 数据表名称更改

```sql
ALTER TABLE table_name RENAME [TO|AS] new_table_name;
-- 批量修改
RENAME TABLE table_name TO new_table_name[,table_name2 TO new_table_name2]
```

#### 插入数据

```sql
-- 自增字段可以使NULL/DEFAULT设置
INSERT [INTO] table_name [(columns_name,....)] {VALUES|VALUE} ({expr|DEFAULT},....) ;

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

```

#### 删除记录

```sql
DELETE FROM table_name 
[WHERE where_condition]
-- 多表删除
DELETE FROM table_name[.*],[,table_name[.*]] 
[WHERE where_condition]
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
	[ORDER BY {column_name|expr|position} [ASC|DESC],...]
    -- offset 从0开始
	[LIMIT {[offset,] row_count|`row_count OFFSET offset`}]
]
```

<img src="http://www.chenjunlin.vip/img/mysql/MySQL%E5%AD%90%E5%8F%A5%E8%A7%A3%E6%9E%90%E9%A1%BA%E5%BA%8F.png" alt="img" style="zoom:80%;" />

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

#### 索引分析(explain)

### 分区语法

