---
title: JPA-遇到的问题
date: 2021-08-26 15:19:47
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 问题
categories:
- JPA
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

#### JPA自动更新问题

##### 环境以及版本

```tcl
SpringBoot 2.2.2.RELEASE
MySQL      8.0.24
```

##### 示例1

* 描述: 不加`@Transactional`注解,两次查询,第一次查询修改实体的某个值
* 结果: 两次查询的结果一致

```java
  	@GetMapping("/test3")
    public void test3() {
        Optional<TestJpa> byId = testDAO.findById(36);
        if (byId.isPresent()) {
            TestJpa test = byId.get();
            // id=36, x=0, y=71, z=53
            log.info("test: {}", test);
            test.setX(123);
        }
        TestJpa test2 = testDAO.findById(36).get();
        log.info("test2: {}", test2);
    } 
```

![img](https://www.holelin.cn/img/spring/jpa/JPA%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%E7%A4%BA%E4%BE%8B1.png)

##### 示例2

* 描述: 相对示例1,多了个`@Transactional`
* 结果: 两次查询的结果不一致,第一次查询后修改的值,在方法结束前自动更新了数据库

```java
    @Transactional
    @GetMapping("/test4")
    public void test4() {
        Optional<TestJpa> byId = testDAO.findById(36);
        if (byId.isPresent()) {
            TestJpa test = byId.get();
            // id=36, x=0, y=71, z=53
            log.info("test: {}", test);
            test.setX(123);
        }
        TestJpa test2 = testDAO.findById(36).get();
        log.info("test2: {}", test2);
    }
```

![img](https://www.holelin.cn/img/spring/jpa/JPA%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%E7%A4%BA%E4%BE%8B2.png)

###### 分析

* Hibernate对象生命周期

  <img src="https://www.holelin.cn/img/spring/jpa/Hibernate%E5%AF%B9%E8%B1%A1%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png" alt="img" style="zoom:67%;" />

* 第一查询到的数据test为持久态,修改X后还是持久态,当第二次查询相同的记录的时候,是直接从持久态上下文获取的.

  ![img](https://www.holelin.cn/img/spring/jpa/JPA%E8%8E%B7%E5%8F%96%E6%8C%81%E4%B9%85%E6%80%81%E7%9A%84%E5%80%BC.png)

* 最后在方法结束前提交事务前会做下列操作

  ```tcl
  Execute all SQL (and second-level cache updates) in a special order so that foreign-key constraints cannot be violated:
  Inserts, in the order they were performed
  Updates
  Deletion of collection elements
  Insertion of collection elements
  Deletes, in the order they were performed
  Params:
  session – The session being flushed
  ```

  ![img](https://www.holelin.cn/img/spring/jpa/beforeTransactionCompletion.png)

  ![img](https://www.holelin.cn/img/spring/jpa/flushBeforeTransactionCompletion.png)

  ![img](https://www.holelin.cn/img/spring/jpa/doFlush.png)

  ![img](https://www.holelin.cn/img/spring/jpa/onFlush.png)

  ![img](https://www.holelin.cn/img/spring/jpa/performExecutions.png)

##### 示例3

* 描述: 相对示例2,多了`entityManager.unwrap(org.hibernate.Session.class).evict(test);`
* 结果和示例1相似,两次查询结果一致

```java
    @Transactional
    @GetMapping("/test5")
    public void test5() {
        Optional<TestJpa> byId = testDAO.findById(36);
        if (byId.isPresent()) {
            TestJpa test = byId.get();
            // id=36, x=0, y=71, z=53
            log.info("test: {}", test);
            test.setX(123);
            entityManager.unwrap(org.hibernate.Session.class).evict(test);
        }
        TestJpa test2 = testDAO.findById(36).get();
        log.info("test2: {}", test2);
    }
```

![img](https://www.holelin.cn/img/spring/jpa/JPA%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%E7%A4%BA%E4%BE%8B3.png)

