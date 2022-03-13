---
title: Java基础-数据库配置
date: 2022-03-10 11:48:30
cover: /img/cover/Java.jpg
tags:
categories:
- Java
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

### 各种数据库URL配置

```tcl
mysql:
jdbc:mysql://host:port//db_name?

SQL Server:
jdbc:sqlserver://host:port//db_name?

Oracle:
jdbc:oracle:thin:@host:port:db_name

MongoDB:
# 无密码
mongodb://host:port/db_name
# 有密码
mongodb://username:password@host:port/db_name
```

