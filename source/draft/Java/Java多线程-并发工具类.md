---
title: Java多线程-并发工具类
mermaid: true
date: 2021-06-24 00:07:27
cover: /img/cover/Java.jpg
tags:
- 并发工具类
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

### `Semaphore`

* 一个控制访问多个共享资源的计数器
* 从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动
* 信号量Semaphore是一个非负整数（>=1）。当一个线程想要访问某个共享资源时，它必须要先获取Semaphore，当Semaphore >0时，获取该资源并使Semaphore – 1。如果Semaphore值 = 0，则表示全部的共享资源已经被其他线程全部占用，线程必须要等待其他线程释放资源。当线程释放资源时，Semaphore则+1
* 应用场景
  * 通常用于限制可以访问某些资源（物理或逻辑的）的线程数目
* 内部采用共享锁实现

### `Exchanger`

* 可以在对中对元素进行配对和交换的线程的同步点
* 允许在并发任务之间交换数据。具体来说，Exchanger类允许在两个线程之间定义同步点。当两个线程都到达同步点时，他们交换数据结构，因此第一个线程的数据结构进入到第二个线程中，第二个线程的数据结构进入到第一个线程中

