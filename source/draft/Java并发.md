---
title: Java并发
date: 2021-06-07 23:11:45
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- 多线程
categories:
- Java
---

### 参考文献

* [并发与并行的区别是什么?](https://www.zhihu.com/question/33515481)
* Java并发编程实战
* [Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)

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

### happens-before规则

> happens-before规则规定了**对共享变量的写操作对其他线程可见**,它是可见性与有序性的一套规则总结,抛开happens-before规则,JVM并不能保证一个线程对共享变量的写,可让其他线程对该共享变量的读可见;

#### 线程解锁m之前对变量的写,对接下来对m加锁的其他线程对该变量的读可见

#### 线程对`volatile`变量的写,对接下来其他线程对该变量读可见

#### 线程`start`前对变量的写,对该线程开始后对该变量的读可见

#### 线程结束前对变量的写,对其他线程得之它结束后的读可见

> 比如其他线程调用t1.isAlive()或t1.join()等待它结束

#### 线程t1打断t2(interrupt)前对变量的写,对于其他线程得知t2被打断后对变量的读可见

> 通过t2.interrupted()或t2.isinterrupted()

### 锁

#### Java对象头

* 普通对象

  ```
  |--------------------------------------------------------------|
  |                     Object Header (64 bits)                  |
  |------------------------------------|-------------------------|
  |        Mark Word (32 bits)         |    Klass Word (32 bits) |
  |------------------------------------|-------------------------|
  ```

* 数组对象

  ```
  |---------------------------------------------------------------------------------|
  |                                 Object Header (96 bits)                         |
  |--------------------------------|-----------------------|------------------------|
  |        Mark Word(32bits)       |    Klass Word(32bits) |  array length(32bits)  |
  |--------------------------------|-----------------------|------------------------|
  ```

* Java的对象头由以下三部分组成

  > * Mark Word
  >
  > * 指向类的指针
  >
  > * 数组长度（只有数组对象才有）

  | biased_lock | lock | 状态     |
  | ----------- | ---- | -------- |
  | 0           | 01   | 无锁     |
  | 1           | 01   | 偏向锁   |
  | 0           | 00   | 轻量级锁 |
  | 0           | 10   | 重量级锁 |
  | 0           | 11   | GC标记   |

  ```
  |-------------------------------------------------------|--------------------|
  |                  Mark Word (32 bits)                  |       State        |
  |-------------------------------------------------------|--------------------|
  | identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |       Normal       |
  |-------------------------------------------------------|--------------------|
  |  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |       Biased       |
  |-------------------------------------------------------|--------------------|
  |               ptr_to_lock_record:30          | lock:2 | Lightweight Locked |
  |-------------------------------------------------------|--------------------|
  |               ptr_to_heavyweight_monitor:30  | lock:2 | Heavyweight Locked |
  |-------------------------------------------------------|--------------------|
  |                                              | lock:2 |    Marked for GC   |
  |-------------------------------------------------------|--------------------|
  
  |------------------------------------------------------------------------------|--------------------|
  |                                  Mark Word (64 bits)                         |       State        |
  |------------------------------------------------------------------------------|--------------------|
  | unused:25 | identity_hashcode:31 | unused:1 | age:4 | biased_lock:1 | lock:2 |       Normal       |
  |------------------------------------------------------------------------------|--------------------|
  | thread:54 |       epoch:2        | unused:1 | age:4 | biased_lock:1 | lock:2 |       Biased       |
  |------------------------------------------------------------------------------|--------------------|
  |                       ptr_to_lock_record:62                         | lock:2 | Lightweight Locked |
  |------------------------------------------------------------------------------|--------------------|
  |                     ptr_to_heavyweight_monitor:62                   | lock:2 | Heavyweight Locked |
  |------------------------------------------------------------------------------|--------------------|
  |                                                                     | lock:2 |    Marked for GC   |
  |------------------------------------------------------------------------------|--------------------|
  ```

  

#### 管程(Moitor)

> 每个Java对象都可以关联一个Monitor对象,
>
> 如果使用`synchronized`给对象上锁(重量级锁)之后,该对象的Mark Word中就被设置指向Monitor对象的指针
>
> Moitor内部属性
>
> * WaitSet
> * EntryList
> * Owner

##### Java实现管程的方式: `synchronized`

> 采用互斥的方式让同一时刻至多只有一个线程能持有对象锁,其他线程再想获得这个对象锁是就会被阻塞;
>
> 虽然在Java中互斥和同步都可以采用`synchronized`关键字来实现,但是它们还是有区别的:
>
> * 互斥是避免临界区的竞态条件发生,同一时刻只能有一个线程执行临街区代码;
> * 同步是由于线程执行的先后顺序不同,需要一个线程等待其他线程运行到某个点;

* `synchronized`的作用范围

  * 作用于成员变量和非静态方法上,锁住的是对象的实例,即this;

    ```java
      	public synchronized void testThread(){
            
        }
    	==> 
        public  void testThread(){
            synchronized(this){
            }
        }
    ```

  * 作用于静态方法时,锁住的是Class实例;

    ```java
        public synchronized static void testThread(){
            
        }
    	==> 
        public void testThread(){
            synchronized(Test.class){
            }
        }
    ```

  * 作用于一个代码块时,锁住的是所有代码块中配置的对象

* `synchronized`的使用

  ```java
      // synchronized实际上是用对象锁保证了临界区代码的原子性
      synchronized(对象){
          // 临界区
      }
  ```

#### 锁的分类

<img src="http://www.chenjunlin.vip/img/thread/lock.png" alt="锁的分类" style="zoom: 67%;" />

#### 乐观锁与悲观锁

##### 悲观锁

> 对于同一个数据的并发操作,悲观锁认为自己在使用数据的时候一定有别的线程来修改数据,因此在获取数据的时候会先加锁,确保数据不会被别的线程修改.Java中,`synchronized`关键字和`Lock`的实现类都是悲观锁.

* **处理流程**
  * 多个线程尝试获取同步资源的锁(给同步资源加锁);
  * 某个线程加锁成功并执行操作,其他线程则需要等待;
  * 获取锁的线程操作完成之后会释放锁,然后CPU唤醒等待的线程,其他线程竞争来获取锁;
  * 获取锁之后再执行自己的操作;

##### 乐观锁

> 乐观锁则认为自己在使用数据时没哟别的线程修改,所以不会添加锁,只是在更新数据的时候去判断之前有没有别的线程更新了这个数据.如果这个数据没有被更新,当前线程将自己修改的数据成功写入.如果数据已经被其他线程更新,则根据不同的实现方式执行不同的操作(报错或者自动重试).

* 乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的

##### 使用场景

* 悲观锁适合写操作多的场景,先加锁保证写操作时数据正确;
* 乐观锁适合读操作多的场景,不加锁的特点能够使其读操作的性能大幅度提升;

##### 示例

```java
// ------------------------- 悲观锁的调用方式 -------------------------
// synchronized
public synchronized void testMethod() {
	// 操作同步资源
}
// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); // 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
	lock.lock();
	// 操作同步资源
	lock.unlock();
}
```

```java
// ------------------------- 乐观锁的调用方式 -------------------------
// 需要保证多个线程使用的是同一个AtomicInteger
private AtomicInteger atomicInteger = new AtomicInteger();  
//执行自增1
atomicInteger.incrementAndGet(); 
```

#### volatile

* 获取共享变量时,为了保证变量的可见性,需求使用`volatile`修饰;
* 它可以用来修饰成员变量和静态变量,可以避免线程从自己的工作缓存中查找变量的值,必须到主存中获取他的值,线程操作`volatile`变量都是直接操作主存,即一个线程对`volatile`变量的修改,对另一个线程可见;
  * `volatile`仅仅保证了共享变量的可见性,让其他线程能够看到最新值,但不能解决指令交错问题(不能保证原子性);
* CAS必须借助`volatile`才能读取共享变量的最新值来实现比较-交换的效果;

#### CAS

> Compare And Swap 比较与交换,是一种无锁算法.在不适用锁(没有线程被阻塞)的情况下实现多线程之间的变量同步.`java.util.concurrent`包中的原子类就是通过CAS来实现了乐观锁。
>
> CAS的底层是`lock cmpxing`指令(x86架构),在单核CPU和多核CPU下都能保证比较-交换的原子性.
>
> 在多核状态下,某个执行到待`lock`的指令,CPU会让总线锁住,当这个核把此执行执行完毕,再开启总线,这个过程中不会被线程调度机制打断,保证了多线程对内存操作的准确性.

* CAS算法涉及到三个操作数

  * 需要读写的内存值V
  * 进行比较的值A
  * 要写入的新值B

  > 当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

CAS虽然很高效，但是它也存在三大问题，这里也简单说一下：

* **ABA问题**

  > CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。
  >
  > - JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

* **循环时间长开销大**。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

* **只能保证一个共享变量的原子操作**

  > * 对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
  >
  > - Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

#### 可重入(Reentrancy)

> **可重入锁又名递归锁**，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞
>
> 当一个线程请求其他线程已经占有的锁时,请求线程将被阻塞,然而内部锁`synchronized`是可重入的,因此线程在试图获取它自己占有的锁时,请求会成功.

* 可重入意味着所有的请求是基于"每个线程(per-thread)",而不是基于"每个调用(pre-invocation)"的.
* 可重入实现是通过每个锁关联**一个请求计数器(acquisition count)**和**一个占有它的线程**.当计数为0时,认为锁是未被占有的.线程请求一个未被占有的锁时,JVM将记录锁的占有者,并且将请求计数置为1.若同一线程再次请求这个锁,计数将递增,每次占用线程退出同步块,计数器值将递减,直到计数器达到0时,锁被释放.

**优点:** 

* 可重入锁的一个优点是可一定程度避免死锁

#### 互斥锁(mutual exclusion local)

> 也称为mutex.
>
> 意味着至多只有一个线程可以拥有锁,当线程A尝试请求一个被线程B占有的锁时,线程A必须等待或者阻塞,直到B线程释放这个锁,.若线程B永远不释放锁,A将永远等待下去.

#### 自旋锁和适应性自旋锁

##### 自旋锁



##### 适应性自选锁

