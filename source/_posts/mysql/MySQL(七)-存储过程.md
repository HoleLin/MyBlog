---
title: MySQL(七)-存储过程,视图,触发器
date: 2021-08-20 14:46:11
cover: /img/cover/MySQL.jpg
tags:
- 存储过程
- 视图
- 触发器
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

### 存储过程

```
-- 修改最外层语句结束符
delimiter 自定义结束符号
    SQL语句
自定义结束符号
delimiter ;     -- 修改回原来的分号

-- 语句块包裹
begin
    语句块
end

-- 特殊的执行
1. 只要添加记录，就会触发程序。
2. Insert into on duplicate key update 语法会触发：
    如果没有重复记录，会触发 before insert, after insert;
    如果有重复记录并更新，会触发 before insert, before update, after update;
    如果有重复记录但是没有发生更新，则触发 before insert, before update
3. Replace 语法 如果有记录，则执行 before insert, before delete, after delete, after insert
```

```
--// 局部变量 ----------
-- 变量声明
    declare var_name[,...] type [default value]
    这个语句被用来声明局部变量。要给变量提供一个默认值，请包含一个default子句。值可以被指定为一个表达式，不需要为一个常数。如果没有default子句，初始值为null。
-- 赋值
    使用 set 和 select into 语句为变量赋值。
    - 注意：在函数内是可以使用全局变量（用户自定义的变量）


--// 全局变量 ----------
-- 定义、赋值
set 语句可以定义并为变量赋值。
set @var = value;
也可以使用select into语句为变量初始化并赋值。这样要求select语句只能返回一行，但是可以是多个字段，就意味着同时为多个变量进行赋值，变量的数量需要与查询的列数一致。
还可以把赋值语句看作一个表达式，通过select执行完成。此时为了避免=被当作关系运算符看待，使用:=代替。（set语句可以使用= 和 :=）。
select @var:=20;
select @v1:=id, @v2=name from t1 limit 1;
select * from tbl_name where @var:=30;
select into 可以将表中查询获得的数据赋给变量。
    -| select max(height) into @max_height from tb;
-- 自定义变量名
为了避免select语句中，用户自定义的变量与系统标识符（通常是字段名）冲突，用户自定义变量在变量名前使用@作为开始符号。
@var=10;
    - 变量被定义后，在整个会话周期都有效（登录到退出）



--// 控制结构 ----------
-- if语句
if search_condition then
    statement_list   
[elseif search_condition then
    statement_list]
...
[else
    statement_list]
end if;

-- case语句
CASE value WHEN [compare-value] THEN result
[WHEN [compare-value] THEN result ...]
[ELSE result]
END

-- while循环
[begin_label:] while search_condition do
    statement_list
end while [end_label];
- 如果需要在循环内提前终止 while循环，则需要使用标签；标签需要成对出现。
    -- 退出循环
        退出整个循环 leave
        退出当前循环 iterate
        通过退出的标签决定退出哪个循环

--// 存储函数，自定义函数 ----------
-- 新建
    CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
        函数体
    - 函数名，应该合法的标识符，并且不应该与已有的关键字冲突。
    - 一个函数应该属于某个数据库，可以使用db_name.funciton_name的形式执行当前函数所属数据库，否则为当前数据库。
    - 参数部分，由"参数名"和"参数类型"组成。多个参数用逗号隔开。
    - 函数体由多条可用的mysql语句，流程控制，变量声明等语句构成。
    - 多条语句应该使用 begin...end 语句块包含。
    - 一定要有 return 返回值语句。
-- 删除
    DROP FUNCTION [IF EXISTS] function_name;
-- 查看
    SHOW FUNCTION STATUS LIKE 'partten'
    SHOW CREATE FUNCTION function_name;
-- 修改
    ALTER FUNCTION function_name 函数选项

--// 存储过程，自定义功能 ----------
-- 定义
存储存储过程 是一段代码（过程），存储在数据库中的sql组成。
一个存储过程通常用于完成一段业务逻辑，例如报名，交班费，订单入库等。
而一个函数通常专注与某个功能，视为其他程序服务的，需要在其他语句中调用函数才可以，而存储过程不能被其他调用，是自己执行 通过call执行。
-- 创建
CREATE PROCEDURE sp_name (参数列表)
    过程体
参数列表：不同于函数的参数列表，需要指明参数类型
IN，表示输入型
OUT，表示输出型
INOUT，表示混合型
注意，没有返回值。


/* 存储过程 */ ------------------
存储过程是一段可执行性代码的集合。相比函数，更偏向于业务逻辑。
调用：CALL 过程名
-- 注意
- 没有返回值。
- 只能单独调用，不可夹杂在其他语句中
-- 参数
IN|OUT|INOUT 参数名 数据类型
IN      输入：在调用过程中，将数据输入到过程体内部的参数
OUT     输出：在调用过程中，将过程体处理完的结果返回到客户端
INOUT   输入输出：既可输入，也可输出
-- 语法
CREATE PROCEDURE 过程名 (参数列表)
BEGIN
    过程体
END
```

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


### 视图

```
什么是视图：
    视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
    视图具有表结构文件，但不存在数据文件。
    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。
    视图是存储在数据库中的查询的sql语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。
-- 创建视图
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
    - 视图名必须唯一，同时不能与表重名。
    - 视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    - 可以指定视图执行的算法，通过ALGORITHM指定。
    - column_list如果存在，则数目必须等于SELECT语句检索的列数
-- 查看结构
    SHOW CREATE VIEW view_name
-- 删除视图
    - 删除视图后，数据依然存在。
    - 可同时删除多个视图。
    DROP VIEW [IF EXISTS] view_name ...
-- 修改视图结构
    - 一般不修改视图，因为不是所有的更新视图都会映射到表上。
    ALTER VIEW view_name [(column_list)] AS select_statement
-- 视图作用
    1. 简化业务逻辑
    2. 对客户端隐藏真实的表结构
-- 视图算法(ALGORITHM)
    MERGE       合并
        将视图的查询语句，与外部查询需要先合并再执行！
    TEMPTABLE   临时表
        将视图执行完毕后，形成临时表，再做外层查询！
    UNDEFINED   未定义(默认)，指的是MySQL自主去选择相应的算法。
```



### 触发器

  ```
      触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象
      监听：记录的增加、修改、删除。
  -- 创建触发器
  CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
      参数：
      trigger_time是触发程序的动作时间。它可以是 before 或 after，以指明触发程序是在激活它的语句之前或之后触发。
      trigger_event指明了激活触发程序的语句的类型
          INSERT：将新行插入表时激活触发程序
          UPDATE：更改某一行时激活触发程序
          DELETE：从表中删除某一行时激活触发程序
      tbl_name：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
      trigger_stmt：当触发程序激活时执行的语句。执行多个语句，可使用BEGIN...END复合语句结构
  -- 删除
  DROP TRIGGER [schema_name.]trigger_name
  可以使用old和new代替旧的和新的数据
      更新操作，更新前是old，更新后是new.
      删除操作，只有old.
      增加操作，只有new.
  -- 注意
      1. 对于具有相同触发程序动作时间和事件的给定表，不能有两个触发程序。
  ```

