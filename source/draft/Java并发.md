---
title: Java并发
date: 2021-06-07 23:11:45
index_img: /img/cover/Java.jpg
tags:
- 多线程
categories:
- Java
---

### 参考文献

* [并发与并行的区别是什么?](https://www.zhihu.com/question/33515481)

### 进程与线程

* 进程基本上是相互独立的,而线程存在与进程内,是进程的一个子集;
* 进程拥有共享的资源,如内存空间等,供其内部的线程共享;
* 进程间通信较为复杂
  * 同一台计算机的进程通信称为IPC(Inter-process commuication);
  * 不同计算机之间的进程通信,需要通过网络,并遵守共同的协议,如HTTP
* 线程通信相对简单,因为它们共享进程内的内存(多个线程可以访问同一个共享变量);
* 线程更轻量,线程上下文切换成本一般要比进程上下文切换低;

### 并行与并发

> 单核CPU下,线程实际上还是串行执行的.操作系统中有一个组件做任务调度器,将CPU的时间片(Windows下时间片最小为15毫秒)分给不同的线程使用,只是由于CPU在线程间(时间片很短)的切换的非常快,人类感觉是同时运行的.
>
> * 总结为一句话: **微观串行,宏观并行**

* 一般会将这种线程轮流使用CPU的做法称为**并发(Concurrent)**;
* 多核CPU下,每个核Core都可以调度运行线程,这时候线程可以**并行(Parallel)**;
* **并发(Concurrent)是同一时间应对(dealing with)多件事情的能力;**
* **并行(Parallel)是同一时间动手做(doing)多件事情的能力;**

> 「并发」强调的是可以一起「出『发』」，「并行」强调的是可以一起「执『行』」
>
> * 顺序：上一个开始执行的任务完成后，当前任务才能开始执行,
>
> - 并发：无论上一个开始执行的任务是否完成，当前任务都可以开始执行
>
> （也就是说，A B 顺序执行的话，A 一定会比 B 先完成，而并发执行则不一定。）
>
> 与可以一起执行的并行（parallel）相对的是不可以一起执行的串行（serial）：
>
> * 串行：有一个任务执行单元，从物理上就只能一个任务、一个任务地执行
>
> * 并行：有多个任务执行单元，从物理上就可以多个任务一起执行
>
> （也就是说，在任意时间点上，串行执行时必然只有一个任务在执行，而并行则不一定。）
>
> 综上，并发与并行并不是互斥的概念，只是前者关注的是任务的抽象调度、后者关注的是任务的实际执行。而它们又是相关的，比如并行一定会允许并发。

<img src="http://www.chenjunlin.vip/img/thread/concurrent_parallel.jpg" alt="img" style="zoom:80%;" />

### 线程运行的原理

> 每个线程启动后,JVM(Java Virtual Machine Java虚拟机)就会为其分配一块内存.
>
> * 每个栈由多个栈帧(Frame)组成,对应着每次方法调用时所占用的内存;
>
> * 每个新城只能有一个活动栈帧,对应着当前正在执行的那个方法;

### 线程的创建和运行

* 直接使用`Thread`

  ```java
  Thread t= new Thread(()->{}).start();
  ```

* `Runnable`配合`Thread`

  ```java
  Runnable runnable = () -> log.debug("hellow");
  Thread t = new Thread(runnable);
  t.start;
  ```

* `FutureTask`配合`Thread`

  ```java
  // FutureTask能接收Callable类型的参数,用来处理返回结果
  FutureTasl<String> task = new FutureTask<>(()->{
  	log.debug("hello");
  	return "demo";
  })
  new Thread(task,"t").start();
  // 主线程阻塞,同步等待task执行完毕的结果
  String result = task.get();
  ```

  