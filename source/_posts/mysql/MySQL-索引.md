---
title: MySQL索引
mermaid: true
date: 2021-06-12 19:41:38
cover: /img/cover/MySQL.jpg
tags:
- 索引
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

### 索引

> 关键字与数据的映射关系称为索引（包含关键字和对应的记录在磁盘中的地址）。关键字是从数据当中提取的用于标识、检索数据的特定内容。

#### 索引检索为什么快？

- 关键字相对于数据本身，数据量小
- 关键字是有序的，二分查找可快速确定位置

#### 索引分类

* **单值索引**

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

* **唯一索引**

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

* **主键索引**

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

* **复合索引**

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

* **全文索引**（`fulltext key`

#### 如何创建索引

建立基础索引：在`where、order by、join`字段上建立索引。

优化组合索引：基于业务逻辑

- 如果条件经常性出现在一起，那么可以考虑将多字段索引升级为`复合索引`;
- 如果通过增加个别字段的索引，就可以出现`索引覆盖`，那么可以考虑为该字段建立索引;
- 查询时，不常用到的索引，应该删除掉;

#### 前缀索引

* 语法：`index(field(10))`，使用字段值的前10个字符建立索引，默认是使用字段的全部内容建立索引。
* 前提：前缀的标识度高。比如密码就适合建立前缀索引，因为密码几乎各不相同。
* 实操的难度：在于前缀截取的长度。
* 我们可以利用`select count(*)/count(distinct left(password,prefixLen));`，通过从调整`prefixLen`的值（从1自增）查看不同前缀长度的一个平均匹配度，接近1时就可以了（表示一个密码的前`prefixLen`个字符几乎能确定唯一一条记录）

#### 索引使用场景

##### where

> 可以尝试在一个字段未建立索引时，根据该字段查询的效率，然后对该字段建立索引（`alter table 表名 add index(字段名)`），同样的SQL执行的效率，你会发现查询效率会有明显的提升（数据量越大越明显）

##### order by

> 当我们使用`order by`将查询结果按照某个字段排序时，如果该字段没有建立索引，那么执行计划会将查询出的所有数据使用外部排序（将数据从硬盘分批读取到内存使用内部排序，最后合并排序结果），这个操作是很影响性能的，因为需要将查询涉及到的所有数据从磁盘中读到内存（如果单条数据过大或者数据量过多都会降低效率），更无论读到内存之后的排序了。
>
> 但是如果我们对该字段建立索引`alter table 表名 add index(字段名)`，那么由于索引本身是有序的，因此直接按照索引的顺序和映射关系逐条取出数据即可。而且如果分页的，那么只用**取出索引表某个范围内的索引对应的数据**，而不用像上述那**取出所有数据**进行排序再返回某个范围内的数据。（从磁盘取数据是最影响性能的）

##### join

> 对join语句匹配关系（on）涉及的字段建立索引能够提高效率

##### 索引覆盖

> 如果要查询的字段都建立过索引，那么引擎会直接在索引表中查询而不会访问原始数据（否则只要有一个字段没有建立索引就会做全表扫描），这叫索引覆盖。因此我们需要尽可能的在`select`后
>
> 只写必要的查询字段，以增加索引覆盖的几率。
>
> 这里值得注意的是不要想着为每个字段建立索引，因为优先使用索引的优势就在于其体积小。

#### 语法细节

> 在满足索引使用的场景下（where/order by/join on或索引覆盖），索引也不一定被使用

##### 字段要独立出现

> 比如下面两条SQL语句在语义上相同，但是第一条会使用主键索引而第二条不会。
>
> ```sql
> select * from user where id = 20-1;
> select * from user where id+1 = 20;
> ```

##### like查询不能以通配符开头

```sql
# id_card 设置了索引 该条语句不走索引
select * from innodb1 where id_card like '%123%'
# id_card 设置了索引 该条语句走索引
select * from innodb1 where id_card like '123%';

mysql> explain select * from innodb1 where id_card like '%123%';
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | innodb1 | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   16 |    11.11 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set (0.10 sec)

mysql> explain select * from innodb1 where id_card like '123%';
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-----------------------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | innodb1 | NULL       | range | id_card       | id_card | 75      | NULL |    3 |   100.00 | Using index condition |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-----------------------+
1 row in set (0.12 sec)
```

##### 复合索引只对第一个字段有效

* 建立复合索引：

  ```sql
  alter table person add index(first_name,last_name);
  ```

* 其原理就是将索引先按照从`first_name`中提取的关键字排序，如果无法确定先后再按照从`last_name`提取的关键字排序，也就是说该索引表只是按照记录的first_name字段值有序。因此`select * from person where first_name = ?`是可以利用索引的，而`select * from person where last_name = ?`无法利用索引。

* 适用于**组合查询**的场景

  ```sql
  select * person from first_name = ? and last_name = ?
  ```

##### or两边条件都有索引可用

> 一但有一边无索引可用就会导致整个SQL语句的全表扫描

##### 状态值，不容易使用到索引

> 如性别、支付状态等状态值字段往往只有极少的几种取值可能，这种字段即使建立索引，也往往利用不上。这是因为，一个状态值可能匹配大量的记录，这种情况MySQL会认为利用索引比全表扫描的效率低，从而弃用索引。索引是随机访问磁盘，而全表扫描是顺序访问磁盘

* 

### 索引优化

#### 单表查询优化

* **全值匹配很快捷**

  ```sql
  --建立复合索引（age, deptId, name）
  CREATE INDEX idx_emp_ade ON t_emp(age, deptId, NAME);
  
  --查找
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90;
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90 AND deptId = 1;
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90 AND deptId = 1 AND name = '风清扬';
  
  --和上一条SQL语句中WHERE后字段的顺序不同，但是不影响查询结果
  EXPLAIN SELECT empno FROM t_emp WHERE deptId = 1 AND name = '风清扬' AND age = 90;
  ```

  * **全值匹配很快捷指的是，查询的字段按照顺序在索引中都可以匹配到**

* **最佳左前缀法则**

  ```sql
  --先删除之前创建的单值索引
  DROP INDEX idx_dept_id ON t_emp; 
  
  --查询，未按照最佳左前缀法则
  EXPLAIN SELECT empno FROM t_emp WHERE deptId = 1;
  EXPLAIN SELECT empno FROM t_emp WHERE deptId = 1 AND name = '风清扬';
  
  --查询，部分按照最佳左前缀法则（age字段和复合索引匹配，但name没有）
  EXPLAIN SELECT empno FROM t_emp WHERE  age = 90 AND name = '风清扬';
  
  --查询，完全按照最佳左前缀法则
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90 AND deptId = 1;
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90 AND deptId = 1 AND name = '风清扬';
  ```

  * **过滤条件要使用索引必须按照索引建立时的顺序，依次满足，一旦跳过某个字段，索引后面的字段都无法被使用**

* **索引列上不计算**

  > 不在索引列上做任何操作（计算、函数、(自动 or 手动)类型转换），**可能会导致索引失效而转向全表扫描**

  ```sql
  --直接查询
  EXPLAIN SELECT empno FROM t_emp WHERE age = 90 AND deptId = 1 AND NAME = '风清扬';
  
  --使用MySQL函数查询
  EXPLAIN SELECT empno FROM t_emp WHERE LEFT(age,2) = 90 AND deptId = 1 AND name = '风清扬';
  ```

* **范围之后全失效**

  ```sql
  --范围查询
  EXPLAIN SELECT empno FROM t_emp WHERE age > 50 AND deptId = 1 AND name = '风清扬';
  EXPLAIN SELECT empno FROM t_emp WHERE age = 50 AND deptId > 1 AND NAME = '风清扬';
  
  --未使用范围查询
  EXPLAIN SELECT empno FROM t_emp WHERE age = 50 AND deptId = 1 AND name = '风清扬';
  ```

  * **建议：**将可能做范围查询的字段的索引顺序**放在最后**
  * **结论：使用范围查询后，如果范围内的记录过多，会导致索引失效**，因为从自定义索引映射到主键索引需要耗费太多的时间，反而不如全表扫描来得快

* **覆盖索引多使用**

  ```sql
  --查询所有字段
  EXPLAIN SELECT * FROM t_dept WHERE id = 1;
  
  --查询索引字段
  EXPLAIN SELECT id FROM t_dept WHERE id = 1;
  ```

  * **结论：使用覆盖索引（Using index）会提高检索效率**

* **使用不等会失效**

  > 在使用**不等于(!= 或者<>)时**，有时会无法使用索引会导致全表扫描

  ```sql
  --SQL语句中有不等于
  EXPLAIN SELECT * FROM t_emp WHERE age != 90;
  EXPLAIN SELECT * FROM t_emp WHERE age <> 90;
  
  --SQL语句中没有不等于
  EXPLAIN SELECT * FROM t_emp WHERE age = 90;
  ```

* **使用NULL值要小心**

  > 在使用`IS NULL` 或者 `IS NOT NULL`时，可能会导致索引失效,但是如果**允许字段为空**，则
  >
  > * IS NULL 不会导致索引失效
  > * IS NOT NULL 会导致索引失效

  ```sql
  EXPLAIN SELECT * FROM t_emp WHERE age IS NULL;
  
  EXPLAIN SELECT * FROM t_emp WHERE age IS NOT NULL;
  ```

* **模糊查询加右边**

  > 要使用模糊查询时，**百分号最好加在右边，而且进行模糊查询的字段必须是单值索引**

  ```sql
  --创建单值索引
  CREATE INDEX idx_emp_name ON t_emp(NAME);
  
  --进行模糊查询
  EXPLAIN SELECT * FROM t_emp WHERE name LIKE '%风';
  EXPLAIN SELECT * FROM t_emp WHERE name LIKE '风%';
  EXPLAIN SELECT * FROM t_emp WHERE name LIKE '%风%';
  ```

  ```sql
  -- 有时必须使用其他类型的模糊查询，这时就需要用覆盖索引来解决索引失效的问题
  EXPLAIN SELECT name FROM t_emp WHERE name LIKE '%风';
  EXPLAIN SELECT name FROM t_emp WHERE name LIKE '风%';
  
  EXPLAIN SELECT NAME FROM t_emp WHERE name LIKE '%风%';
  ```

  * **结论：对索引进行模糊查询时，最好在右边加百分号。必须在左边或左右加百分号时，需要用到覆盖索引来提升查询效率**

* **字符串加单引号**

  > 当字段为字符串时，查询时必须带上单引号。否则**会发生自动的类型转换**，从而发生全表扫描

  ```sql
  --使用了单引号
  EXPLAIN SELECT card_id FROM person WHERE card_id = '1';
  
  --未使用单引号，发生自动类型转换
  EXPLAIN SELECT card_id FROM person WHERE card_id = 1;
  ```

* **尽量不用or查询**

  > 如果使用or，可能导致索引失效。所以要减少or的使用，可以**使用 union all 或者 union 来替代**

  ```sql
  --使用or进行查询
  EXPLAIN SELECT * FROM t_emp WHERE age = 90 OR NAME = '风清扬';
  ```

* 口诀

  > 全职匹配我最爱，最左前缀要遵守
  >
  > 带头大哥不能死，中间兄弟不能断
  >
  > 索引列上少计算，范围之后全失效
  >
  > LIKE 百分写最右，覆盖索引不写*
  >
  > 不等空值还有 OR，索引影响要注意
  >
  > VARCHAR 引号不可丢，SQL 优化有诀窍

#### 关联查询优化

* **LEFT JOIN优化**

  * 在优化关联查询时，只有在**被驱动表上建立索引才有效**
  * left join 时，左侧的为驱动表，**右侧为被驱动表**

* **INNER JOIN优化**

  * **结论**：inner join 时，**mysql 会把小结果集的表选为驱动表**（小表驱动大表）

    **所以最好把索引建立在大表（数据较多的表）上**

* **RIGHT JOIN优化**

  * 优化类型和LEFT JOIN类似，只不过被驱动表变成了左表

#### 排序分组优化

> 在查询中难免会对查询结果进行排序操作。进行排序操作时要**避免出现 Using filesort**，应使用索引给排序带来的方便

* **ORDER BY 优化**

  * **结论**：

    要想在排序时使用索引，避免 Using filesort，首先需要发生**索引覆盖**，其次

    - ORDER BY 后面字段的顺序要和复合索引的**顺序完全一致**
    - ORDER BY 后面的索引必须按照顺序出现，**排在后面的可以不出现**
    - 要进行升序或者降序时，**字段的排序顺序必须一致**。不能一部分升序，一部分降序，可以都升序或者都降序
    - 如果复合索引前面的**字段作为常量**出现在过滤条件中，**排序字段可以为紧跟其后的字段**

#### MySQL的排序算法

当发生 Using filesort 时，MySQL会根据自己的算法对查询结果进行排序

- **双路排序**
  - MySQL 4.1 之前是使用双路排序,字面意思就是**两次扫描磁盘**，最终得到数据，读取行指针和 order by 列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出
  - 从磁盘取排序字段，在 buffer 进行排序，再从磁盘取其他字段
  - 简单来说，**取一批数据，要对磁盘进行了两次扫描**，众所周知，I\O 是很耗时的，所以在 mysql4.1 之后，出现了第二种改进的算法，就是单路排序
- **单路排序**
  - 从磁盘读取查询需要的所有列，按照 order by 列**在 buffer 对它们进行排序**，然后扫描排序后的列表进行输出， 它的效率更快一些，避免了第二次读取数据。并且把随机 IO 变成了顺序 IO,但是它会使用更多的空间， 因为它把每一行都保存在内存中了
  - **存在的问题**：在 sort_buffer 中，方法 B 比方法 A 要多占用很多空间，因为方法 B 是把所有字段都取出, 所以有可能**取出的数据的总大小超出了 sort_buffer 的容量**，导致每次只能取 sort_buffer 容量大小的数据，进行排序（创建 tmp 文件，多 路合并），排完再取取 sort_buffer 容量大小，再排……从而多次 I/O。也就是**本来想省一次 I/O 操作，反而导致了大量的 I/O 操作，反而得不偿失**
- **优化Using filesort**
  - 增大 sort_butter_size 参数的设置
    - 不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对**每个进程的 1M-8M 之间调整**
  - 增大 max_length_for_sort_data 参数的设置
    - mysql 使用单路排序的前提是**排序的字段大小要小于 max_length_for_sort_data**
    - 提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出 sort_buffer_size 的概率就增大， 明显症状是高的磁盘 I/O 活动和低的处理器使用率。（1024-8192 之间调整）
  - 减少 select 后面的查询的字段
    - 查询的字段减少了，缓冲里就能容纳更多的内容了，**间接增大了sort_buffer_size**

### B树与B+树

##### 区别

  - B树的**关键字和记录是放在一起的**，叶子节点可以看作外部节点，不包含任何信息；B+树的非叶子节点中只有关键字和指向下一个节点的索引，**记录只放在叶子节点中**
  - 在 B树中，越靠近根节点的记录查找时间越快，只要找到关键字即可确定记录的存在；而 B+树中每个记录 的查找时间基本是一样的，都需要从根节点走到叶子节点，而且在叶子节点中还要再比较关键字。从这个角度看 B树的性能好像要比 B+树好，而在实际应用中却是 B+树的性能要好些。因为 B+树的非叶子节点不存放实际的数据， 这样每个节点可容纳的元素个数比 B树多，树高比 B树小，这样带来的好处是减少磁盘访问次数。尽管 B+树找到 一个记录所需的比较次数要比 B树多，但是一次磁盘访问的时间相当于成百上千次内存比较的时间，因此实际中 B+树的性能可能还会好些，而且 B+树的叶子节点使用指针连接在一起，方便顺序遍历（例如查看一个目录下的所有 文件，一个表中的所有记录等），这也是很多数据库和文件系统使用 B+树的缘故
