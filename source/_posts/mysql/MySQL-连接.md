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

<img src="https://www.holelin.cn/img/mysql/join.png" alt="img" style="zoom: 67%;" />

```sql
 将多个表的字段进行连接，可以指定连接条件。
-- 内连接(inner join)
    - 默认就是内连接，可省略inner。
    - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    -- Using连接
    还有 using, 但需字段名相同。using(字段名)
    -- 交叉连接 cross join
        即，没有条件的内连接。
        select * from tb1 cross join tb2;
-- 外连接(outer join)
    - 如果数据不存在，也会出现在连接结果中。
    -- 左外连接 left join
        如果数据不存在，左表记录会出现，而右表为null填充
    -- 右外连接 right join
        如果数据不存在，右表记录会出现，而左表为null填充
-- 自然连接(natural join)
    自动判断连接条件完成连接。
    相当于省略了using，会自动查找相同字段名。
    natural join
    natural left join
    natural right join
    
select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;
```



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