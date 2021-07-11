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

```sql
-- // 内置函数 ----------
-- 数值函数
abs(x)          -- 绝对值 abs(-10.9) = 10
format(x, d)    -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
ceil(x)         -- 向上取整 ceil(10.1) = 11
floor(x)        -- 向下取整 floor (10.1) = 10
round(x)        -- 四舍五入去整
mod(m, n)       -- m%n m mod n 求余 10%3=1
pi()            -- 获得圆周率
pow(m, n)       -- m^n
sqrt(x)         -- 算术平方根
rand()          -- 随机数
truncate(x, d)  -- 截取d位小数
-- 时间日期函数
now(), current_timestamp();     -- 当前日期时间
current_date();                 -- 当前日期
current_time();                 -- 当前时间
date('yyyy-mm-dd hh:ii:ss');    -- 获取日期部分
time('yyyy-mm-dd hh:ii:ss');    -- 获取时间部分
date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j'); -- 格式化时间
unix_timestamp();               -- 获得unix时间戳
from_unixtime();                -- 从时间戳获得时间
-- 字符串函数
length(string)          -- string长度，字节
char_length(string)     -- string的字符个数
substring(str, position [,length])      -- 从str的position开始,取length个字符
replace(str ,search_str ,replace_str)   -- 在str中用replace_str替换search_str
instr(string ,substring)    -- 返回substring首次在string中出现的位置
concat(string [,...])   -- 连接字串
charset(str)            -- 返回字串字符集
lcase(string)           -- 转换成小写
left(string, length)    -- 从string2中的左边起取length个字符
load_file(file_name)    -- 从文件读取内容
locate(substring, string [,start_position]) -- 同instr,但可指定开始位置
lpad(string, length, pad)   -- 重复用pad加在string开头,直到字串长度为length
ltrim(string)           -- 去除前端空格
repeat(string, count)   -- 重复count次
rpad(string, length, pad)   --在str后用pad补充,直到长度为length
rtrim(string)           -- 去除后端空格
strcmp(string1 ,string2)    -- 逐字符比较两字串大小
-- 流程函数
case when [condition] then result [when [condition] then result ...] [else result] end   多分支
if(expr1,expr2,expr3)  双分支。
-- 聚合函数
count()
sum();
max();
min();
avg();
group_concat()
-- 其他常用函数
md5();
default();
```

