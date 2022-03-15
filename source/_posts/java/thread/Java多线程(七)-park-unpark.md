---
title: Java多线程(七)-park/unpark
mermaid: true
date: 2021-06-17 22:58:31
cover: /img/cover/Java.jpg
tags:
- 多线程
- park/unpark
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

### park/unpark

#### 基本使用

```java
// 暂停当前线程  WAIT
LockSupport.park();
// 恢复某个线程 可以在park之前用,也可以在之后用
LockSupport.unpark(暂停线程对象);
```

#### 与`Object`的`wait`&`notify`相比

* `wait`,`notify`和`notifyAll`必须配合`Object Monitor`一起使用,而`unpark`不必;
* `park&unpark`是以线程为单位来阻塞的唤醒线程,而`notify`只能随机唤醒一个等待线程,`notifyAll`是唤醒所有等待线程,就不那么精确;
* `park&unpark`可以先`unpark`,而`wait&notify`不能先`notify`

#### `park&unpark`原理

* 每个线程都有自己的一个`Parker`对象,由三部分组成`_counter`,`_cond`和`_mutex`打个比喻
  * 线程就像一个旅人,`Parker`就像他随身携带的背包,条件变量就好比背包中的帐篷,`_counter`就好比背包中的备用干粮(0为耗尽,1为充足);
  * 调用`park`就是要看需不需要停下来歇息
    * 如果备用干粮耗尽,那么钻进帐篷歇息;
    * 如果备用干粮充足,那么不需停留,继续前进;
  * 调用`unpark`,就好比令干粮充足
    * 如果这时线程还在帐篷,就唤醒让他继续前进;
    * 如果这时线程还在运行,那么下次调用`park`时,仅是消耗备用干粮,不需停留继续前进
      * 因为背包空间有限,多次调用`unpark`仅会补充一份备用干粮;

<img src="https://www.holelin.cn/img/java/thread/park_unpark1.png" alt="img" style="zoom: 67%;" />

> 1. `当前线程调用`Unsafe.park()`方法;
>
> 2. 检查`_counter`,本情况为0,获得`_mutex`互斥锁;
>
> 3. 线程进入`_cond`条件变量阻塞;
>
> 4. 设置`_counter`=0;

<img src="https://www.holelin.cn/img/java/thread/park_unpark2.png" alt="img" style="zoom: 67%;" />

> 1. 调用`Unsafe.unpark(Thread-0)`,设置`_counter`为1;
>
> 2. 唤醒`_cond`条件变量中的Thread-0;
>
> 3. Thread-0恢复运行;
>
> 4. 设置`_counter`为0

<img src="https://www.holelin.cn/img/java/thread/park_unpark3.png" alt="img" style="zoom: 67%;" />

> 1. 调用`Unsafe.unpark(Thread-0)`方法,设置`_counter`为1;
> 2. 当前线程调用`Unsafe.park()`方法;
> 3. 检查`_counter`,本情况为1,这时线程无需阻塞,继续运行;
> 4. 设置`_counter`为0;
