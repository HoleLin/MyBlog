---
title: Java多线程-ThreadLocal
mermaid: true
date: 2021-06-24 00:06:28
cover: /img/cover/Java.jpg
tags:
- ThreadLocal
- 多线程
categories:
- Java
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

* [【对线面试官】ThreadLocal](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247484118&idx=1&sn=9526a1dc0d42926dd9bcccfc55e6abc2&scene=21#wechat_redirect)

#### `ThreadLocal`

> 它能提供线程的局部变量,让每个线程都可以通过set/get来对这个局部变量进行操作,不会和其他线程的局部变量进行冲突,实现了线程的数据隔离;

* 一种解决多线程环境下成员变量的问题的方案，但是与线程同步无关。其思路是为每一个线程创建一个单独的变量副本，从而每个线程都可以独立地改变自己所拥有的变量副本，而不会影响其他线程所对应的副本
* ThreadLocal 不是用于解决共享变量的问题的，也不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制
* 四个方法
  * get()：返回此线程局部变量的当前线程副本中的值
  * initialValue()：返回此线程局部变量的当前线程的“初始值”
  * remove()：移除此线程局部变量当前线程的值
  * set(T value)：将此线程局部变量的当前线程副本中的值设置为指定值
* ThreadLocalMap
  * 实现线程隔离机制的关键
  * 每个Thread内部都有一个ThreadLocal.ThreadLocalMap类型的成员变量，该成员变量用来存储实际的ThreadLocal变量副本。
  * 提供了一种用键值对方式存储每一个线程的变量副本的方法，key为当前ThreadLocal对象，value则是对应线程的变量副本
* 注意点
  * ThreadLocal实例本身是不存储值，它只是提供了一个在当前线程中找到副本值得key
  * 是ThreadLocal包含在Thread中，而不是Thread包含在ThreadLocal中
* 内存泄漏问题
  * ThreadLocalMap:key 弱引用 value 强引用，无法回收
  * 显示调用remove()

