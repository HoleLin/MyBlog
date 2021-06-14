---
title: Java多线程-volatile
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 多线程
- volatile
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

#### volatile

> * 获取共享变量时,为了保证变量的可见性,需求使用`volatile`修饰;
> * 它可以用来修饰**成员变量**和**静态变量**,可以避免线程从自己的工作缓存中查找变量的值,必须到主存中获取他的值,线程操作`volatile`变量都是直接操作主存,即一个线程对`volatile`变量的修改,对另一个线程可见;
>   * `volatile`仅仅保证了共享变量的可见性,让其他线程能够看到最新值,但不能解决指令交错问题(不能保证原子性);
> * CAS必须借助`volatile`才能读取共享变量的最新值来实现比较-交换的效果;

##### 内存配置

* 可见性
  * 写屏障(Sfence)保证在该屏障之前的,对共享变量的改动,都同步到主存中.
  * 读屏障(Ifence)保证在该屏障之后,对共享变量的读取,加载的是主存中最新的数据
* 有序性
  * 写屏障会保证指令重排序时,不会将写屏障之前的代码排在写屏障之后;
  * 读屏障会保证指令重排序时,不会将读屏障之后的代码排在读屏障之前;

##### volatile原理

* volatile的底层原理是内存屏障Memory Bairrier(Memory Fence)

  * 对volatile变量的写指令后加写屏障;
  * 对volatile变量的读指令前加读屏障;

###### 如何保证可见性

* 写屏障(Sfence)保证在该屏障之前的,对共享变量的改动,都同步到主存中.

  ```java
  public void actor2(I_Result r){
  	num = 2;
  	// ready 是volatile赋值带写屏障
  	ready = true;
  	// 写屏障
  }
  ```

* 读屏障(Ifence)保证在该屏障之后,对共享变量的读取,加载的是主存中最新的数据

  ```java
  public void actor1(I_Result r){
  	// 读屏障
  	// ready 是volatile读取值带读屏障
  	if(ready){
  		r.r1 = num+num;
  	} else{
  		r.r1 = 1;
  	}
  }
  ```

  <img src="http://www.chenjunlin.vip/img/thread/volatile%E8%AF%BB%E5%86%99%E5%B1%8F%E9%9A%9C.png" alt="img" style="zoom: 67%;" />

##### volatile不能解决指令交错的问题

* 写屏障仅仅是保证之后的读能读到最新的结果,但不能保证读跑到它前面去;
* 而有序性的保证也是保证了本线程内相关代码不能被重排序;

