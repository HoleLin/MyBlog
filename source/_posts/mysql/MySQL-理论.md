---
title: MySQL-理论
mermaid: false
date: 2021-06-16 18:31:42
cover: /img/cover/MySQL.jpg
tags:
- 理论
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

* [MySQL 数据库面试题（2021最新版）](https://mp.weixin.qq.com/s/8uMUu5vqPShVv4LatCD5qg)

### **数据库三大范式**

- 第一范式：每个列都不可以再拆分。
- 第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。
- 第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

### **MySQL有关权限的表**

>  MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由`mysql_install_db`脚本初始化。这些权限表分别`user，db，table_priv，columns_priv和host`。

* user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。

* db权限表：记录各个帐号在各个数据库上的操作权限。

* table_priv权限表：记录数据表级的操作权限。

* columns_priv权限表：记录数据列级的操作权限。

* host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。

### **MySQL的binlog录入格式**

- statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
- mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。
