---
title: MySQL中utf8与utf8mb4的区别
mermaid: true
date: 2021-06-13 19:43:29
cover: /img/cover/MySQL.jpg
tags:
- 字符编码
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

### MySQL中`utf8`与`utf8mb4`的区别

* `utf8mb4`是MySQL在5.3.3后增加的编码,其中`mb4`的意思是: `most byte 4`,专门兼容四字节的`Unicode`

* 原来的MySQL支持的`utf8`编码最大字符长度为3字节,若遇到4字节的宽字符就会插入异常

  > 3字节的`utf8`最大能编码的`Unicode`字符为`Oxffff`,即`Unicode`中基本多文种平面(`BMP`),即**任何不在基本多文种平面的`Unicode`字符都无法使用MySQL的`utf8`字符集进行存储,包括`Emoji`表情和很多不常用的汉字**

* **当使用`utf8`字符集时,需要保留长度就是`utf8`最长字符乘以字符串长度**,例`CHAR(100)`,MySQL会保留300字节长度

* `utf8`升级`utf8mb4`步骤

  * 首先将数据库默认字符集有`utf8`改为`utf8mb4`,对应的表默认字符集也要改为`utf8mb4`,已经存储的字段默认字符集也要进行相应调整;

    ```
    -- 修改数据库
    ALTER DATABASE db_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
    -- 修改表
    ALTER TABLE table_name CONVERT TO CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
    -- 修改字段
    ALTER TABLE table_name CHANGE column_name column_name VARCHAR(100) CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
    ```

  * 修改MySQL配置文件

    ```yaml
    default-character = uft8mb4
    character-set-client-handshake = FALSE
    character-set-server = utfmb4
    collation-server = utfmb4_unicode_ci
    init_connect = 'SET NAMES utf8mb4'
    ```

    

