---
title: Java多线程(五)-Java内存模型
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 多线程
- volatile
- JMM
categories:
- Java
- Thread
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

### Java内存模型(JMM)

* Java Memory Model

* JMM体现在一下几个方面
  * **原子性**: 保证指令不会受到线程上下文切换的影响;
  * **可见性**: 保证指令不受CPU缓存的影响;
    * 使用`volatile`关键字
  * **有序性**: 保证指令不受CPU指令并行优化影响;

#### volatile

* 获取共享变量时,为了保证变量的可见性,需求使用`volatile`修饰;
* 它可以用来修饰**成员变量**和**静态变量**,可以避免线程从自己的工作缓存中查找变量的值,必须到主存中获取他的值,线程操作`volatile`变量都是直接操作主存,即一个线程对`volatile`变量的修改,对另一个线程可见;
  * `volatile`仅仅保证了共享变量的可见性,让其他线程能够看到最新值,但不能解决指令交错问题(不能保证原子性);
* `CAS`必须借助`volatile`才能读取共享变量的最新值来实现比较-交换的效果;

##### 内存屏障

* 可见性
  * 写屏障(Sfence)保证在该屏障之前的,对共享变量的改动,都同步到主存中.
  * 读屏障(Ifence)保证在该屏障之后,对共享变量的读取,加载的是主存中最新的数据
* 有序性
  * 写屏障会保证指令重排序时,不会将写屏障之前的代码排在写屏障之后;
  * 读屏障会保证指令重排序时,不会将读屏障之后的代码排在读屏障之前;

##### volatile原理

* volatile的底层原理是内存屏障Memory Barrier(Memory Fence)

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

  <img src="https://www.holelin.cn/img/java/thread/volatile%E8%AF%BB%E5%86%99%E5%B1%8F%E9%9A%9C.png" alt="img" style="zoom: 67%;" />

###### 如何保证有序性

* 写屏障会确保指令重排序时,不会将写屏障之前的代码排在写屏障之后;
* 读屏障会确保指令重排序时,不会将读屏障之后的代码排在读屏障之前;

##### volatile不能解决指令交错的问题

* 写屏障仅仅是保证之后的读能读到最新的结果,但不能保证读跑到它前面去;
* 而有序性的保证也是保证了本线程内相关代码不能被重排序;

##### **volatile 类型变量提供什么保证？**

> volatile 变量提供顺序和可见性保证，例如，JVM 或者 JIT为了获得更好的性能会对语句重排序，但是 volatile 类型变量即使在没有同步块的情况下赋值也不会与其他语句重排序。volatile 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。某些情况下，volatile 还能提供原子性，如读 64 位数据类型，像 long 和 double 都不是原子的，但 volatile 类型的 double 和 long 就是原子的。

##### **volatile 修饰符的有过什么实践？**

> 一种实践是用 volatile 修饰 long 和 double 变量，使其能按原子类型来读写。double 和 long 都是64位宽，因此对这两种类型的读是分为两部分的，第一次读取第一个 32 位，然后再读剩下的 32 位，这个过程不是原子的，但 Java 中 volatile 型的 long 或 double 变量的读写是原子的。
>
> volatile 修复符的另一个作用是提供内存屏障（memory barrier），例如在分布式框架中的应用。简单的说，就是当你写一个 volatile 变量之前，Java 内存模型会插入一个写屏障（write barrier），读一个 volatile 变量之前，会插入一个读屏障（read barrier）。意思就是说，在你写一个 volatile 域时，能保证任何线程都能看到你写的值，同时，在写之前，也能保证任何数值的更新对所有线程是可见的，因为内存屏障会将其他所有写的值更新到缓存。

##### `volatile`与`synchronized`的区别

* `volatile`本质是告诉`JVM`当前变量在寄存器中的值是不确定的,需要从主存中读取,`synchronized`则是锁定当前变量,只有当前线程可以访问改变量,其他线程被阻塞住;
* `volatile`仅能使用变量级别,`synchronized`则可以使用在变量,方法上;
* `volatile`仅能实现变量的修改可见性,但不具备原子特性,而`synchronized`则可以保证变量的修改可见性和原子性;
* `volatile`不会造成线程阻塞,而`synchronized`可能会造成线程阻塞;
* `volatile`标记的变量不会被编译优化,而`synchronized`标记的变量可以被编译优化;

