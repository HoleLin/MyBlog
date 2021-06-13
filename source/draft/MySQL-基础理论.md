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

### MySQL目录结构

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

### 约束(Constraint)

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

### 连接(JOIN)

<img src="http://www.chenjunlin.vip/img/mysql/join.png" alt="img" style="zoom: 67%;" />

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

#### 由外连接

* 显示右表的全部记录以及左表中符合连接条件的记录;

### 索引

#### 索引的分类

* 单值索引

  > 即一个索引只包含单个列，一个表可以有多个单列索引

  * 示例

    ```sql
    CREATE TABLE `user` (
        id INT(10) UNSIGNED AUTO_INCREMENT,
        age int(3),
        name VARCHAR(200), 
        PRIMARY KEY(id), 
        -- 单值索引   
        KEY (name)   
    );
    
    -- 单独创建单值索引
    CREATE INDEX idx_user_name ON `user`(name);
    ```
  
* 唯一索引

  > 索引列的值必须唯一，但允许有空值

  * 示例

    ```sql
    CREATE TABLE user (
        id INT(10) UNSIGNED AUTO_INCREMENT,
        age int(3),
        name VARCHAR(200), 
        id_card varchar(32),
        PRIMARY KEY(id), 
        -- 单值索引   
        KEY (name),
        -- 唯一索引
        UNIQUE KEY (id_card)
    );
    
    -- 单独创建索引
    CREATE INDEX idx_user_id_card ON `user`(id_card);
    ```

* 主键索引

  > 设定为主键后数据库会**自动建立索引**，innodb为聚簇索引

  * 示例

    ```sql
    -- 和表一起创建
    CREATE TABLE `user` (
        id INT(10) UNSIGNED AUTO_INCREMENT,
        PRIMARY KEY(id)
    );
    
    --单独创建主键索引
    ALTER TABLE `user` ADD PRIMARY KEY user(id);
    
    --删除主键索引
    ALTER TABLE `user` DROP PRIMARY KEY;
    ```
  
* 复合索引

  > 即一个索引包含多个列

  * 示例

    ```sql
    CREATE TABLE `user` (
        id INT(10) UNSIGNED AUTO_INCREMENT,
        age int(3),
        name VARCHAR(200), 
        department_id varchar(100),
        id_card varchar(32),
        PRIMARY KEY(id), 
        -- 单值索引   
        KEY (name),
        -- 唯一索引
        UNIQUE KEY (id_card),
        -- 复合索引
        KEY (name,age,department_id) 
    );
    
    --单独创建复合索引
    CREATE INDEX idx_student ON `user`(name,age,department_id);
    ```

#### B树与B+树

##### 区别

  - B树的**关键字和记录是放在一起的**，叶子节点可以看作外部节点，不包含任何信息；B+树的非叶子节点中只有关键字和指向下一个节点的索引，**记录只放在叶子节点中**
  - 在 B树中，越靠近根节点的记录查找时间越快，只要找到关键字即可确定记录的存在；而 B+树中每个记录 的查找时间基本是一样的，都需要从根节点走到叶子节点，而且在叶子节点中还要再比较关键字。从这个角度看 B树的性能好像要比 B+树好，而在实际应用中却是 B+树的性能要好些。因为 B+树的非叶子节点不存放实际的数据， 这样每个节点可容纳的元素个数比 B树多，树高比 B树小，这样带来的好处是减少磁盘访问次数。尽管 B+树找到 一个记录所需的比较次数要比 B树多，但是一次磁盘访问的时间相当于成百上千次内存比较的时间，因此实际中 B+树的性能可能还会好些，而且 B+树的叶子节点使用指针连接在一起，方便顺序遍历（例如查看一个目录下的所有 文件，一个表中的所有记录等），这也是很多数据库和文件系统使用 B+树的缘故

  

  

  

  

