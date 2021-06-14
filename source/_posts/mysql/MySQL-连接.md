---
title: MySQL连接
mermaid: true
date: 2021-06-12 19:46:40
cover: /img/cover/MySQL.jpg
tags:
- 基础理论
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



### 连接(JOIN)

<img src="http://www.chenjunlin.vip/img/mysql/join.png" alt="img" style="zoom: 67%;" />

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