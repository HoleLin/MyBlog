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

### 触发器

- Before Insert
- After Insert
- Before Update
- After Update
- Before Delete
- After Delete

#### **使用场景**

- 可以通过数据库中的相关表实现级联更改。
- 实时监控某张表中的某个字段的更改而需要做出相应的处理。
- 例如可以生成某些业务的编号。
- 注意不要滥用，否则会造成数据库及应用程序的维护困难。
- 大家需要牢记以上基础知识点，重点是理解数据类型CHAR和VARCHAR的差异，表存储引擎InnoDB和MyISAM的区别。

#### 

### 常见问题

#### **char、varchar的区别是什么？**

> varchar是变长而char的长度是固定的。

#### **FLOAT和DOUBLE的区别是什么?**

- FLOAT类型数据可以存储至多8位十进制数，并在内存中占4字节。
- DOUBLE类型数据可以存储至多18位十进制数，并在内存中占8字节。

#### **请说明varchar和text的区别**

- varchar可指定字符数，text不能指定，内部存储varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，text是实际字符数+2个字节。
- text类型不能有默认值。
- varchar可直接创建索引，text创建索引要指定前多少个字符。varchar查询速度快于text,在都创建索引的情况下，text的索引几乎不起作用。
- 查询text需要创建临时表。

#### **varchar(50)中50的含义**

* 最多存放50个字符，varchar(50)和(200)存储hello所占空间一样，但后者在排序时会消耗更多内存，因为order by col采用fixed_length计算col长度(memory引擎也一样)。

#### **int(20)中20的含义**

* 是指显示字符的长度，不影响内部存储，只是当定义了ZEROFILL时，前面补多少个 0
