---
title: MySQL事务
mermaid: true
date: 2021-06-12 19:43:29
cover: /img/cover/MySQL.jpg
tags:
- 事务
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

### 事务

> 事务支持在引擎层实现的

#### ACID

* Atomicity 原子性
* Consistency 一致性
* Isolation 隔离性
* Durability 持久性

#### 事务中可能出现的问题

* 脏读(dirty read)
* 不可重复读(non-repeatable read)
* 幻读(phantom read)

#### SQL标准的事务隔离级别

* 读未提交(read uncommited)

  > 一个事务还没提交时,它做的变更就能被别的事务看到;
  >
  > 别人改数据的事务尚未提交,我在我的事务中也能读到;

* 读提交(read commited)

  > 一个事务提交之后,他的变更才能被其他事务看到;
  >
  > 别人改数据的事务已提交,我在我的事务中才能读到;

* 可重复读(repeatable read)

  > 一个事务执行过程中看到的数据,总是跟这个事务在启动时看到的数据是一致.当然在可重复读隔离级别,未提交变更对其他事务也是不可见的;
  >
  > 别人改数据的事务已经提交,我在我的事务中也不去读;

* 串行(serializable)

  > 对于同一行记录,"写"会加"写锁","读"会加"读锁".当出现读写锁冲突的时候,后访问的事务必须等前一个事务执行完成,才能继续执行;
  >
  > 我的事务尚未提交,别人就不能改我的数据;

> Oracle默认隔离级别为"读提交"
>
> 在实现上,数据库里面会创建一个视图,访问的时候以视图的逻辑结果为准.
>
> * 在"可重复读"隔离级别下,这个视图是在事务启动是创建的,整个事务存在期间都用这个视图.
> * 在"读提交"隔离级别下,这个视图是在每个SQL语句开始执行的时候创建的.
> * 这里需要注意的是,"读未提交"隔离级别下直接返回记录上的最新值,没有视图的概念;
> * 而"串行化"隔离别下直接用加锁的方式来避免并行访问.

#### 设置事务隔离级别

> 参数设置: `trancsaction-isolation`设置为`READ-COMMITED`
>
> 事务的启动: 
>
> * 显示启动事务,`begin`或`start transaction`.配套的提交语句`commit`,回滚语句`rollback`;
> * `set autocommit=0`,这个命令会将线程的自动提交关掉;
