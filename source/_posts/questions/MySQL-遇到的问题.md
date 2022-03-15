---
title: MySQL-遇到的问题
date: 2021-07-19 15:42:07
cover: /img/cover/MySQL.jpg
tags:
- questions
categories:
- MySQL
updated: 2021-08-30 14:42:07
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

* [8.2.1.19 LIMIT Query Optimization](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html)

### 数据库大小敏感问题

* `Oracle`
  * 默认是**大小写不敏感**,表名,字段名等不区分大小写,小写字母会自动转换为大写字母;
  * 需要用小写字母时需要使用双引号或借助函数`lower()`;
* `PostgreSQL`
  * 默认是大小写不敏感,表名,字段名等不区分大小写,大写字母会自动转换为小写字母;
  * 需要用大写字母时需要使用双引号或借助函数`upper()`;
* `SQLServer`
  * 默认是大小写不敏感;
* `MySQL`
  * 在**Linux环境**下数据库名,表名,列名,别名的大小写规则是这样的:
    * **数据库名与表名是严格区分大小写的;**
    * **表的别名是严格区分大小的;**
    * *列名与列的别名在所有情况下均是忽略大小写的;*
    * **变量名也是严格区分大小写的;**
  * 在**Windows环境**下都是不区分大小写的
* 在不同操作系统中为了能是程序和数据库都能正常运行,最好的办法是在设计的时候都转为小写,但是如果在设计的时候已经规范大小写了,那么可以在`MySQL`配置文件中`my.ini`中的`[mysqld]`中增加一行`low_case_table_names=1`
  * `0`: 区分大小写;
  * `1`: 不区分大小写;

### 数据库编码和字符集引发的乱码问题

* 乱码的本质原因是:**数据编码和数据解码所使用的编码方式一致**.

#### 问题现象

* MySQL版本8.0.25

  ```mysql
  mysql> select version();
  +-----------+
  | version() |
  +-----------+
  | 8.0.25    |
  +-----------+
  1 row in set (0.01 sec)
  ```

* MySQL字符集变量所设置的值

  ```mysql
  mysql> show variables like '%character%';
  +--------------------------+--------------------------------+
  | Variable_name            | Value                          |
  +--------------------------+--------------------------------+
  | character_set_client     | latin1                         |
  | character_set_connection | latin1                         |
  | character_set_database   | utf8mb4                        |
  | character_set_filesystem | binary                         |
  | character_set_results    | latin1                         |
  | character_set_server     | utf8mb4                        |
  | character_set_system     | utf8mb3                        |
  | character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
  +--------------------------+--------------------------------+
  8 rows in set (0.01 sec)
  ```

* 程序中配置的MySQL的URL中所指定的字符编码为`UTF-8`

  ```mysql
  jdbc:mysql://$IP/xxxx?useUnicode=true&characterEncoding=utf8
  ```

* 使用`mysql -u $USER -p`命令登录,在数据表中插入中文字符并使用查询语句查询,显示正常.但是使用程序来读取的时候出现了中文乱码.

  ```mysql
  mysql> select * from unique_test;
  +----+--------------+
  | id | content      |
  +----+--------------+
  |  1 | 测试编码测试编码测试编码测试编码测试编码测试编码|
  +----+--------------+
  1 row in set (0.00 sec)
  ```
  
* 使用`mysql -u root --default-character-set=utf8mb4 -p`登录
  
  ```
  root@bef53fced63c:/# mysql -u root --default-character-set=utf8mb4 -p
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 15
  Server version: 8.0.22 MySQL Community Server - GPL
  
  Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql> use test;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A
  
  Database changed
  mysql> select * from unique_test;
  +----+-----------------------------+
  | id | content                     |
  +----+-----------------------------+
  |  1 | æµ‹è¯•ç¼–ç                 |
  +----+-----------------------------+
  1 row in set (0.00 sec)
  ```
  
  
  
  <img src="https://www.holelin.cn/img/mysql/questions/MySQL%E4%B9%B1%E7%A0%81%E9%97%AE%E9%A2%98.png" alt="img" style="zoom:50%;" />

#### 问题原因

* 因为mysql服务配置项`character_set_client `和`character_set_connection`都为`latin1`编码方式,使用`mysql -u $USER -p`并为指定字符集故而登录成功后使用系统默认的字符集(即`latin1`),而程序使用到的编码为`UTF-8`,故而编码和解码所使用的方式不一致,最终导致乱码的出现.

#### 处理方法

* 治标
  * 在登录mysql的使用指定字符集`mysql -u $USER --default-character-set=utf8mb4 -p`
* 治本
  * 修改mysql服务的客户端编码和连接编码改为`utf8mb4`

#### MySQL `datetime`类型长度问题

* 示例

  ```
  // create_time字段 类型为datetime 长度为6
  created_time  datetime 6
  ```

* 以`2022-03-14 18:32:33.000000`为例`datetime`长度为6,6为这个时间格式"."后面数字的长度.

* 若不指定`datetime`的长度,则数据格式为`2022-03-14 18:32:33`

### MySQL `order by` 排序使用的字段有重复值导致排序随机的情况

#### 问题描述

* 有一张数据表,其中有部分数据支付时间是一样的,通过SQL对创建时间排序以及分页(`pageNo`,`pageSize`)  发现调整`pageSize`的值 如5,10,400第一条的数据会变化，即数据的顺序不固定

#### 官方说明

> If multiple rows have identical values in the `ORDER BY` columns, the server is free to return those rows in any order, and may do so differently depending on the overall execution plan. In other words, the sort order of those rows is nondeterministic with respect to the nonordered columns.
>
> 如果ORDER BY列中的多行具有相同的值，服务器可以**自由地以任何顺序**返回这些行，而且根据总体执行计划的不同，返回顺序也可能不同。换句话说，与无序列相比，这些行的排序顺序是不确定的。

#### 解决方案

##### 官方给的解决方案

> If it is important to ensure the same row order with and without `LIMIT`, include additional columns in the `ORDER BY` clause to make the order deterministic. For example, if `id` values are unique, you can make rows for a given `category` value appear in `id` order by sorting like this:
>
> 如果确保使用和不使用“LIMIT”的行顺序很重要，那么在“order BY”子句中包括额外的列，以使顺序具有确定性。例如，如果' id '值是唯一的，你可以让一个给定的' category '值的行出现在' id '顺序，排序如下:

```mysql
mysql> SELECT * FROM ratings ORDER BY category, id;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
|  2 |        3 |    5.0 |
|  7 |        3 |    2.7 |
+----+----------+--------+

mysql> SELECT * FROM ratings ORDER BY category, id LIMIT 5;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
+----+----------+--------+
```

* 即加上额外的列,保证顺序确定.
