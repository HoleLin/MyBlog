---
title: MySQL(一)-基础操作
date: 2021-09-03 15:32:31
cover: /img/cover/MySQL.jpg
tags:
- 操作
- 数据类型
- 连接
- 约束
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

#### 操作MySQL客户端
##### MySQL登录:`mysql 参数`

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| -D,--database = name | 打开指定的数据库                                             |
| --delimiter = name   | 指定分割符                                                   |
| -h,--host = name     | 服务器名称                                                   |
| -p,--port = #        | 端口号                                                       |
| --prompt = name      | 设置提示符,[\D:完整的日期;\d: 当前数据库;\h:服务器名称;\u: 当前用户] |
| -u,--user = name     | 用户名                                                       |
| -V,--version         | 输出版本信息并且推出                                         |

##### MySQL退出: `exit/quit/\q`

##### 显示数据库信息:`\s`

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

##### 查看MySQL默认读取`my.cnf`的目录

* 若没有设置使用指定目录的`my.cnf`,MySQL启动是会读取安装目录及默认目录下的`my.conf`,查看MySQL默认读取`my.conf的目录

  ```sh
  [root@holelin docs]# mysql --help |grep 'my.cnf'
                        order of preference, my.cnf, $MYSQL_TCP_PORT,
  /etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
  ```

#### 操作数据库

##### 显示所有数据库

```sql
SHOW DATABASES;
```

##### 选择(打开)数据库:

```sql
USE db_name;
```

##### 创建数据库

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

##### 查看当前服务器下的数据库列表

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

##### 查看警告信息

```sql
SHOW WARNINGS;
```

##### 查看数据库创建时使用的指令信息

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

##### 修改数据库编码方式

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

##### 删除数据库

```sql
DROP {DATABASE|SCHEMA} [IF EXISTS] db_name;
```

##### 显示当前数据库

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

#### 操作数据表

##### 创建表

```mysql
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

##### 查看数据库中数据表列表

```sql
SHOW TABLES [FROM db_name] [LIKE 'patter'|WHERE expr];
```

##### 查看数据库表结构

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

##### 编辑数据表的默认存储引擎

* 修改MySQL配置文件

  ```sql
  default-storage-engine=INNODB
  ```

##### 添加字段

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

##### 删除字段

```sql
ALTER TABLE table_name 
	DROP [COLUMN] column_name;
```

##### 添加/删除约束

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

##### 修改列定义

```sql
ALTER TABLE table_name
	MODIFY [COLUMN] column_name column_definiton [FIRST|AFTER column_name];
ALTER TABLE table_name
	CHANGE [COLUMN] old_column_name new_col_name column_definition [FIRST|AFTER column_name];
```

##### 数据表名称更改

```sql
ALTER TABLE table_name RENAME [TO|AS] new_table_name;
-- 批量修改
RENAME TABLE table_name TO new_table_name[,table_name2 TO new_table_name2]
RENAME TABLE 原表名 TO 库名.表名 （可将表移动到另一个数据库）
```

##### 插入数据

```sql
-- 自增字段可以使NULL/DEFAULT设置
INSERT [INTO] table_name [(columns_name,....)] {VALUES|VALUE} ({expr|DEFAULT},....),[({expr|DEFAULT},....)];

-- INSERT/SET语句
INSERT [INTO] table_name SET column_name={expr|DEFAULT},....;

-- INSERT/SELECT语句 此方法将查询结果插入到指定数据表中
INSERT [INTO] table_name [column_name,....] SELECT ...
```

##### 更新数据

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

##### 删除记录

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

##### 清空表数据

```sql
TRUNCATE [TABLE] 表名
```

* **清空数据(DELETE)和删除重建表区别**
  * `truncate `是删除表再创建，`delete` 是逐条删除
  * `truncate `重置`auto_increment`的值。而delete不会
  * `truncate` 不知道删除了几条，而`delete`知道。
  * 当被用于带分区的表时，`truncate` 会保留分区

##### 复制表结构

```sql
-- 复制表结构
CREATE TABLE 表名 LIKE 要复制的表名
-- 复制表结构和数据
CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名
```

##### 表维护

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

#### 查询

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
-- 使用 as 关键字，也可省略 as.
select stu+10 as add10 from tb;

b. FROM 子句
-- 用于标识查询来源。
-- 可以为表起别名。使用as关键字。
    SELECT * FROM tb1 AS tt, tb2 AS bb;
-- from子句后，可以同时出现多个表。
    -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
    SELECT * FROM tb1, tb2;
-- 向优化符提示如何选择索引
USE INDEX IGNORE INDEX、FORCE INDEX
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

##### 子查询

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

##### 联合查询UNION

```sql
-- 将多个select查询的结果组合成一个结果集合。
SELECT ... UNION [ALL|DISTINCT] SELECT ...
-- 默认 DISTINCT 方式，即所有返回的行都是唯一的
-- 建议，对每个SELECT查询加上小括号包裹。
ORDER BY 排序时，需加上 LIMIT 进行结合。
-- 需要各select查询的字段数量一样。
-- 每个select查询的字段列表(数量、类型)应一致，因为结果中的字段名以第一条select语句为准。
```

### 数据类型

* 整型
* 浮点型
* 日期时间型
* 字符型

#### 整型

| 数据类型    | 存储范围                                                     | 对应Java     | 字节(byte) | 位(bit) |
| ----------- | ------------------------------------------------------------ | ------------ | ---------- | ------- |
| `TINYINT`   | 有符号值:-128~127(-2<sup>7</sup>~2<sup>7</sup>-1)<br />无符号值:0~255(0~2<sup>8</sup>-1) | byte/boolean | 1          | 8       |
| `SMALLINT`  | 有符号值:-32768~32767(-2<sup>15</sup>~2<sup>15</sup>-1)<br />无符号值:0~65533(0~2<sup>16</sup>-1) | short        | 2          | 16      |
| `MEDIUMINT` | 有符号值:-2<sup>23</sup>~2<sup>23</sup>-1<br />无符号值:0~2<sup>24</sup>-1 |              | 3          | 24      |
| `INT`       | 有符号值:-2<sup>31</sup>~2<sup>31</sup>-1<br />无符号值: 0~2<sup>32</sup>-1 | int          | 4          | 32      |
| `BIGINT`    | 有符号值:-2<sup>63</sup>~2<sup>63</sup>-1<br />无符号值: 0~2<sup>64</sup>-1 | long         | 8          | 64      |

#### 浮点型

| 数据类型        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `FLOAT[(M,D)]`  | M是数字总位数,D是小数点后面的位数,如果M和D被省略,根据硬件允许的限制来保存值,单精度浮点数精确到大约7位小数位 |
| `DOUBLE[(M,D)]` | M是数字总位数,D是小数点后面的位数                            |

#### 定点数

| 数据类型       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| DECIMAL[(M,D)] | M表示总位数,D表示小数位数,保存一个精确的数值,不会发生数据的改变,不同于浮点数的四舍五入,将浮点数转换为字符串来保存,每9位数字保存4个字节 |

#### 日期时间型

| 数据类型    | 描述      | 范围                                       | 支持格式                                                     | 字节 | 位   |
| ----------- | --------- | ------------------------------------------ | ------------------------------------------------------------ | ---- | ---- |
| `YEAR`      | 年        | 1901 - 2155                                | `YYYY`<br />`YY`                                             | 1    | 8    |
| `TIME`      | 时间      | -838:59:59 到 838:59:59                    | `hh:mm:ss`<br />`hhmmss`                                     | 3    | 24   |
| `DATE`      | 日期      | 1000-01-01 到 9999-12-31                   | `YYYY-MM-DD`<br />`YY-MM-DD`<br />`YYYYMMDD`<br />`YYMMDD`<br />`YYYYMMDD`<br />`YYMMDD` | 3    | 24   |
| `DATETIME`  | 日期+时间 | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 | ` YYYY-MM-DD hh:mm:ss`                                       | 8    | 64   |
| `TIMESTAMP` | 时间戳    | 19700101000000 到 2038-01-19 03:14:07      | ` YY-MM-DD hh:mm:ss`<br />` YYYYMMDDhhmmss`<br />`YYMMDDhhmmss`<br />`YYYYMMDDhhmmss`<br />`YYMMDDhhmmss` | 4    | 32   |

![img](https://www.holelin.cn/img/tools/%E6%97%B6%E9%97%B4%E6%A0%BC%E5%BC%8F/%E6%97%B6%E9%97%B4%E6%A0%BC%E5%BC%8F1.png)

![img](https://www.holelin.cn/img/tools/%E6%97%B6%E9%97%B4%E6%A0%BC%E5%BC%8F/%E6%97%B6%E9%97%B4%E6%A0%BC%E5%BC%8F2.png)

#### 字符型

| 数据类型                      | 存储需求                                             | 说明                                                         |
| ----------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| `CHAR(M)`                     | M个字节,0<=M<=255                                    | 定长字符串,速度快,但浪费空间.最多255个字符,与编码无关        |
| `VARHCHAR(M)`                 | L+1个字节,其中L<=M且0<=M<=65535                      | 变长字符串,速度慢,但节省空间.最多65535个字符,与编码有关<br />`utf8`最大为21844个字符<br />`gbk`最大为32766个字符<br />`latin1`最大为65532个字符 |
| `TINYTEXT`                    | L+1个字节,其中L<2<sup>8</sup>                        |                                                              |
| `TEXT`                        | L+2个字节,其中L<2<sup>16</sup>                       | 非二进制字符串(字符字符串)                                   |
| `MEDIUMTEXT`                  | L+3个字节,其中L<2<sup>24</sup>                       |                                                              |
| `LONGTEXT`                    | L+4个字节,其中L<2<sup>32</sup>                       |                                                              |
| `ENUM('value1','value2',...)` | 1或2个字节,取决于枚举值的个数(最多65535个)           |                                                              |
| `SET('value1','value2',...)`  | 1,2,3,4或者8个字节,取决于set成员的数目(最多64个成员) |                                                              |
| `BLOB`                        |                                                      | 二进制字符串(字节字符串)                                     |
| `TINYBLOB`                    |                                                      |                                                              |
| `MEDIUMBLOB`                  |                                                      |                                                              |
| `LONGBLOB`                    |                                                      |                                                              |

### 连接(JOIN)

<img src="https://www.holelin.cn/img/mysql/join.png" alt="img" style="zoom: 67%;" />

```sql
 将多个表的字段进行连接，可以指定连接条件。
-- 内连接(inner join)
    - 默认就是内连接，可省略inner。
    - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    -- Using连接
    还有 using, 但需字段名相同。using(字段名)
    -- 交叉连接 cross join
        即，没有条件的内连接。
        select * from tb1 cross join tb2;
-- 外连接(outer join)
    - 如果数据不存在，也会出现在连接结果中。
    -- 左外连接 left join
        如果数据不存在，左表记录会出现，而右表为null填充
    -- 右外连接 right join
        如果数据不存在，右表记录会出现，而左表为null填充
-- 自然连接(natural join)
    自动判断连接条件完成连接。
    相当于省略了using，会自动查找相同字段名。
    natural join
    natural left join
    natural right join
    
select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;
```

#### 内连接(INNER)

> 在MySQL中JOIN,CROSS JOIN和INNER JOIN是等价的

* 仅显示符合连接条件的记录;

#### 左外连接

* 显示左表的全部记录以及右表中符合连接提条件的记录;

  ```sql
  SELECT * 
  FROM A
  LEFT JOIN B join_condition
  ```

  * 数据表B的结果集依赖于数据表A
  * 数据表A的结果集根据左连接条件依赖所有数据表(B表除外)
  * 左外连接条件决定如何检索数据表B(在没有指定WHERE条件的情况下)
  * 若数据表A的某条记录符合WHERE条件,但是在数据表B中不存在符合连接条件的记录,将生成一个所有列为NULL的额外行;

#### 右外连接

* 显示右表的全部记录以及左表中符合连接条件的记录;

### 约束(Constraint)

#### 约束的作用

> 保证数据的完整性和一致性;

#### 约束的分类

* 表级约束(对两个或两个以上字段进行约束);
* 列级约束

#### 主键约束(PRIMARY KEY)

* 每一张表只能存在一个主键(主键自带唯一性);
* 主键保证记录的唯一性;
* 主键自动为`NOT NULL`;
* **自动编码(AUTO_INCREMENT)**
  * 自动编码必须与主键组合使用;
  * 默认情况下,起始值为1,每次的增量为1;

#### 唯一性约束(UNIQUE KEY)

* 唯一约束可以保证记录的唯一性;
* 唯一约束的字段可以为空值(`NULL`)
* 每张表可以存着多个唯一约束

#### 默认约束(DEFAULT)

* 默认值
* 当插入记录时,如果没有明确为方法字段赋值免责自动赋予默认值

#### 非空约束(NOT NULL)

#### 外键约束(FOREIGN KEY)

* 保持数据一致性,完整性;

* 实现一对一或一对多关心;

###### 外键约束的要求

* 父表和子表不行使用相同的存储引擎,而且禁止使用临时表;
* 数据表存储引擎只能为`InnodDB`;
* 外键列和参照列必须具有相似的数据类型,其中数字的长度或是否有符号位必须相同,而字符的长度则可以不同;
* 外键列和参照列必须创建索引,如果外键列不存在索引的话,MySQL将会自动创建索引;

###### 外键约束的参照操作

* **CASCADE**: 从父表删除或更新行且自动删除或更新子表中匹配的行;
* **SET NULL**: 从父表删除或更新行,设置子表中的外键列为NULL,若使用该选项,必须保证子表列没有指定`NOT NULL`;
* **RESTRICT**: 拒绝对父表的删除或更新操作;
* **NOT ACTION**: 标准SQL关键字,在MySQL中**RESTRICT**相同;
