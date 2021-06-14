---
title: Java多线程-Happens-Before规则
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- Happens-Before
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

* Java并发编程实战
* 极客时间:Java并发编程实战

### Happens-Before规则

> happens-before规则规定了**对共享变量的写操作对其他线程可见**,它是可见性与有序性的一套规则总结,抛开happens-before规则,JVM并不能保证一个线程对共享变量的写,可让其他线程对该共享变量的读可见;
>
> 真正要表达的是：**前面一个操作的结果对后续操作是可见的**

#### 程序的顺序性规则

> 这条规则是指在一个线程中，按照程序顺序，前面的操作 Happens-Before 于后续的任意操作。这还是比较容易理解的，比如刚才那段示例代码，按照程序的顺序，第 6 行代码 “x = 42;” Happens-Before 于第 7 行代码 “v = true;”，这就是规则 1 的内容，也比较符合单线程里面的思维：**程序前面对某个变量的修改一定是对后续操作可见的。**

```java
// 以下代码来源于【参考1】
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }
  public void reader() {
    if (v == true) {
      // 这里x会是多少呢？
    }
  }
}
```

#### volatile 变量规则

> 这条规则是指对一个 volatile 变量的写操作， Happens-Before 于后续对这个 volatile 变量的读操作。

#### 传递性

> 这条规则是指如果 A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C。
>
> <img src="http://www.chenjunlin.vip/img/thread/Happens-Before%E4%BC%A0%E9%80%92%E6%80%A7.png" alt="img" style="zoom:80%;" />
>
> 从图中，我们可以看到：
>
> * “x=42” Happens-Before 写变量 “v=true” ，这是规则 1 的内容；
>
> * 写变量“v=true” Happens-Before 读变量 “v=true”，这是规则 2 的内容 。
>
> 再根据这个传递性规则，我们得到结果：“x=42” Happens-Before 读变量“v=true”。
>
> 如果线程 B 读到了“v=true”，那么线程 A 设置的“x=42”对线程 B 是可见的。也就是说，线程 B 能看到 “x == 42” .这就是 1.5 版本对 volatile 语义的增强，这个增强意义重大，1.5 版本的并发工具包（java.util.concurrent）就是靠 volatile 语义来搞定可见性的

#### 管程中锁的规则

> 这条规则是指对一个锁的解锁 Happens-Before 于后续对这个锁的加锁。

* **管程**是一种通用的同步原语，在 Java 中指的就是 synchronized，synchronized 是 Java 里对管程的实现;

* 管程中的锁在 Java 里是隐式实现的，例如下面的代码，在进入同步块之前，会自动加锁，而在代码块执行完会自动释放锁，加锁以及释放锁都是编译器帮我们实现的。

  ```java
  synchronized (this) { //此处自动加锁
    // x是共享变量,初始值=10
    if (this.x < 12) {
      this.x = 12; 
    }  
  } //此处自动解锁
  ```

* 所以结合规则 4——管程中锁的规则，可以这样理解：假设 x 的初始值是 10，线程 A 执行完代码块后 x 的值会变成 12（执行完自动释放锁），线程 B 进入代码块时，能够看到线程 A 对 x 的写操作，也就是线程 B 能够看到 x==12。

#### 线程 start() 规则

> 这条是关于线程启动的。它是指主线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作。

* 换句话说就是，如果线程 A 调用线程 B 的 start() 方法（即在线程 A 中启动线程 B），那么该 start() 操作 Happens-Before 于线程 B 中的任意操作。具体可参考下面示例代码。

  ```java
  Thread B = new Thread(()->{
    // 主线程调用B.start()之前
    // 所有对共享变量的修改，此处皆可见
    // 此例中，var==77
  });
  // 此处对共享变量var修改
  var = 77;
  // 主线程启动子线程
  B.start();
  ```

####  线程 join() 规则

> 这条是关于线程等待的。它是指主线程 A 等待子线程 B 完成（主线程 A 通过调用子线程 B 的 join() 方法实现），当子线程 B 完成后（主线程 A 中 join() 方法返回），主线程能够看到子线程的操作。当然所谓的“看到”，指的是对**共享变量**的操作。

* 换句话说就是，如果在线程 A 中，调用线程 B 的 join() 并成功返回，那么线程 B 中的任意操作 Happens-Before 于该 join() 操作的返回。具体可参考下面示例代码。

  ```java
  Thread B = new Thread(()->{
    // 此处对共享变量var修改
    var = 66;
  });
  // 例如此处对共享变量修改，
  // 则这个修改结果对线程B可见
  // 主线程启动子线程
  B.start();
  B.join()
  // 子线程所有对共享变量的修改
  // 在主线程调用B.join()之后皆可见
  // 此例中，var==66
  ```

#### 程序中断规则

> 对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
