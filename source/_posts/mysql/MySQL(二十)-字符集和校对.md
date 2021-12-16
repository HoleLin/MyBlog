---
title: MySQL(二十)-字符集和校对
date: 2021-12-16 13:49:04
cover: /img/cover/MySQL.jpg
tags:
- 字符集和校对
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

* 高性能MySQL(第三版)

### 字符集和校对

* 字符集是指一种二进制编码到某类字符符号的映射.
* 校对是指一组用于某个字符集的排序规则.
* MySQL有很多的选项用于控制字符集.这些选项和字符集很容易混淆,一定要记住:只有基于字符的值才真正的"有"字符集的概念.对于其他类型的值,字符集只是一个设置,指定用哪一种字符集来做比较或者其他操作.基于字符的值能存放在某列中,查询的字符串中,表达式的计算结果中或者某个用户变量中,等等.
* MySQL的设置可以分为两类:
  * 创建对象时的默认值;
  * 在服务器和客户端通信时的设置;

#### 创建对象是的默认设置

* MySQL服务器有默认的字符集和校对规则,每个数据库也有自己的默认值,每个表也有自己的默认值.这是一个逐层继承的默认设置,最终最靠底层的默认设置将影响创建的对象.

  * 创建数据库的时候,将根据服务器上的`character_set_server`设置来设定该数据库的默认字符集.

  * 创建表的时候,将根据数据库的字符集设置指定这个表的字符集设置.

  * 创建列的时候,将根据表的设置指定列的字符集设置.

* 需要注意的是,真正存放数据的列,所以更高阶梯的设置只是指定默认值.一个表的默认字符集设置无法影响存储在这个表中某个列的值.只有当创建列而没有为列指定字符集的时候,如果没有指定字符集,表的默认字符集才能起作用.

#### 服务器和客户端通信时的设置

* 当服务器和客户端通信时候,它们可能使用不同的字符集.这时,服务器端将进行必要的翻译转换工作:
  * 服务器端总是假设客户端是按照`character_set_clinet`设置的字符来传输数据和SQL语句的.
  * 当服务器收到客户端的SQL语句时,它先将其转换成字符集`character_set_connection`.它还使用这个设置来决定如何将数据转换成字符串.
  * 当服务器端返回数据或者错误信息给客户端时,它会将其转换成`character_set_result`;

#### 一些特殊情况

##### 诡异的`character_set_database`设置

* `character_set_database`设置的默认值和默认数据库的设置相同.当改变默认数据库字符集的时候,这个变量也会跟着变.所以当连接到MySQL实例上又没有指定要使用的数据库字符集,默认值会和`character_set_server`相同.

##### `LOAD DATA INFILE`

* 当使用`LOAD DATA INFILE`的时候,数据库总是将恩建中字符按照字符集`character_set_database`来解析.在MySQL5.0和更新的版本中,可以在`LOAD DATA INFILE`中使用子句`CHARACTER SET`来设定字符集,不过最好不要依赖这个设定.最好的方式是先使用USE指定数据库,在执行`SET NAMES`来设定字符集,最后在加载数据.
* MySQL在加载数据的时候,总是以同样的字符集处理所有数据,而不管表中的列是否有不同的字符集设定.

##### `SELECT INTO OUTFILE`

* MySQL会将`SELECT INTO OUTFILE`的结果不做任何转码的写入文件.

#### 查看和设置字符集

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

