---
title: MySQL-存储过程
date: 2021-08-20 14:46:11
cover: /img/cover/MySQL.jpg
tags:
- 存储过程
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

#### 语法

```mysql
-- 创建存储过程
CREATE PROCEDURE 存储过程名称([参数列表])
BEGIN
	需要执行的语句
END
-- 删除存储过程
DROP PROCEDURE
-- 更新存储过程
ALTER PROCEDURE
```

#### 参数

| 参数类型 | 是否返回 | 作用                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| IN       | 否       | 向存储过程传入参数，存储过程中修改该参数的值，不能被返回     |
| OUT      | 是       | 把存储过程计算的结果放到该参数中，调用者可以得到返回值       |
| INOUT    | 是       | IN和OUT的结合，既用于存储过程的传入参数，同时又可以把计算结果放到参数中，调用者可以得到返回值 |

#### 流程控制语句

* `BEGIN..END`: `BEGIN...END`中间包含了多个语句，每个语句都以(;)号为结束符。

* `DECLARE`: `DECLARE`用来声明变量，使用的位置在与`BEGIN...END`语句中间，而且需要在其他语句使用之前进行变量的声明

* `SET`:赋值语句，用于对变量进行赋值

* `SELECT...INTO`: 把数据表中查询的结果存放到变量中，也就是为变量赋值

* `IF...THEN...ENDIF`:条件判断语句，还可以使用`IF...THEN...ENDIF`

* `CASE`:用于多条件判断

  ```mysql
  CASE 
  	WHEN experssion1 THEN ...
  	WHEN experssion1 THEN ...
  	-- ELSE可以加，可不加。加的话代表的所有条件都不满足时采用的方式
  	ELSE
  END	
  ```

* `LOOP,LEAVE,ITERATE`:`LOOP`是循环语句，使用`LEAVE`（BREAK）可以跳出循环，使用`ITERATE`（CONTINUE）则可以进入下一次循环

* `REPEAT...UNIT...END REPEAT`:这个循环语句，首先会执行一次，饭后在UNITL中进行表达式判断，如果满足条件就退出，即END REPAEAT；如果条件不满足，则会继续执行循环，知道满足退出条件为止。

* `WHILE...DO...END WHILE`:也是循环，和REPEAT循环不同的是，这个语句需要先进行条件判断，如果满足条件就进行循环，如果不满足条件就退出循环。

#### 游标

##### 游标操作步骤

* 定义

  ```mysql
  DECLARE cursor_name CURSOR FOR select_stament
  ```

* 打开游标

  ```mysql
  OPEN cursor_name
  ```

* 从游标中取得数据

  ```mysql
  FETCH cursor_name INTO var_name,....
  ```

* 关闭游标

  ```mysql
  CLOSE cursor_name
  ```

* 释放游标

  ```mysql
  DEALLOCATE PREPARE
  ```

  

  
