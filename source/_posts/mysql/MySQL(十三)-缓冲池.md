---
title: MySQL(十三)-缓冲池
date: 2021-08-06 10:42:31
cover: /img/cover/MySQL.jpg
tags:
- 缓冲池
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

#### 参考文献

* 极客时间--SQL必知必会(陈旸)

#### 数据库缓冲池

##### 缓冲池是如何读取数据的?

* 缓冲池管理器会尽量将经常使用的数据保存起来,在数据库在进行页读操作的时候,首先会判断该页是否在缓冲池中,如果存在就直接读取,如果不存在,就会通过内存或磁盘将页存放到缓冲池中,再进行读取.

* 缓存在数据库中的结构和作用如下图

  ![img](https://www.holelin.cn/img/mysql/%E7%BC%93%E5%86%B2%E6%B1%A0%E7%BB%93%E6%9E%84.png)

* 执行SQL语句的时候更新了缓冲池的数据,那么这些数据是否会同步到磁盘上?

  * 对数据库中的记录进行修改的时候,首先会修改缓冲池中页里面的记录信息,然后数据库会以一定的频率刷新到磁盘上.注意并不是发生更新操作,都会立即进行磁盘回写.缓冲池会采用与一种叫做`checkpoint`的机制将数据写到磁盘上,这样做的好处就是提升了数据库的整体性能.
  * 当缓冲池不够用时,相应释放掉一些不常用的页,就可以强行采用`checkpoint`的方式,将不常用的脏页写会到磁盘上,然后再从缓冲池中将这些页释放掉.
    * 脏页指的是缓冲池中被修改过的页,与磁盘上的数据页不一致.

#### 查看缓冲池的大小

* MySQL MyISAM存储引擎,它只缓存索引,不缓存数据,对应的键缓存参数为`key_buffer_size`;
* InnoDB存储引擎,可以通过查看`innodb_buffer_pool_size`变量查看缓冲池的大小;
  * 修改缓冲池大小`set global innodb_buffer_pool_size`
* 在InnoDB存储引擎中,可以同时开启多个缓冲池,查看缓冲池的个数`show variables like 'innodb_buffer_pool_instances'`
  * 若想要开启多个缓冲池,首先需要将`innodb_buffer_pool_size`参数设置为大于等于1G,这时`innodb_buffer_pool_instances`才会大于1.

#### 数据页加载的三种方式

* **内存读取**
  * 若数据在内存中,基本上执行时间在1ms左右;
* **随机读取**
  * 若数据没有在内存中,就需要在磁盘上对该页进行查找,整体时间预估在10ms左右,这10ms中有6ms是磁盘的实际繁忙时间(包括了寻道和半圈旋转时间),有3ms是对可能发生的排队时间的估计值,另外还有1ms的传输时间,将页从磁盘服务器缓冲区传输到数据库缓冲区中.
* **顺序读取**
  * 顺序读取其实是一种批量读取的方式,因为请求的数据在磁盘上往往都是相邻存储的,顺序读取可以批量读取页面.

#### 通过`last_query_cost`统计SQL语句的查询成本

* 若想要查看某条SQL语句的查询成本,可以在执行完这条SQL语句之后,通过查看当前会话中的`last_query_cost`变量值来得到当前查询的成本.`show status like 'last_query_cost';`

  ![img](https://www.holelin.cn/img/mysql/last_query_cost.png)
  * 上图表示查询了4个页
