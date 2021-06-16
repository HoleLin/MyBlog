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
* [JAVA多线程之线程间的通信方式](https://www.cnblogs.com/hapjin/p/5492619.html)
* [Java并发编程相关面试问题-含程序答案](https://blog.csdn.net/qq_34039315/article/details/78542498)

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

* 实现`Callable`接口通过`FutureTask`包装器来创建`Thread`线程；

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
  
* 使用`ExecutorService`、`Callable`、`Future`实现有返回结果的多线程（也就是使用了`ExecutorService`来管理前面的三种方式）

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

#### 死线程

* 有两个原因会导致线程死亡:
  * 因为`run`方法正常退出而自然死亡;
  * 因为一个未捕获的异常终止了`run`方法而使线程猝死;
* 为了确定线程当前是否还活着(就是要么可运行,要么被阻塞了),需要使用`isAlive`方法,如果线程是可运行或阻塞的,这个方法返回`true`,如果线程任旧处于new状态且是不可运行的,或者线程死亡false;

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

### 线程的属性

* 线程优先级
* 守护线程
* 线程组
* 处理未捕获异常的处理器

#### 线程优先级

> 默认情况下,一个线程继承它的父线程的优先级,一个线程的父线程就是启动它的那个线程;
>
> 线程优先级是高度依赖于系统的,因此,最好仅将线程优先级看作线程调度器的一个参考因素.

#### 主线程与守护线程

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

### 多线程之线程间的通信方式

#### 同步

> 这里讲的同步是指多个线程通过synchronized关键字这种方式来实现线程间的通信。

```java
public class MyObject {

    synchronized public void methodA() {
        //do something....
    }

    synchronized public void methodB() {
        //do some other thing
    }
}

public class ThreadA extends Thread {

    private MyObject object;
	//	省略构造方法
    @Override
    public void run() {
        super.run();
        object.methodA();
    }
}

public class ThreadB extends Thread {

    private MyObject object;
	// 省略构造方法
    @Override
    public void run() {
        super.run();
        object.methodB();
    }
}

public class Run {
    public static void main(String[] args) {
        MyObject object = new MyObject();

        // 线程A与线程B 持有的是同一个对象:object
        ThreadA a = new ThreadA(object);
        ThreadB b = new ThreadB(object);
        a.start();
        b.start();
    }
}
```

> 由于线程A和线程B持有同一个MyObject类的对象object，尽管这两个线程需要调用不同的方法，但是它们是同步执行的，比如：**线程B需要等待线程A执行完了methodA()方法之后，它才能执行methodB()方法。这样，线程A和线程B就实现了 通信。**
>
> **这种方式，本质上就是“共享内存”式的通信。多个线程需要访问同一个共享变量，谁拿到了锁（获得了访问权限），谁就可以执行。**

#### **while轮询的方式**

```java
import java.util.ArrayList;
import java.util.List;

public class MyList {

    private List<String> list = new ArrayList<String>();
    public void add() {
        list.add("elements");
    }
    public int size() {
        return list.size();
    }
}

import mylist.MyList;

public class ThreadA extends Thread {

    private MyList list;

    public ThreadA(MyList list) {
        super();
        this.list = list;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 10; i++) {
                list.add();
                System.out.println("添加了" + (i + 1) + "个元素");
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

import mylist.MyList;

public class ThreadB extends Thread {

    private MyList list;

    public ThreadB(MyList list) {
        super();
        this.list = list;
    }

    @Override
    public void run() {
        try {
            while (true) {
                if (list.size() == 5) {
                    System.out.println("==5, 线程b准备退出了");
                    throw new InterruptedException();
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

import mylist.MyList;
import extthread.ThreadA;
import extthread.ThreadB;

public class Test {

    public static void main(String[] args) {
        MyList service = new MyList();

        ThreadA a = new ThreadA(service);
        a.setName("A");
        a.start();

        ThreadB b = new ThreadB(service);
        b.setName("B");
        b.start();
    }
}
```

> 在这种方式下，线程A不断地改变条件，线程ThreadB不停地通过while语句检测这个条件(list.size()==5)是否成立 ，从而实现了线程间的通信。但是**这种方式会浪费CPU资源**。之所以说它浪费资源，是因为JVM调度器将CPU交给线程B执行时，它没做啥“有用”的工作，只是在不断地测试 某个条件是否成立。*就类似于现实生活中，某个人一直看着手机屏幕是否有电话来了，而不是： 在干别的事情，当有电话来时，响铃通知TA电话来了。*关于线程的轮询的影响
>
> 线程都是先把变量读取到本地线程栈空间，然后再去再去修改的本地变量。因此，如果线程B每次都在取本地的 条件变量，那么尽管另外一个线程已经改变了轮询的条件，它也察觉不到，这样也会造成死循环。

#### **wait/notify机制**

```java
import java.util.ArrayList;
import java.util.List;

public class MyList {

    private static List<String> list = new ArrayList<String>();

    public static void add() {
        list.add("anyString");
    }

    public static int size() {
        return list.size();
    }
}


public class ThreadA extends Thread {

    private Object lock;

    public ThreadA(Object lock) {
        super();
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            synchronized (lock) {
                if (MyList.size() != 5) {
                    System.out.println("wait begin "
                            + System.currentTimeMillis());
                    lock.wait();
                    System.out.println("wait end  "
                            + System.currentTimeMillis());
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


public class ThreadB extends Thread {
    private Object lock;

    public ThreadB(Object lock) {
        super();
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            synchronized (lock) {
                for (int i = 0; i < 10; i++) {
                    MyList.add();
                    if (MyList.size() == 5) {
                        lock.notify();
                        System.out.println("已经发出了通知");
                    }
                    System.out.println("添加了" + (i + 1) + "个元素!");
                    Thread.sleep(1000);
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Run {

    public static void main(String[] args) {

        try {
            Object lock = new Object();

            ThreadA a = new ThreadA(lock);
            a.start();

            Thread.sleep(50);

            ThreadB b = new ThreadB(lock);
            b.start();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

> 线程A要等待某个条件满足时(list.size()==5)，才执行操作。线程B则向list中添加元素，改变list 的size。
>
> A,B之间如何通信的呢？也就是说，线程A如何知道 list.size() 已经为5了呢？
>
> 这里用到了Object类的 wait() 和 notify() 方法。
>
> 当条件未满足时(list.size() !=5)，线程A调用wait() 放弃CPU，并进入阻塞状态。---不像②while轮询那样占用CPU
>
> 当条件满足时，线程B调用 notify()通知 线程A，所谓通知线程A，就是唤醒线程A，并让它进入可运行状态。
>
> 这种方式的一个好处就是CPU的利用率提高了。
>
> 但是也有一些缺点：比如，线程B先执行，一下子添加了5个元素并调用了notify()发送了通知，而此时线程A还执行；当线程A执行并调用wait()时，那它永远就不可能被唤醒了。因为，线程B已经发了通知了，以后不再发通知了。这说明：**通知过早，会打乱程序的执行逻辑。**

#### **管道通信**就是使用java.io.PipedInputStream 和 java.io.PipedOutputStream进行通信

> 具体就不介绍了。分布式系统中说的两种通信机制：共享内存机制和消息通信机制。感觉前面的①中的synchronized关键字和②中的while轮询 “属于” 共享内存机制，由于是轮询的条件使用了volatile关键字修饰时，这就表示它们通过判断这个“共享的条件变量“是否改变了，来实现进程间的交流。
>
> 而管道通信，更像消息传递机制，也就是说：通过管道，将一个线程中的消息发送给另一个。

### 实操

#### 如何让一段程序并发的执行，并最终汇总结果？

> 使用CyclicBarrier 在多个关口处将多个线程执行结果汇总， CountDownLatch 在各线程执行完毕后向总线程汇报结果。
>
> CountDownLatch : 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。
>
> CyclicBarrier : N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。
>
> 这样应该就清楚一点了，对于CountDownLatch来说，重点是那个“一个线程”, 是它在等待，而另外那N的线程在把“某个事情”做完之后可以继续等待，可以终止。而对于CyclicBarrier来说，重点是那N个线程，他们之间任何一个没有完成，所有的线程都必须等待。
>
> 从api上理解就是CountdownLatch有主要配合使用两个方法countDown()和await()，countDown()是做事的线程用的方法，await()是等待事情完成的线程用个方法，这两种线程是可以分开的(下面例子:CountdownLatchTest2)，当然也可以是同一组线程;CyclicBarrier只有一个方法await(),指的是做事线程必须大家同时等待，必须是同一组线程的工作。

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 各个线程执行完成后，主线程做总结性工作的例子
 * @author xuexiaolei
 * @version 2017年11月02日
 */
public class CountdownLatchTest2 {
    private final static int THREAD_NUM = 10;
    public static void main(String[] args) {
        CountDownLatch lock = new CountDownLatch(THREAD_NUM);
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < THREAD_NUM; i++) {
            exec.submit(new CountdownLatchTask(lock, "Thread-"+i));
        }
        try {
            lock.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("大家都执行完成了，做总结性工作");
        exec.shutdown();
    }

    static class CountdownLatchTask implements Runnable{
        private final CountDownLatch lock;
        private final String threadName;
        CountdownLatchTask(CountDownLatch lock, String threadName) {
            this.lock = lock;
            this.threadName = threadName;
        }
        @Override public void run() {
            System.out.println(threadName + " 执行完成");
            lock.countDown();
        }
    }
}
mport java.util.concurrent.*;

/**
 *
 * @author xuexiaolei
 * @version 2017年11月02日
 */
public class CyclicBarrierTest {
    private final static int THREAD_NUM = 10;
    public static void main(String[] args) {
        CyclicBarrier lock = new CyclicBarrier(THREAD_NUM, new Runnable() {
            @Override public void run() {
                System.out.println("这阶段大家都执行完成了，我总结一下，然后开始下一阶段");
            }
        });
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < THREAD_NUM; i++) {
            exec.submit(new CountdownLatchTask(lock, "Task-"+i));
        }
        exec.shutdown();
    }

    static class CountdownLatchTask implements Runnable{
        private final CyclicBarrier lock;
        private final String threadName;
        CountdownLatchTask(CyclicBarrier lock, String threadName) {
            this.lock = lock;
            this.threadName = threadName;
        }
        @Override public void run() {
            for (int i = 0; i < 3; i++) {
                System.out.println(threadName + " 执行完成");
                try {
                    lock.await();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}
```

#### 如何实现一个流控程序，用于控制请求的调用次数？

```java
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

/**
 * 阻塞访问的线程，直到获取了访问令牌
 * @author xuexiaolei
 * @version 2017年11月15日
 */
public class FlowControl2 {
    private final static int MAX_COUNT = 10;
    private final Semaphore semaphore = new Semaphore(MAX_COUNT);
    private final ExecutorService exec = Executors.newCachedThreadPool();

    public void access(int i){
        exec.submit(new Runnable() {
            @Override public void run() {
                semaphore.acquireUninterruptibly();
                doSomething(i);
                semaphore.release();
            }
        });
    }

    public void doSomething(int i){
        try {
            Thread.sleep(new Random().nextInt(100));
            System.out.println(String.format("%s 通过线程:%s 访问成功",i,Thread.currentThread().getName()));
        } catch (InterruptedException e) {
        }
    }

    public static void main(String[] args) {
        FlowControl2 web = new FlowControl2();
        for (int i = 0; i < 2000; i++) {
            web.access(i);
        }
    }
}
```

#### 多读少写的场景应该使用哪个并发容器，为什么使用它？比如你做了一个搜索引擎，搜索引擎每次搜索前需要判断搜索关键词是否在黑名单里，黑名单每天更新一次

> 用CopyOnWriteArrayList、CopyOnWriteArraySet。
>
> CopyOnWriteArrayList特性：
> CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

```java
import java.util.Collections;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.*;

/**
 * 做了一个搜索引擎，搜索引擎每次搜索前需要判断搜索关键词是否在黑名单里，黑名单每天更新一次。
 * @author xuexiaolei
 * @version 2017年11月03日
 */
public class SearchEngineBlackListCache {
    private final Set<String> blackList = new CopyOnWriteArraySet<>();//黑名单集合
    private final Set<String> todayBlackList = new ConcurrentSkipListSet<>();//当天添加的黑名单


    /******内部类单例写法 start******/
    private SearchEngineBlackListCache(){}
    private static class Holder {
        private static SearchEngineBlackListCache singleton = new SearchEngineBlackListCache();
    }
    public static SearchEngineBlackListCache getInstance(){
        return Holder.singleton;
    }
    /******内部类单例写法 end******/

    /**
     * 获取黑名单列表
     * @return
     */
    public Set<String> getBlackList() {
        return Collections.unmodifiableSet(blackList);
    }

    /**
     * 判断是否在黑名单内
     * @param name
     * @return
     */
    public boolean isBlack(String name){
        return blackList.contains(name);
    }

    /**
     * 将今天的黑名单加入到黑名单内，外部系统可以定时每天执行这个方法
     */
    public void mergeBlackList(){
        synchronized (todayBlackList){
            blackList.addAll(todayBlackList);
            todayBlackList.clear();
        }
    }

    /**
     * 加入黑名单
     * @param name
     */
    public void addBlackList(String name){
        synchronized (todayBlackList) {
            todayBlackList.add(name);
        }
    }

    /**
     * 随机生成50个线程来测试
     * @param args
     */
    public static void main(String[] args) {
        SearchEngineBlackListCache cache = SearchEngineBlackListCache.getInstance();
        final int COUNT = 50;
        CountDownLatch countDownLatch = new CountDownLatch(COUNT);
        ExecutorService exec = Executors.newFixedThreadPool(COUNT);
        for (int i = 0; i < COUNT; i++) {
            exec.execute(new Runnable() {
                @Override public void run() {
                    try {
                        countDownLatch.countDown();
                        countDownLatch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    cache.addBlackList(UUID.randomUUID().toString());//随机增加黑名单字符串
                    cache.mergeBlackList();
                    System.out.println(cache.getBlackList().size());
                }
            });
        }
        exec.shutdown();
    }
}
```

#### **你是如何调用 wait（）方法的？****使用 if 块还是循环？为什么？**

> wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：

```

// The standard idiom for using the wait method
synchronized (obj) {
while (condition does not hold)
obj.wait(); // (Releases lock, and reacquires on wakeup)
... // Perform action appropriate to condition
}
```

