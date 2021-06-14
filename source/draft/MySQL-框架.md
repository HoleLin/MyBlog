---
title: MySQL
mermaid: true
date: 2021-06-14 14:46:40
cover: /img/cover/MySQL.jpg
tags:
- 原理
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

### MySQL基本框架

<img src="http://www.chenjunlin.vip/img/mysql/MySQL%E5%9F%BA%E6%9C%AC%E6%A1%86%E6%9E%B6%E5%9B%BE.png" alt="img" style="zoom: 67%;" />

#### 连接器

> 负责与客户端建立连接,获取权限,维持和管理连接

```sh
mysql -h$ip -P$port -u#user -p
```

> "Access defined for user" 用户名或密码不正确

* 认证通过后,连接器会读取当前账号的权限(**当前读到的权限**),若此时修改该账号的权限,也不影响该账户的权限,只有重新建立连接,才会使用新权限;
* 查看数据库的连接状态:`show processlist`

#### 查询缓存

* 设置不使用查询缓存,设置`query_cache_type`为`DEMAND`;

* 显示查询缓存

  ```sql
  SELECT SQL_CACHE * FROM T WHERE id = 10;
  -- MySQL8.0版本,查询缓存被移除了
  ```

#### 分析器

* 对于语句进行分析
  * **词法分析**: 识别出SQL语句中字符串分别是什么(根据MySQL的关键字进行验证和解析),代表什么;
  * **语法分析**: 语法分析器会根据语法规则,判断SQL语句是否满足MySQL语法;
    * 在词法分析的基础上进一步做表和字段名称的验证和解析;

#### 优化器

> 优化器是在表里面有多个索引的时候,决定使用哪个索引,或者在语句有多表关联(join)的时候,决定各个表的连接顺序.

#### 执行器

> 执行前会判断是否有对该表的操作权限;
>
> * 若无权限,则会返回没有权限;
> * 有权限,则打开表继续执行,打开表的时候,执行器就会根据表的引擎定义,使用引擎提供的接口;