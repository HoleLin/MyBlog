---
title: Java多线程-线程操作
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
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

### 线程

#### 线程运行的原理

> 每个线程启动后,JVM(Java Virtual Machine Java虚拟机)就会为其分配一块内存.
>
> * 每个栈由多个栈帧(Frame)组成,对应着每次方法调用时所占用的内存;
> * 每个新城只能有一个活动栈帧,对应着当前正在执行的那个方法;

#### 线程的创建和运行

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

#### 线程上下文切换(Thread Context Switch)

* 以下一些原因会导致DPU不再执行当前的线程,转而执行另一个线程的代码
  * 线程的CPU时间片用完
  * 垃圾回收
  * 有更高优先级的线程需要运行
  * 线程自己调用`sleep`,`yield`,`wait`,`join`,`park`,`lock`等方法或者使用`synchronized`关键字
* 当Context Switch发生时,需要由操作系统保存当前线程的状态,并恢复另一个线程的状态,Java中对应的概念就是**程序计数器(Program Counter,Register)**,它的作用是记住下一条JVM指令的执行地址,是线程私有的;
  * 状态包括程序计数器,虚拟机栈中的每一个栈帧信息,其中包含局部变量,操作数栈,返回地址
  * Context Switch频繁会影响性能

#### 线程中常用的方法

| 方法名          | 是否是静态 | 功能说明                                                  | 注意点                                                       |
| --------------- | ---------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| start()         |            | 启动一个新的现场,在新的线程运行run方法中的代码            | start方法只能让线程进入就绪状态,里面的代码不一定立刻运行(因为CPU的时间片还没分给它),每个线程对象的start方法只能调用一次, |
| run()           |            | 新线程启动后会调用的方法                                  | 如果在构造Thread对象是传递了Runnable参数,则线程启动后调用Runnable中的run方法,否则默认不执行任何操作,但可以创建Thread的子类对象类覆盖默认行为 |
| join()          |            | 等待线程运行结果                                          |                                                              |
| join(long n)    |            | 等待线程运行结果,最多等待n毫秒                            |                                                              |
| isAlive()       |            | 线程是否存活(还没有运行完毕)                              |                                                              |
| interrupt()     |            | 打断线程                                                  | 如果被打断线程正在`sleep`,`wait`,`join`会导致被打断的线程抛出`interruptedException`,并清除`打断标记`;如果打断的正在运行的线程,则会设置`打断标记`;`park`的线程被打断,也会设置`打断标记` |
| interrupted()   | static     | 打断当前线程是否被打断                                    | 会清除`打断标记`                                             |
| currentThread() | static     | 获取当前正在执行的线程                                    |                                                              |
| getId()         |            | 获取线程长整型的id                                        | id唯一                                                       |
| getName()       |            | 获取线程名                                                |                                                              |
| setName()       |            | 获取线程优先级                                            |                                                              |
| setProity()     |            | 修改线程优先级                                            | Java中规定线程优先级是1~10的整数,较大的优先级能提高该线程被CPU调度的几率 |
| getState()      |            | 获取线程状态                                              | Java中线程状态是用6个enum表示,分别为`NEW`,`RUNNABLE`,`BLOCKED`,`WAITING`,`TIMED_WAITING`,`TERMINATED` |
| isInterrupted() |            | 判断是否被打断                                            | 不会清楚`打断标记`,返回值为true表示被打断了,返回值为false表示未打断 |
| sleep(long n)   | static     | 让当前执行的线程休眠n毫秒,休眠时让出CPU的时间片给其他线程 |                                                              |
| yield()         | static     | 提示线程调度器让出当前线程对CPU的使用                     | 主要是为了测试和调试                                         |

* tip: `notify`唤醒的线程并不会在执行notify的一瞬间重新运行,因为执行`notify`的那一瞬间执行`notify`的线程还持有锁,所以其他线程还无法获取这个实例的锁.

* **sleep的作用**

  * 防止CPU占用100%

    * 当没有利用CPU来计算时,不用让white(true)空转浪费CPU,这时可以使用`yield`或者`sleep`来让CPU的使用权给其他程序

    ```java
         while (true){
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    ```

    * 可以使用`wait`或者条件变量达到类似的效果,不同的是这两种都要加锁,并且需要相应的唤醒操作,一般适用与要进行同步的场景
    * `sleep`适用于无需锁同步的场景

* **sleep与yield的区别**

  > **sleep**
  >
  > * 调用`sleep`会让当前线程从Running进入Timed waiting的状态;
  > * 其他线程可以使用`interrupt`方法打断正在睡眠的进程,这对`sleep`方法会抛出`interruptedException`;
  > * 睡眠结束后的线程未必会立刻得到执行;
  > * 建议使用TimeUnit的sleep代替Thread的sleep来获得更好的可读性;
  >
  > **yield**
  >
  > * 调用`yield`会让当前线程从Running进入Runnable状态,然后调用执行其他同优先级的线程,若此时没有同优先级的线程.那么则不能保证产生让当前线程暂停的效果;
  > * 具体的实现依赖于操作系统的任务调用器;
  >
  > **线程优先级**
  >
  > * 线程优先级会提示(hint)调度器优先调度改线程,但它仅仅是一个提示,调度器可以忽略它;
  > * 若CPU较忙,那么优先级较高的线程会获得更多的时间片,但CPU闲时,优先级几乎没有作用;

* join方法

  ```java
  // 未加join方法
  @Slf4j
  public class ThreadTest {
      static int num = 0;
      @Test
      public void test() throws InterruptedException {
          log.info("开始");
          Thread t1 = new Thread(() -> {
              log.info("开始");
  
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              log.info("结束");
              num = 20;
          }, "t1");
          t1.start();
          log.info("结果为:{}", num);
          log.info("结束");
      }
  }
  // 输出
  22:56:17.706 [main] INFO com.holelin.sundry.common.ThreadTest - 开始
  22:56:17.746 [t1] INFO com.holelin.sundry.common.ThreadTest - 开始
  22:56:17.745 [main] INFO com.holelin.sundry.common.ThreadTest - 结果为:0
  22:56:17.747 [main] INFO com.holelin.sundry.common.ThreadTest - 结束
      
  // 加入join方法
  @Slf4j
  public class ThreadTest {
      static int num = 0;
      @Test
      public void testJoin() throws InterruptedException {
          log.info("开始");
          Thread t1 = new Thread(() -> {
              log.info("开始");
  
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              log.info("结束");
              num = 20;
          }, "t1");
          t1.start();
          t1.join();
          log.info("结果为:{}", num);
          log.info("结束");
      }
  }
  // 输出
  22:57:23.860 [main] INFO com.holelin.sundry.common.ThreadTest - 开始
  22:57:23.895 [t1] INFO com.holelin.sundry.common.ThreadTest - 开始
  22:57:24.907 [t1] INFO com.holelin.sundry.common.ThreadTest - 结束
  22:57:24.907 [main] INFO com.holelin.sundry.common.ThreadTest - 结果为:20
  22:57:24.908 [main] INFO com.holelin.sundry.common.ThreadTest - 结束
  ```

* **interrupt方法**

  * 打断`sleep`,`wait`,`join`的线程,这几个方法都会让线程进入阻塞阶段

  * 打断`sleep`的线程会清空`打断标记`

    ```java
        @Test
        public void testInterrupt() {
            Thread t = new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "t");
            t.start();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("打断状态:{}", t.isInterrupted());
        }
    //  输出
    23:01:04.734 [main] INFO com.holelin.sundry.common.ThreadTest - 打断状态:false
        @Test
        public void testInterrupt() {
            Thread t = new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "t");
            t.start();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t.interrupt();
            log.info("打断状态:{}", t.isInterrupted());
        }
    // 输出
    java.lang.InterruptedException: sleep interrupted
    	at java.lang.Thread.sleep(Native Method)
    	at java.lang.Thread.sleep(Thread.java:340)
    	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
    	at com.holelin.sundry.common.ThreadTest.lambda$testInterrupt$1(ThreadTest.java:37)
    	at java.lang.Thread.run(Thread.java:748)
    23:05:29.313 [main] INFO com.holelin.sundry.common.ThreadTest - 打断状态:false
    ```

  * **打断正常运行的线程**

    ```java
    // 打断正常运行的线程,不会清空打断标记
    	@Test
        public void testInterrupt2() {
            Thread t = new Thread(() -> {
            }, "t");
            t.start();
            t.interrupt();
            log.info("打断状态:{}", t.isInterrupted());
        }
    // 输出
    23:08:08.748 [main] INFO com.holelin.sundry.common.ThreadTest - 打断状态:true
    ```

  * **打断park线程**

    ```java
    // 打断park线程,不会清空打断标记
        @Test
        public void testPark() {
            Thread t = new Thread(() -> {
                log.info("park...");
                LockSupport.park();
                log.info("打断状态:{}", Thread.currentThread().isInterrupted());
            }, "t");
            t.start();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t.interrupt();
        }
    // 输出
    23:17:15.826 [t] INFO com.holelin.sundry.common.ThreadTest - park...
    23:17:16.838 [t] INFO com.holelin.sundry.common.ThreadTest - unpark...
    23:17:19.643 [t] INFO com.holelin.sundry.common.ThreadTest - 打断状态:true
        
    // 若打断标记已经是true,则park会失效,可以使用Thread.interrupted()清楚打断标记
        @Test
        public void testPark2() {
            Thread t = new Thread(() -> {
                for (int i = 0; i < 5; i++) {
                    log.info("park...");
                    LockSupport.park();
                    log.info("打断状态:{}", Thread.currentThread().isInterrupted());
                }
            }, "t");
            t.start();
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t.interrupt();
        }
    ```

  * "如果被打断线程正在`sleep`,`wait`,`join`会导致被打断的线程抛出`interruptedException`,并清除`打断标记`"证据

    ```java
        /**
         * Causes the currently executing thread to sleep (temporarily cease
         * execution) for the specified number of milliseconds, subject to
         * the precision and accuracy of system timers and schedulers. The thread
         * does not lose ownership of any monitors.
         *
         * @param  millis
         *         the length of time to sleep in milliseconds
         *
         * @throws  IllegalArgumentException
         *          if the value of {@code millis} is negative
         * // 出现异常时会清除打断标记
         * @throws  InterruptedException
         *          if any thread has interrupted the current thread. The
         *          <i>interrupted status</i> of the current thread is
         *          cleared when this exception is thrown.
         */
        public static native void sleep(long millis) throws InterruptedException;
      	/**
      	 * ...省略
         * @param      timeout   the maximum time to wait in milliseconds.
         * @throws  IllegalArgumentException      if the value of timeout is
         *               negative.
         * @throws  IllegalMonitorStateException  if the current thread is not
         *               the owner of the object's monitor.
         * // 出现异常时会清除打断标记
         * @throws  InterruptedException if any thread interrupted the
         *             current thread before or while the current thread
         *             was waiting for a notification.  The <i>interrupted
         *             status</i> of the current thread is cleared when
         *             this exception is thrown.
         * @see        java.lang.Object#notify()
         * @see        java.lang.Object#notifyAll()
         */
        public final native void wait(long timeout) throws InterruptedException;
    ```

#### 线程的六种状态

* **初始状态(NEW)**: 

  > 仅是在语言层面上创建了线程对象,还未与操作系统关联;
  >
  > 新创建了一个线程对象。

* **运行(RUNNABLE)**

  > Java线程中将就绪（READY）和运行中（RUNNING）两种状态笼统的称为“运行”。
  >
  > * 线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于**就绪状态（READY）**
  > * 就绪状态的线程在获得CPU时间片后变为运行中状态（RUNNING）。

* **阻塞状态(BLOCKED)**

  > 如果调用阻塞API,此时该线程实际上不会使用CPU会导致线程切换上下文进入阻塞状态,等BIO操作完毕后会由操作系统唤醒阻塞线程,状态由阻塞状态转变为就绪状态;
  >
  > 与就绪状态的区别是,对于阻塞状态的线程来说,只要它们一直不被唤醒,CPU就一直不会考虑调度它们;

* **等待(WAITING)**

  > 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）

* **超时等待(TIMED_WAITING)**

  > 该状态不同于WAITING，它可以在指定的时间后自行返回。

* **终止状态(TERMINATED)**

  > 表示线程已经执行完毕,生命周期已经结束,不会转变为其他状态了.

<img src="http://www.chenjunlin.vip/img/thread/thread_status_change.jpg" alt="img" style="zoom:80%;" />

### 两阶段终止模式(Two-Phase Termination Patter)

#### 错误的终止线程的方式

* 使用线程对象的`stop()`方法停止线程

  > stop()方法会真正杀死线程,如果这时线程锁住了共享资源,那么当它被杀死后就再也没有机会释放锁,其他线程将永远无法获取锁;

* 使用`System.exit(int)`方法

  > 目的是停止一个线程,但这种做法会让整个程序都停止;

#### 正确的终止线程的方式: 两阶段终止模式

* 利用`isInterrupted()`判断是否被打断, `interrupt()`可以打断正在执行的线程,无论整个线程是在`sleep`,`wait`还是正常运行

  ```java
      @Test
      public void testTwoPhaseTermination() {
          Thread t = new Thread(() -> {
              while (true) {
                  Thread currentThread = Thread.currentThread();
                  if (currentThread.isInterrupted()) {
                      log.info("料理后事");
                      break;
                  }
                  try {
                      Thread.sleep(1000);
                      log.info("保存结果");
                  } catch (InterruptedException e) {
                      currentThread.interrupt();
                  }
              }
          });
          t.start();
          try {
              Thread.sleep(3000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          log.info("准备终止线程");
          t.interrupt();
      }
  ```

* 利用停止标记

  ```java
  	// 停止标记用volatile是为了保证该变量在多个线程之间的可见性
  	// 例子中即主线程把它修改为true对t线程可见
  	private volatile boolean isStop = false;
  
      @Test
      public void testTwoPhaseTermination2() {
          Thread t = new Thread(() -> {
              while (true) {
                  Thread currentThread = Thread.currentThread();
                  if (isStop) {
                      log.info("料理后事");
                      break;
                  }
                  try {
                      Thread.sleep(1000);
                      log.info("保存结果");
                  } catch (InterruptedException e) {
                      currentThread.interrupt();
                  }
              }
          });
          t.start();
          try {
              Thread.sleep(3000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          log.info("准备终止线程");
          isStop = true;
          t.interrupt();
      }
  ```

### 主线程与守护线程

> 默认情况下,Java进程需要等待所有线程都运行结束才会结束.
>
> 有种特殊的线程叫做守护线程,
>
> * 只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；
>
> * 只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。
>
> Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它就是一个很称职的守护者。
>
> **注意点**
>
> * thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。
>
>   ```java
>       /**
>        * Marks this thread as either a {@linkplain #isDaemon daemon} thread
>        * or a user thread. The Java Virtual Machine exits when the only
>        * threads running are all daemon threads.
>        * // 在其中之前设置
>        * <p> This method must be invoked before the thread is started.
>        *
>        * @param  on
>        *         if {@code true}, marks this thread as a daemon thread
>        *
>        * @throws  IllegalThreadStateException
>        *          if this thread is {@linkplain #isAlive alive}
>        *
>        * @throws  SecurityException
>        *          if {@link #checkAccess} determines that the current
>        *          thread cannot modify this thread
>        */
>       public final void setDaemon(boolean on) {
>           checkAccess();
>           if (isAlive()) {
>               throw new IllegalThreadStateException();
>           }
>           daemon = on;
>       }
>   ```
>
> * 在Daemon线程中产生的新线程也是Daemon的。

#### 设置守护线程的方法

```java
t.setDaemon(true);
```

