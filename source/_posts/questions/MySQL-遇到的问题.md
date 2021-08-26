---
title: MySQL-遇到的问题
date: 2021-07-19 15:42:07
cover: /img/cover/MySQL.jpg
tags:
- 问题
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

#### 数据库大小敏感问题

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

