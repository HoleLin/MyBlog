---
title: MySQL(二)-运维操作(二)
date: 2021-09-03 16:07:22
cover: /img/cover/MySQL.jpg
tags:
- 运维操作
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

#### 慢查询

#### 命令行接口

* `mysql --column-type-info` 显示结果集元数据

  ```
  -- 字段名
  Field  11:  `produced_time`
  -- 目录名称
  Catalog:    `def`
  -- 数据库名称
  Database:   `lack`
  -- 数据表名称, 当使用select field_name from table_name as alias_name语法时,这里显示的是表的别名
  Table:      `orders`
  -- 原始表名,当前面行显示的别名,知道这个就非常有用了
  Org_table:  `orders`
  -- 前面的行显示字段类型
  Type:       DATETIME
  -- 排序规则
  Collation:  binary (63)
  -- 表定义中定义的字段长度
  Length:     26
  -- 返回结果集字段长度的最大值
  Max_length: 0
  -- 如果是一个整数类型,则表示该字段中小数点后的位数
  Decimals:   6
  Flags:      BINARY 
  ```

#### MySQL沙箱

* [参考文献](https://www.cnblogs.com/gomysql/p/3767445.html)

#### MySQL通用日志

* 开启通用日志

  ```mysql
  set global general_log=1;
  set global log_output='table';
  ```

* 查看通用日志的内容

  ```mysql
  SELECT argument FROM mysql.general_log ORDER BY	event_time desc
  ```

### 故障排除的一般步骤

* 尝试确定造成问题的实际查询;
* 检查以确保查询的语句正确;
* 确认查询里有问题;
* 如果查询返回错误数据时,请尝试重写它以得到正确的结果;
* 如果重写没用,可以检查服务器选项并尝试确定它们是否影响结果;
* 如果问题不能再MySQL CLI重现,请检查它是否有并发问题;
* 如果该问题会导致系统崩溃或挂起,首先检查错误日志;

