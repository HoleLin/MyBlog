---
title: MySQL(十八)-排序
date: 2021-12-14 13:30:58
cover: /img/cover/MySQL.jpg
tags:
- 排序
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

* 高性能MySQL(第三版)
* MySQL实战45讲

### 排序

* MySQL有两种方式可以生成有序的结果:通过**排序操作**或者**按索引顺序扫描**;

  * 若EXPLAIN出来的type列的值为"index",则说明MySQL使用了索引扫描来做排序.

* 扫描索引本身是很快的,因为只需要从一条索引记录移动到紧接着的下一条记录.但如果索引不能覆盖查询所需要的全部列,那就不得不每扫描一条索引记录都回表查询一次对应的行.这基本都是随机I/O,因此按索引顺序读取数据的速度通常要比顺序地全表扫描慢,尤其是在I/O密集型的工作负载时.

* **只有当索引的列顺序和ORDER BY子句的顺序完全一致,并且所有列的排序方向(倒序或正序)一样时**,MySQL才能够使用索引来对接过做排序.

  * 如果查询需要关联多张表,则只有当ORDER BY子句引用的字段全部为第一个表时,才能使用索引做排序.

  * ORDER BY子句和查找性查询的限制是一样的:需要满足索引的最左前缀的原则,否则MySQL都需要执行排序操作,而无法利用索引排序.

  * 有一种情况下ORDER BY子句可以不满足索引的最左前缀的要求,就是**前导列为常量的时候**,如果WHERE子句或者JOIN子句中对这些列指定了常量,就可以弥补索引的不足.

    ```mysql
    CREATE TABLE rental(
    	...
    	PRIMARY KEY (rental_id),
    	UNIQUE KEY rental_data(rental_date,invertory_id,customer_id),
    	KEY idx_fk_inventory_id(inventory_id),
    	KEY idx_fk_customer_id(customer_id),
    	KEY idx_fk_staff_id(staff_id),
    	...
    );
    
    EXPLAIN SELECT rental_id,staff_id FROM rental WHERE rental_data = '2021-05-09' ORDER BY inventory_id,customer_id;
    ```
  
* `sort_buffer_size`就是MySQL为排序开辟的内存(sort_buffer)的大小.如果要排序的数据量小于`sort_buffer_size`,排序就在内存中完成.但如果排序数据量太大,内存放不下,则不得不利用磁盘临时文件辅助排序.

* 确定一个排序语句是否使用了临时文件

  ```mysql
  /* 打开optimizer_trace，只对本线程有效 */
  SET optimizer_trace='enabled=on'; 
  
  /* @a保存Innodb_rows_read的初始值 */
  select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';
  
  /* 执行语句 */
  SELECT xxxx  FROM table WHERE xxx = xxx ORDER BY xxx limit 100;
  
  /* 查看 OPTIMIZER_TRACE 输出 */
  SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G
  
  /* @b保存Innodb_rows_read的当前值 */
  select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';
  
  /* 计算Innodb_rows_read差值 */
  select @b-@a;
  ```

  * 从`number_of_tmp_files`中可以看到是否使用了临时文件.
  * 如果`sort_buffer_size`超过了需要排序的数据量的大小,`number_of_tmp_files`就是0,表示后排序可以直接在内存中完成.否则就需要放在临时文件中排序.`sort_buffer_size`越小,需要分成的文件份数越多,`number_of_tmp_files`的值就越大.

#### MySQL的排序算法

* 如果需要排序的数据量小于"排序缓冲区",MySQL使用内存进行"快速排序"操作.如果内存不够排序,那么MySQL会先将数据分块,对每个独立块使用"快速排序"进行排序,并将各个块的排序结果存放在磁盘上,然后将各个排好序的块进行合并(merge),最后返回排序结果.

##### **双路排序**(两次传输排序)

- 读取行指针和需要排序的字段,对其进行排序,然后再根据排序结果读取所需要的数据行.
- 这需要进行两次数据传输,即需要从数据表中读取两次数据,第二次读取数据的时候,因为读取排序列进行排序后的所有记录,这会**产生大量的随机I/O**,所以两次数据传输的成本非常高.
- 优点: 在排序的时候存储尽可能少的数据,这就让"排序缓冲区"中可能容纳尽可能多的行数进行排序.

##### **单路排序**(单次传输排序)

* 先读取查询所需要的所有列,然后在根据给定列进行排序,最后直接返回排序结果.**在MySQL4.1及以后引入**.
* 优点: 不需要从数据表中读取两次数据,对于I/O密集型的应用,效率高了很多.相比两次传输排序,这个算法只需要一次**顺序I/O**读取所有的数据,无需任何的随机I/O.
* 缺点: 如果需要返回的列非常多,非常大,会额外占用大量的空间,而这些列对排序操作本身来说是没有任何作用的.因为单条排序记录很大,所以可能会更多的排序块需要合并.
* 当查询需要所有列的总长度不超过参数`max_length_for_sort_data`时,MySQL使用"单次传输排序",可以通过调整这个参数来影响MySQL排序算法的选择.

#### 关联查询中需要排序

* 若关联查询中需要排序,MySQL会分两种情况来处理这样的文件排序.
  * 如果`ORDER BY`子句中所有列都来自关联的第一个表,那么MySQL在关联处理第一个表的时候就进行文件排序.如果是这样,那么在MySQL的`EXPLAIN`结果中可以看到`Extra`字段会有"Using filesort".
  * 除此之外的所有情况,MySQL都会将先关联的结果存放到一个临时表中,然后在所有的关联都结束后,在进行文件排序.这种情况下,在MySQL的`EXPLAIN`结果的Extra字段可以看到"Using temporary;Using filesort".
  * 在MySQL5.6之前,如果查询中有`LIMIT`的话,`LIMIT`也会在排序之后应用,所以即使需要返回较少的数据,临时表和需要排序的数据量仍然会非常大;在MySQL5.6时,在这里做了改进,当只需要返回部分排序结果的时候,例如使用了`LIMIT`子句,MySQL不再对所有的结果进行排序,而是根据实际情况,选择抛弃不满足条件的结果,然后再进行排序.

#### **优化Using filesort**

- 增大 `sort_buffer_size` 参数的设置
  - 不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对**每个进程的 1M-8M 之间调整**
- 增大` max_length_for_sort_data` 参数的设置
  - mysql 使用单路排序的前提是**排序的字段大小要小于 max_length_for_sort_data**
  - 提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出 sort_buffer_size 的概率就增大， 明显症状是高的磁盘 I/O 活动和低的处理器使用率。（1024-8192 之间调整）
- 减少 select 后面的查询的字段
  - 查询的字段减少了，缓冲里就能容纳更多的内容了，**间接增大了sort_buffer_size**
