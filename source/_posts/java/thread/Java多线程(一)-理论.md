---
title: Java多线程(一)-理论
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 理论
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

* [并发与并行的区别是什么?](https://www.zhihu.com/question/33515481)
* Java并发编程实战
* 极客时间:Java并发编程实战
* [必懂系列！Java并发面试题](https://mp.weixin.qq.com/s/lEXTsIZ8JqYZVIUes22cDA)

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

<img src="http://www.chenjunlin.vip/img/java/thread/concurrent_parallel.jpg" alt="img" style="zoom:80%;" />

#### 弱并发

* 表现: 限制并发调用的数量,并非可用处理器资源,而是应用程序自身结构.

### 同步和异步

* 同步: 需要等待结果返回,才能继续运行;
* 异步: 无需等待结果返回,就能继续运行;

### 临界区与竞态条件

#### 临界区

> 在同一程序中运行多个线程本身不会导致问题，问题在于多个线程访问了相同的资源。如，同一内存区（变量，数组，或对象）、系统（数据库，web services等）或文件。实际上，这些问题只有在一或多个线程向这些资源做了写操作时才有可能发生，只要资源没有发生变化,多个线程读取相同的资源就是安全的。
>
> **临界资源**：**一次仅允许一个进程使用的资源**。例如：物理设备中的打印机、输入机和进程之间共享的变量、数据。
>
> **临界区**：每个进程中，**访问临界资源的**那段**代码**。

#### 竞态条件

> 多个线程在临界区内执行,由于代码的执行序列不同而导致结果无法预测,称之为发生了竞态条件;
>
> 当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。

#### 避免临界区的竞态条件的方式的解决方案

* 阻塞式的解决方案
  * `synchronized`
  * `Lock`
* 非阻塞式的解决方案
  * 原子变量

### 变量的线程安全分析

#### 成员变量和静态变量是否线程安全?

* 若它们没有共享,则是线程安全的;
* 若它们被共享,根据它们的状态是否能够改变,又分为两种情况:
  * 若只有读操作,则线程安全;
  * 若既有读操作又有写操作,则这段代码是临界区,需要考虑线程安全问题;

#### 局部变量是否线程安全?

* 局部变量是线程安全的;

* 但局部变量引用的对象,则未必

  * 若该对象没有逃离方法的作用范围,它们是线程安全的;

  * 若该对象逃离了方法的作用范围,则需要考虑线程安全;

    ```java
    // 不是线程安全的。虽然sb对象是在方法内生成的对象。但是sb作为一个返回变量返回，其他线程可以去拿取它，对它进行并发的操作
    public static StringBuilder method() {
    	StringBuilder sb = new StringBuilder();
    	sb.append(1);
    	sb.append(2);
    	return sb;
    }
    ```

#### Java内存模型定义了以下八种操作来完成

- lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。
- unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

> 如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作，如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。
>
> 也就是read和load之间，store和write之间是可以插入其他指令的，如对主内存中的变量a、b进行访问时，可能的顺序是read a，read b，load b， load a。

##### Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：

- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。
- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
- 一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
- 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。
- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。

### 设计模式-保护性暂停

* 同步模式之保护性暂停即(Guarded Suspension),用在一个线程等待另一个线程的执行结果;

* **要点**

  * 有一个结果需要从一个线程传递到另一个线程,让他们关联同一个`GuardedObject`;
  * 如果有结果不断从一个线程到另一个线程那么可以使用消息队列(生产者/消费者);
  * `JDK`中`join`的实现,`Future`的实现,采用此模式;
  * 因为要等待,另一方的结果,因此归类到同步模式;

  ![img](http://www.chenjunlin.vip/img/java/thread/%E4%BF%9D%E6%8A%A4%E6%80%A7%E6%9A%82%E5%81%9C.png)

* 示例

  ```java
  import com.google.common.collect.Lists;
  import lombok.extern.slf4j.Slf4j;
  
  import java.util.List;
  
  @Slf4j
  public class GuardedObject {
      private Object response;
      private Object lock = new Object();
  
      public Object get() {
          synchronized (lock) {
              // 条件不满足则等待
              while (response == null) {
                  try {
                      lock.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
              return response;
          }
      }
  
      public void complete(Object response) {
          synchronized (lock) {
              // 条件满足,通知等待线程
              this.response = response;
              lock.notifyAll();
          }
      }
  
      public static void main(String[] args) {
          GuardedObject guardedObject = new GuardedObject();
          new Thread(() -> {
              List<String> response = download();
              guardedObject.complete(response);
          }).start();
          // 主线程阻塞等待
          Object response = guardedObject.get();
          log.info("get response:[{}] lines", ((List<String>) response).size());
      }
  
      public static List<String> download() {
          List<String> list = Lists.newArrayList();
          list.add("GuardedObject");
          try {
              log.info("等待两秒");
              Thread.sleep(2000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          log.info("获取数据");
          return list;
      }
  }
  ```

  ```java
      /**
       * Waits at most {@code millis} milliseconds for this thread to
       * die. A timeout of {@code 0} means to wait forever.
       *
       * <p> This implementation uses a loop of {@code this.wait} calls
       * conditioned on {@code this.isAlive}. As a thread terminates the
       * {@code this.notifyAll} method is invoked. It is recommended that
       * applications not use {@code wait}, {@code notify}, or
       * {@code notifyAll} on {@code Thread} instances.
       *
       * @param  millis
       *         the time to wait in milliseconds
       *
       * @throws  IllegalArgumentException
       *          if the value of {@code millis} is negative
       *
       * @throws  InterruptedException
       *          if any thread has interrupted the current thread. The
       *          <i>interrupted status</i> of the current thread is
       *          cleared when this exception is thrown.
       */
      public final synchronized void join(long millis)
      throws InterruptedException {
          long base = System.currentTimeMillis();
          long now = 0;
  
          if (millis < 0) {
              throw new IllegalArgumentException("timeout value is negative");
          }
  
          if (millis == 0) {
              while (isAlive()) {
                  wait(0);
              }
          } else {
              while (isAlive()) {
                  long delay = millis - now;
                  if (delay <= 0) {
                      break;
                  }
                  wait(delay);
                  now = System.currentTimeMillis() - base;
              }
          }
      }
  ```

#### 异步模式之生产者消费者

* **要点**

  * 与之前的保护性暂停中的`GuardedObject`不同,不需要产生结果和消费结果一一对应;
  * 消费队列可以用来平衡生产和消费的线程资源;
  * 生产者仅负责生产结果数据,不关心数据该如何处理,而消费者专心处理结果数据;
  * 消息队列是有容量限制,满时不会再加入数据,空时不会再消耗数据;
  * `JDK`中各种阻塞队列,采用的就是这种模式;

  <img src="http://www.chenjunlin.vip/img/java/thread/%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85.png" alt="img" style="zoom:67%;" />

### 设计模式-固定运行顺序

#### `wait/notify`版

```java
package com.holelin.sundry.test.thread;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class SequentialOutputTest {
    public static void main(String[] args) {
        waitAndNotify();
    }

    /**
     * 用来同步的对象
     */
    static Object object = new Object();
    /**
     * t2运行标志,表示t2是否执行过
     */
    static boolean t2Runned = false;

    public static void waitAndNotify() {
        Thread t1 = new Thread(() -> {
            synchronized (object) {
                while (t2Runned) {
                    try {
                        object.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            log.info("1");
        });
        Thread t2 = new Thread(() -> {
            log.info("2");
            synchronized (object) {
                // 修改运行标记
                t2Runned = true;
                object.notifyAll();
            }
        });
        t1.start();
        t2.start();
    }
}

```

* 首先,需要保证先`wait`在`notify`,否则`wait`线程永远得不到唤醒,因此使用**运行标记**来判断该不该`wait`;
* 如果有些干扰线程错误的`notify`了`wait`线程,条件不满足时还需要重新等待,使用while循环来解决此问题;
* 唤醒对象上的`wait`线程需要使用`notifyAll,`因为同步对象上的等待线程可能不止一个

#### `park/unpark`版

```java
    public static void parkAndUnpark() {
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 当没有许可时,当前线程暂停运行,有许可时,用掉这个许可,当前线程恢复运行
            LockSupport.park();
            log.info("1");
        });
        Thread t2 = new Thread(() -> {
            log.info("2");
            // 给线程t1发放许可.(多次连续调用unpark,只会发放一个许可)
            LockSupport.unpark(t1);

        });
        t1.start();
        t2.start();
    }
```

#### 交替输出

```java
package com.holelin.sundry.test.thread;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class AlternateOutputTest {
    public static void main(String[] args) {
        waitAndNotify();
    }

    public static void waitAndNotify() {
        SyncWaitNotify syncWaitNotify = new SyncWaitNotify(1, 5);
        new Thread(() -> {
            syncWaitNotify.print(1, 2, "a");
        }).start();
        new Thread(() -> {
            syncWaitNotify.print(2, 3, "b");
        }).start();
        new Thread(() -> {
            syncWaitNotify.print(3, 1, "c");
        }).start();
    }

    static class SyncWaitNotify {
        private int flag;
        private int loopNumber;

        public SyncWaitNotify(int flag, int loopNumber) {
            this.flag = flag;
            this.loopNumber = loopNumber;
        }

        private void print(int waitFlag, int nextFlag, String str) {
            for (int i = 0; i < loopNumber; i++) {
                synchronized (this) {
                    while (this.flag != waitFlag) {
                        try {
                            this.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    log.info(str);
                    flag = nextFlag;
                    this.notifyAll();
                }
            }
        }
    }
}
23:04:11.123 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:04:11.125 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:04:11.125 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:04:11.125 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:04:11.125 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:04:11.125 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:04:11.125 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:04:11.125 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:04:11.125 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:04:11.125 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:04:11.125 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:04:11.125 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:04:11.125 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:04:11.125 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:04:11.125 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
```

```java
    static class AwaitSignal extends ReentrantLock {
        private int loopNumber;

        public AwaitSignal(int loopNumber) {
            this.loopNumber = loopNumber;
        }

        public void start(Condition first) {
            this.lock();
            try {
                log.info("start");
                first.signal();
            } finally {
                this.unlock();
            }
        }

        public void print(Condition current, Condition next, String str) {
            for (int i = 0; i < loopNumber; i++) {
                this.lock();
                try {
                    current.await();
                    log.info(str);
                    next.signal();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    this.unlock();
                }
            }
        }

    }
    public static void awaitSignal() {
        AwaitSignal as = new AwaitSignal(5);
        Condition aWaitSet = as.newCondition();
        Condition bWaitSet = as.newCondition();
        Condition cWaitSet = as.newCondition();
        new Thread(() -> {
            as.print(aWaitSet, bWaitSet, "a");
        }).start();
        new Thread(() -> {
            as.print(bWaitSet, cWaitSet, "b");
        }).start();
        new Thread(() -> {
            as.print(cWaitSet, aWaitSet, "c");
        }).start();
        as.start(aWaitSet);

    }

23:14:26.140 [main] INFO com.holelin.sundry.test.thread.AlternateOutputTest - start
23:14:26.141 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:14:26.142 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:14:26.142 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:14:26.142 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:14:26.142 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:14:26.142 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:14:26.142 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:14:26.142 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:14:26.142 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:14:26.142 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:14:26.142 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:14:26.142 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:14:26.142 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:14:26.142 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:14:26.142 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
```

```java
 static class SyncPark {
        private int loopNumber;
        private Thread[] threads;

        public SyncPark(int loopNumber) {
            this.loopNumber = loopNumber;
        }

        public void setThreads(Thread... threads) {
            this.threads = threads;
        }

        public void print(String str) {
            for (int i = 0; i < loopNumber; i++) {
                LockSupport.park();
                log.info(str);
                LockSupport.unpark(nextThread());
            }
        }

        private Thread nextThread() {
            Thread currentThread = Thread.currentThread();
            int index = 0;
            for (int i = 0; i < threads.length; i++) {
                if (threads[i] == currentThread) {
                    index = i;
                    break;
                }
            }
            if (index < threads.length - 1) {
                return threads[index + 1];
            } else {
                return threads[0];
            }
        }

        public void start() {
            for (Thread thread : threads) {
                thread.start();
            }
            LockSupport.unpark(threads[0]);
        }
    }
    
    public static void syncPark() {
        SyncPark syncPark = new SyncPark(5);
        Thread t1 = new Thread(() -> {
            syncPark.print("a");
        });
        Thread t2 = new Thread(() -> {
            syncPark.print("b");
        });
        Thread t3 = new Thread(() -> {
            syncPark.print("c");
        });
        syncPark.setThreads(t1, t2, t3);
        syncPark.start();
    }
23:21:08.881 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:21:08.882 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:21:08.882 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:21:08.883 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:21:08.883 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:21:08.883 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:21:08.883 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:21:08.883 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:21:08.883 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:21:08.883 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:21:08.883 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:21:08.883 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
23:21:08.883 [Thread-0] INFO com.holelin.sundry.test.thread.AlternateOutputTest - a
23:21:08.883 [Thread-1] INFO com.holelin.sundry.test.thread.AlternateOutputTest - b
23:21:08.883 [Thread-2] INFO com.holelin.sundry.test.thread.AlternateOutputTest - c
```

### 常见问题

#### `notify()`和`notifyAll()`有什么区别？

* 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁.
* 当有线程调用了对象的 `notifyAll()`方法（唤醒所有 wait 线程）或 `notify()`方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而`notifyAll`会将该对象等待池内的所有线程移动到锁池中，等待锁竞争。
* 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 `synchronized `代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁

#### `sleep()`和 `wait()`有什么区别?

* 对于`sleep()`方法，我们首先要知道该方法是属于`Thread`类中的。而`wait()`方法，则是属于`Object`类中的;
* `sleep()`方法导致了程序暂停执行指定的时间，让出CPU该其他线程，但是他的监控状态依然保持者，当指定的时间到了又会自动恢复运行状态。在调用sleep()方法的过程中，线程不会释放对象锁;
* 当调用`wait()`方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用`notify()`方法后本线程才进入对象锁定池准备，获取对象锁进入运行状态;

#### 什么是Daemon线程？它有什么意义？

* Java语言自己可以创建两种进程“用户线程”和“守护线程”
  * 用户线程：就是我们平时创建的普通线程.
  * 守护线程：主要是用来服务用户线程
* Daemon就是守护线程，他的意义是：
  * 只要当前`JVM`实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着`JVM`一同结束工作。
  * `Daemon`的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 `GC` (垃圾回收器)，它就是一个很称职的守护者。

#### `synchronized`和`java.util.concurrent.locks.Lock`的异同？

* 主要相同点：Lock能完成Synchronized所实现的所有功能。
* 主要不同点：Lock有比Synchronized更精确的线程予以和更好的性能。Synchronized会自动释放锁，但是Lock一定要求程序员手工释放，并且必须在finally从句中释放。

#### `SynchronizedMap`和`ConcurrentHashMap`有什么区别？

* `SynchronizedMap()`和`Hashtable`一样，实现上在调用map所有方法时，都对整个map进行同步。而`ConcurrentHashMap`的实现却更加精细，它对map中的所有桶加了锁。所以，只要有一个线程访问map，其他线程就无法进入map，而如果一个线程在访问`ConcurrentHashMap`某个桶时，其他线程，仍然可以对map执行某些操作。
* 所以，`ConcurrentHashMap`在性能以及安全性方面，明显比`Collections.synchronizedMap()`更加有优势。同时，同步操作精确控制到桶，这样，即使在遍历map时，如果其他线程试图对map进行数据修改，也不会抛出`ConcurrentModificationException`。

#### `CopyOnWriteArrayList`可以用于什么应用场景？

* `CopyOnWriteArrayList`的特性是针对读操作，不做处理，和普通的`ArrayList`性能一样。而在写操作时，会先拷贝一份，实现新旧版本的分离，然后在拷贝的版本上进行修改操作，修改完后，将其更新至就版本中;
* 那么他的使用场景就是：一个需要在多线程中操作，并且频繁遍历。其解决了由于长时间锁定整个数组导致的性能问题，解决方案即写时拷贝。
* 另外需要注意的是`CopyOnWrite`容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用`CopyOnWrite`容器

#### **什么是线程局部变量？**

> 当使用ThreadLocal维护变量时,ThreadLocal为每个使用该变量的线程提供独立的变量副本,每个线程都可以独立地改变自己的副本,而不会影响其它线程所对应的副本,是线程隔离的。线程隔离的秘密在于ThreadLocalMap类(ThreadLocal的静态内部类)
>
> 线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java 提供 ThreadLocal 类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。
>
> ThreadLocal的方法：void set(T value)、T get()以及T initialValue()。
>
> ThreadLocal是如何为每个线程创建变量的副本的：
>
> 首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。
>
> 总结：
>
> a、实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的
>
> b、为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；
>
> c、在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法

#### **Java 中，编写多线程程序的时候你会遵循哪些最佳实践？**

* 给线程命名，这样可以帮助调试。

* 最小化同步的范围，而不是将整个方法同步，只对关键部分做同步。

* 如果可以，更偏向于使用 volatile 而不是 synchronized。

* 使用更高层次的并发工具，而不是使用 wait() 和 notify() 来实现线程间通信，如 BlockingQueue，CountDownLatch 及 Semeaphore。

* 优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。
