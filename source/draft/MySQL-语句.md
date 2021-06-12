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

### 操作数据库

### 操作表

### 约束

#### 索引

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

