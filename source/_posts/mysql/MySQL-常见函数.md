---
title: MySQL常见函数
mermaid: true
date: 2021-06-12 19:45:53
cover: /img/cover/MySQL.jpg
tags:
- 函数
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

### MySQL常见函数

* 字符函数
* 数值运算符与函数
* 比较运算符与函数
* 日期时间函数
* 信息函数
* 聚合函数
* 加密函数

#### 字符函数

| 函数名称    | 描述                         |
| ----------- | ---------------------------- |
| CONCAT()    | 字符连接                     |
| CONCAT_WS() | 使用指定的分隔符进行字符连接 |
| FORMAT()    | 数字格式化函数               |
| LOWER()     | 转换为小写字母               |
| UPPER()     | 转换为大写字母               |
| LEFT()      | 获取左侧字符                 |
| RIGHT()     | 获取右侧字符                 |
| LENGTH()    | 获取字符串长度               |
| LTRIM()     | 删除前导空格                 |
| RTRIM()     | 删除后导空格                 |
| SUBSTRING() | 字符串截取                   |
| [NOT] LIKE  | 模式匹配                     |
| REPLACE()   | 字符串替换                   |

#### 数值运算符与函数

| 函数名称   | 描述               |
| ---------- | ------------------ |
| CEIL()     | 进一取整(向上取整) |
| FLOOR()    | 舍一取整(向下取整) |
| DIV        | 整数除法           |
| MOD        | 取模               |
| POWER()    | 幂运算             |
| ROUND()    | 四舍五入           |
| TRUNCATE() | 数字截取           |

#### 比较运算符与函数

| 函数名称            | 描述               |
| ------------------- | ------------------ |
| [NOT] BETWEEN...AND | [不]在范围之内     |
| [NOT]IN()           | [不]在列出值范围内 |
| IS [NOT] NULL       | [不]为空           |

#### 日期时间函数 

| 函数名称      | 描述           |
| ------------- | -------------- |
| NOW()         | 当前日期和时间 |
| CURDATE()     | 当前日期       |
| CURTIME()     | 当前时间       |
| DATE_ADD()    | 日期变化       |
| DATEDIFF()    | 日期差值       |
| DATE_FORMAT() | 日期格式化     |

#### 信息函数

| 函数名称        | 描述               |
| --------------- | ------------------ |
| CONNECTION_ID() | 连接ID             |
| DATABASE()      | 当前数据库         |
| LAST_INSERT_ID  | 最后插入记录的ID号 |
| USER()          | 当前用户           |
| VERSION()       | 版本信息           |

#### 聚合函数(只有一个返回值)

| 函数名称 | 描述   |
| -------- | ------ |
| AVG()    | 平均值 |
| COUNT()  | 计数   |
| MAX()    | 最大值 |
| MIN()    | 最小值 |
| SUM()    | 求和   |

#### 加密函数

| 函数名称  | 描述         |
| --------- | ------------ |
| MD5()     | 信息摘要算法 |
| PSSWORD() | 密码算法     |

