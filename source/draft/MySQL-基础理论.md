---
title: MySQL基础理论
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

#### MySQL目录结构

> bin: 存储可执行文件;
>
> data: 存储数据文件;
>
> docs: 文档
>
> include: 存储包含的头文件
>
> lib: 存储库文件
>
> share: 错误消息和字符集文件

### 约束

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
