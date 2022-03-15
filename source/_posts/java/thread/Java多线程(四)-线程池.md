---
title: Java多线程(四)-线程池
date: 2021-06-16 16:42:44
cover: /img/cover/Java.jpg
tags:
- 多线程
- 线程池
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

### 参考文献

* [Java 线程池必备知识点：工作流程、常见参数、调优、监控](https://mp.weixin.qq.com/s/R1NNht2JwaHze5U5b5dq-A)

### 线程池

#### 合理使用线程池的好处

- 降低资源消耗。

  - 通过重复利用已经创建的线程降低线程创建的和销毁造成的消耗。例如，工作线程Woker会无线循环获取阻塞队列中的任务来执行。

- 提高响应速度。

  - 当任务到达时，任务可以不需要等到线程创建就能立即执行。

- 提高线程的可管理性。

  - 线程是稀缺资源，Java的线程池可以对线程资源进行统一分配、调优和监控。

#### 实现线程池

* 自定义拒绝策略接口

  ```java
  @FunctionalInterface
  public interface RejectPolicy<T> {
      void reject(BlockingQueue<T> queue, T task);
  }
  ```

* 自定义任务队列

  ```java
  package com.holelin.sundry.test.thread;
  
  import lombok.extern.slf4j.Slf4j;
  
  import java.util.ArrayDeque;
  import java.util.Deque;
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.locks.Condition;
  import java.util.concurrent.locks.ReentrantLock;
  
  @Slf4j
  public class BlockingQueue<T> {
      /**
       * 任务队列
       */
      private Deque<T> queue = new ArrayDeque<>();
      /**
       * 锁
       */
      private ReentrantLock lock = new ReentrantLock();
      /**
       * 生产者条件变量
       */
      private Condition fullWaitSet = lock.newCondition();
      /**
       * 消费者条件变量
       */
      private Condition emptyWaitSet = lock.newCondition();
  
      /**
       * 容量
       */
      private int capacity;
  
      /**
       * 带超时阻塞获取
       *
       * @param timeout
       * @param unit
       * @return
       */
      public T poll(long timeout, TimeUnit unit) {
          lock.lock();
          try {
              long nanos = unit.toNanos(timeout);
              while (queue.isEmpty()) {
                  try {
                      if (nanos <= 0) {
                          return null;
                      }
                      emptyWaitSet.awaitNanos(nanos);
                  } catch (InterruptedException e) {
  
                  }
              }
              T t = queue.removeFirst();
              fullWaitSet.signal();
              return t;
          } finally {
              lock.unlock();
          }
      }
  
      /**
       * 阻塞获取
       *
       * @return
       */
      public T take() {
          lock.lock();
          try {
              while (queue.isEmpty()) {
                  try {
                      emptyWaitSet.await();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
              T t = queue.removeFirst();
              fullWaitSet.signal();
              return t;
          } finally {
              lock.unlock();
          }
      }
  
      public void put(T task) {
          lock.lock();
          try {
              while (queue.size() == capacity) {
                  log.info("等待加入任务队列:{}", task);
                  try {
                      fullWaitSet.await();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
              log.info("加入任务队列:{}", task);
              queue.addLast(task);
              emptyWaitSet.signal();
          } finally {
              lock.unlock();
          }
      }
  
      public boolean offer(T task, long timeout, TimeUnit unit) {
          lock.lock();
          try {
              long nanos = unit.toNanos(timeout);
              while (queue.size() == capacity) {
                  if (nanos <= 0) {
                      return false;
                  }
                  log.info("等待加入任务队列:{}", task);
                  try {
                      nanos = fullWaitSet.awaitNanos(nanos);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
              log.info("加入任务队列:{}", task);
              queue.addLast(task);
              emptyWaitSet.signal();
              return true;
          } finally {
              lock.unlock();
          }
      }
  
      public int size() {
          lock.lock();
          try {
              return queue.size();
          } finally {
              lock.unlock();
          }
      }
  
      public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
          lock.lock();
          try {
              if (queue.size()==capacity){
                  rejectPolicy.reject(this,task);
              }else {
                  log.info("加入任务队列:{}", task);
                  queue.addLast(task);
                  emptyWaitSet.signal();
              }
          } finally {
              lock.unlock();
          }
      }
  }
  ```

* 自定义线程池

  ```java
  package com.holelin.sundry.test.thread;
  
  import lombok.extern.slf4j.Slf4j;
  
  import java.util.HashSet;
  import java.util.Objects;
  import java.util.concurrent.TimeUnit;
  
  @Slf4j
  public class ThreadPool {
      /**
       * 任务队列
       */
      private BlockingQueue<Runnable> taskQueue;
      /**
       * 线程集合
       */
      private HashSet<Worker> workers = new HashSet<>();
      /**
       * 核心线程数
       */
      private int coreSize;
      /**
       * 获取任务时的超时时间
       */
      private long timeout;
  
      private TimeUnit timeUnit;
  
      private RejectPolicy<Runnable> rejectPolicy;
  
      public void execute(Runnable task) {
          synchronized (workers) {
              if (workers.size() < coreSize) {
                  Worker worker = new Worker(task);
                  log.info("新增worker{},{}", worker, task);
                  workers.add(worker);
                  worker.start();
              } else {
  //                taskQueue.put(task);
                  // 1. 死等
                  // 2. 带超时时间等待
                  // 3. 让调用者放弃任务执行
                  // 4. 让调用者抛出异常
                  // 5. 让调用者自己执行任务
                  taskQueue.tryPut(rejectPolicy, task);
              }
          }
      }
  
      public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
          this.workers = workers;
          this.coreSize = coreSize;
          this.timeout = timeout;
          this.timeUnit = timeUnit;
          this.taskQueue = new BlockingQueue<>(queueCapacity);
          this.rejectPolicy = rejectPolicy;
      }
  
      class Worker extends Thread {
          private Runnable task;
  
          public Worker(Runnable task) {
              this.task = task;
          }
  
          @Override
          public void run() {
              /**
               * 执行任务
               * 1. 当task不为空,执行任务
               * 2. 当task执行完毕,再接着从任务队列中获取任务并执行
               */
              while (Objects.nonNull(task) || Objects.nonNull(task = taskQueue.poll(timeout, timeUnit))) {
                  try {
                      log.info("正在执行...{}", task);
                      task.run();
                  } catch (Exception e) {
                      e.printStackTrace();
                  } finally {
                      task = null;
                  }
              }
              synchronized (workers) {
                  log.info("worker被移除{}", this);
                  workers.remove(this);
              }
          }
      }
  }
  
  ```

* 测试

  ```java
  @Slf4j
  public class ThreadPoolTest {
      public static void main(String[] args) {
          ThreadPool threadPool = new ThreadPool(1, 100, TimeUnit.MILLISECONDS, 1, ((queue, task) -> {
              // 1. 死等
  //            queue.put(task);
              // 2. 带超时时间等待
  //            queue.offer(task, 1500, TimeUnit.MILLISECONDS);
              // 3. 让调用者放弃任务执行
  //            log.info("放弃执行:{}", task);
              // 4. 让调用者抛出异常
  //            throw new RuntimeException("任务执行失败" + task);
              // 5. 让调用者自己执行任务
              task.run();
          }));
          for (int i = 0; i < 4; i++) {
              int j = i;
              threadPool.execute(() -> {
                  try {
                      Thread.sleep(1000L);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  log.info("{}", j);
              });
          }
      }
  }
  ```
  
  ```java
  public class ThreadPoolExecutorTest {
      private static AtomicInteger threadId = new AtomicInteger(0);
      private static final int CORE_POOL_SIZE = 5;
      private static final int MAXIMUM_POOL_SIZE = 10;
      private static final long KEEP_ALIVE_TIME = 10;
  
      public static void main(String[] args) {
          // 手动创建线程池
          // 创建有界阻塞队列
          ArrayBlockingQueue<Runnable> runnable = new ArrayBlockingQueue<Runnable>(10);
          // 创建线程工厂
          ThreadFactory threadFactory = new ThreadFactory() {
              @Override
              public Thread newThread(Runnable r) {
                  Thread thread = new Thread(r, "working_thread_" + threadId.getAndIncrement());
                  return thread;
              }
          };
  
          // 手动创建线程池
          // 拒绝策略采用默认策略
          ThreadPoolExecutor executor = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_TIME, TimeUnit.SECONDS, runnable, threadFactory);
  
          for (int i = 0; i < 20; i++) {
              executor.execute(new Runnable() {
                  @Override
                  public void run() {
                      System.out.println(Thread.currentThread());
                      try {
                          Thread.sleep(1000);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
              });
          }
      }
  }
  ```

#### 线程池的工作流程

一个新的任务到线程池时，线程池的处理流程如下：

- 当一个任务通过submit或者execute方法提交到线程池的时候，如果当前池中线程数（包括闲置线程）小于`coolPoolSize`，则创建一个线程执行该任务。
- 如果当前线程池中线程数已经达到`coolPoolSize`，则将任务放入等待队列。
- 如果任务不能入队，说明等待队列已满，若当前池中线程数小于`maximumPoolSize`，则创建一个临时线程（非核心线程）执行该任务。
- 如果当前池中线程数已经等于`maximumPoolSize`，此时无法执行该任务，根据拒绝执行策略处理。

> **注意**：当池中线程数大于`coolPoolSize`，超过`keepAliveTime`时间的闲置线程会被回收掉。回收的是非核心线程，核心线程一般是不会回收的。如果设置`allowCoreThreadTimeOut(true)`，则核心线程在闲置`keepAliveTime`时间后也会被回收。

![img](https://www.holelin.cn/img/java/thread/Java%E7%BA%BF%E7%A8%8B%E6%B1%A0.png)

##### **工作线程（Worker)**

> 线程池在创建线程时，会将线程封装成工作线程Woker。Woker在执行完任务后，不是立即销毁而是循环获取阻塞队列里的任务来执行。

#### 线程池的创建（7个参数）

```java
ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,
                   long keepAliveTime,
                   TimeUnit unit,
                   BlockingQueue<Runnable> workQueue,
                   ThreadFactory threadFactory,
                   RejectedExecutionHandler handler)
```

###### **corePoolSize（线程池的基本大小）**

- 提交一个任务到线程池时，线程池会创建一个新的线程来执行任务。注意：即使有空闲的基本线程能执行该任务，也会创建新的线程。
- 如果线程池中的线程数已经大于或等于corePoolSize，则不会创建新的线程。
- 如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。

###### **maximumPoolSize（线程池的最大数量）：**

>  线程池允许创建的最大线程数。

- 阻塞队列已满，线程数小于maximumPoolSize便可以创建新的线程执行任务。
- 如果使用无界的阻塞队列，该参数没有什么效果

###### **workQueue（工作队列）**

> 用于保存等待执行的任务的阻塞队列。

- **ArrayBlockingQueue**：基于数组结构的有界阻塞队列，按FIFO（先进先出）原则对任务进行排序。使用该队列，线程池中能创建的最大线程数为maximumPoolSize。
- **LinkedBlockingQueue**：基于链表结构的无界阻塞队列，按FIFO（先进先出）原则对任务进行排序，吞吐量高于ArrayBlockingQueue。使用该队列，线程池中能创建的最大线程数为corePoolSize。静态工厂方法 Executor.newFixedThreadPool()使用了这个队列。
- **SynchronousQueue**：一个不存储元素的阻塞队列。添加任务的操作必须等到另一个线程的移除操作，否则添加操作一直处于阻塞状态。静态工厂方法 Executor.newCachedThreadPool()使用了这个队列。
  - SynchronousQueue是不存储任务的，新的任务要么立即被已有线程执行，要么创建新的线程执行。
- **PriorityBlokingQueue**：一个支持优先级的无界阻塞队列。使用该队列，线程池中能创建的最大线程数为corePoolSize。
- 如果使用的阻塞队列为无界队列，则永远不会调用拒绝策略，因为再多的任务都可以放在队列中。


###### **keepAliveTime（线程活动保持时间）：**

>  线程池的工作线程空闲后，保持存活的时间。如果任务多而且任务的执行时间比较短，可以调大keepAliveTime，提高线程的利用率。

###### **unit（线程活动保持时间的单位）**

> 可选单位有DAYS、HOURS、MINUTES、毫秒、微秒、纳秒。

###### **handler（饱和策略，或者又称拒绝策略）**

> 当队列和线程池都满了，即线程池饱和了，必须采取一种策略处理提交的新任务。

- **AbortPolicy**：无法处理新任务时，直接抛出异常，这是默认策略。

- **CallerRunsPolicy**：用调用者所在的线程来执行任务。

- **DiscardOldestPolicy**：丢弃阻塞队列中最靠前的一个任务，并执行当前任务。

- **DiscardPolicy**：直接丢弃任务。

###### **threadFactory**(构建线程的工厂类)

> 可以为线程创建时起个名字

#### 向线程池提交任务

> 使用ThreadPoolExecutor.executor()方法来提交任务

```java
public void execute(Runnable command) {
    // command为null，抛出NullPointerException
    if (command == null)
        throw new NullPointerException();      
    int c = ctl.get();
    // 线程池中的线程数小于corePoolSize，创建新的线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))// 创建工作线程
            return;
        c = ctl.get();
    }
    // 将任务添加到阻塞队列，如果
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }// 阻塞队列已满，尝试创建新的线程，如果超过maximumPoolSize，执行handler.rejectExecution()
    else if (!addWorker(command, false))
        reject(command);
}
```

```java
public class ThreadPoolExecutorTest {
    private static AtomicInteger threadId = new AtomicInteger(0);
    private static final int CORE_POOL_SIZE = 5;
    private static final int MAXIMUM_POOL_SIZE = 10;
    private static final long KEEP_ALIVE_TIME = 10;

    public static void main(String[] args) {
        // 手动创建线程池
        // 创建有界阻塞队列
        ArrayBlockingQueue<Runnable> runnable = new ArrayBlockingQueue<Runnable>(10);
        // 创建线程工厂
        ThreadFactory threadFactory = new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r, "working_thread_" + threadId.getAndIncrement());
                return thread;
            }
        };

        // 手动创建线程池
        // 拒绝策略采用默认策略
        ThreadPoolExecutor executor = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_TIME, TimeUnit.SECONDS, runnable, threadFactory);

        for (int i = 0; i < 20; i++) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread());
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```

#### 线程池的五种运行状态

```java
// 线程池状态
// runState is stored in the high-order bits
// RUNNING 高3位为111
private static final int RUNNING    = -1 << COUNT_BITS;

// SHUTDOWN 高3位为000
private static final int SHUTDOWN   =  0 << COUNT_BITS;

// 高3位 001
private static final int STOP       =  1 << COUNT_BITS;

// 高3位 010
private static final int TIDYING    =  2 << COUNT_BITS;

// 高3位 011
private static final int TERMINATED =  3 << COUNT_BITS;
```

* **RUNNING**：该状态的线程池既能接受新提交的任务，又能处理阻塞队列中任务。
* **SHUTDOWN**：该状态的线程池不能接收新提交的任务，但是能处理阻塞队列中的任务。（政府服务大厅不在允许群众拿号了，处理完手头的和排队的政务就下班。）
  * 处于 RUNNING 状态时，调用 shutdown()方法会使线程池进入到该状态。
  * 注意：finalize() 方法在执行过程中也会隐式调用shutdown()方法。
* **STOP**：该状态的线程池不接受新提交的任务，也不处理在阻塞队列中的任务，还会中断正在执行的任务。（政府服务大厅不再进行服务了，拿号、排队、以及手头工作都停止了。）
  * 在线程池处于 RUNNING 或 SHUTDOWN 状态时，调用 shutdownNow() 方法会使线程池进入到该状态；
* **TIDYING**：如果所有的任务都已终止，workerCount (有效线程数)=0 。
  * 线程池进入该状态后会调用 terminated() 钩子方法进入TERMINATED 状态。
* **TERMINATED**：在terminated()钩子方法执行完后进入该状态，默认terminated()钩子方法中什么也没有做

| 状态名称   | 高3位的值 | 描述                                          |
| ---------- | --------- | --------------------------------------------- |
| RUNNING    | 111       | 接收新任务，同时处理任务队列中的任务          |
| SHUTDOWN   | 000       | 不接受新任务，但是处理任务队列中的任务        |
| STOP       | 001       | 中断正在执行的任务，同时抛弃阻塞队列中的任务  |
| TIDYING    | 010       | 任务执行完毕，活动线程为0时，即将进入终结阶段 |
| TERMINATED | 011       | 终结状态                                      |

* 线程池状态和线程池中线程的数量**由一个原子整型ctl来共同表示**

  * 使用一个数来表示两个值的主要原因是：**可以通过一次CAS同时更改两个属性的值**

  ```java
  // 原子整数，前3位保存了线程池的状态，剩余位保存的是线程数量
  private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
  
  // 并不是所有平台的int都是32位。
  // 去掉前三位保存线程状态的位数，剩下的用于保存线程数量
  // 高3位为0，剩余位数全为1
  private static final int COUNT_BITS = Integer.SIZE - 3;
  
  // 2^COUNT_BITS次方，表示可以保存的最大线程数
  // CAPACITY 的高3位为 0
  private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
  ```

* 获取线程池状态、线程数量以及合并两个值的操作

  ```java
  // Packing and unpacking ctl
  // 获取运行状态
  // 该操作会让除高3位以外的数全部变为0
  private static int runStateOf(int c)     { return c & ~CAPACITY; }
  
  // 获取运行线程数
  // 该操作会让高3位为0
  private static int workerCountOf(int c)  { return c & CAPACITY; }
  
  // 计算ctl新值
  private static int ctlOf(int rs, int wc) { return rs | wc; }
  ```

###### 线程池的关闭（shutdown或者shutdownNow方法）

* 可以通过调用线程池的shutdown或者shutdownNow方法来关闭线程池：遍历线程池中工作线程，逐个调用interrupt方法来中断线程。

###### shutdown方法与shutdownNow的特点：

- shutdown方法将线程池的状态设置为SHUTDOWN状态，只会中断空闲的工作线程。
- shutdownNow方法将线程池的状态设置为STOP状态，会中断所有工作线程，不管工作线程是否空闲。
- 调用两者中任何一种方法，都会使isShutdown方法的返回值为true；线程池中所有的任务都关闭后，isTerminated方法的返回值为true。
- 通常使用shutdown方法关闭线程池，如果不要求任务一定要执行完，则可以调用shutdownNow方法。

#### Executors

##### **newFixedThreadPool**

```java
 /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

* 特点: 
  * 核心线程数==最大线程数(没有非核心线程),因此也无需超时时间;
  * 阻塞队列是无界的,可以放任意数量的任务;
* 使用场景: **适用与任务量已知,相对耗时的任务**;

##### newCachedThreadPool

```java
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

* 特点:
  * 核心线程数为0,最大线程数为`Integer.MAX_VALUE`,非核心线程的生存时间为60s,意味着全都是非核心线程(60s后可以回收)
  * 非核心线程可以无限创建;
  * 队列采用`SynchronousQueue`实现,它没有容量,没有线程来取是放不进去的(一手交钱,一手交货);

##### newSingleThreadExecutor

```java
/**
 * Creates an Executor that uses a single worker thread operating
 * off an unbounded queue. (Note however that if this single
 * thread terminates due to a failure during execution prior to
 * shutdown, a new one will take its place if needed to execute
 * subsequent tasks.)  Tasks are guaranteed to execute
 * sequentially, and no more than one task will be active at any
 * given time. Unlike the otherwise equivalent
 * {@code newFixedThreadPool(1)} the returned executor is
 * guaranteed not to be reconfigurable to use additional threads.
 *
 * @return the newly created single-threaded Executor
 */
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

* 使用场景
  * 希望多个任务排队执行,线程数固定为1,任务数多于1时,会放入无界队列中排队,任务执行完毕,这个唯一的线程也不会被是否;
* 区别:
  * 自己创建一个单线程串行执行任务,如果任务执行失败而终止没有任何补救措施,而线程池还会新建一个线程,保证线程池的正常工作;
  * `ExecutorService newSingleThreadExecutor()`线程个数为1,不能修改;
    * `FinalizableDelegatedExecutorService` 应用的是装饰器模式模式,只对外暴露了`ExecutorService`接口,因此不能调用`ThreadPoolExecutor`中特有的方法;
  * `ExecutorService newFixedThreadPool(1)`初始时为1,以后还可以修改;
    * 对外暴露的是`ThreadPoolExecutor`对象,可以强转后调用`setCorePoolSize`等方法进行修改

##### newScheduledThreadPool

```java
 	/**
     * Creates a thread pool that can schedule commands to run after a
     * given delay, or to execute periodically.
     * @param corePoolSize the number of threads to keep in the pool,
     * even if they are idle
     * @return a newly created scheduled thread pool
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

#### 向线程池提交任务

> 使用ThreadPoolEXecutor.execute()方法来提交任务

```java
	// 执行任务
	public void execute(Runnable command){};

	// 提交任务task,用返回值Future获取任务执行结果
	<T> Future<T> submit(Callable<T> task);
	
	// 提交tasks中所有任务
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

	// 提交tasks中所有任务,带超时时间
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

	// 提交tasks中所有任务,哪个任务先成功执行完毕,返回此任务执行结果,其他任务取消
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

	// 提交tasks中所有任务,哪个任务先成功执行完毕,返回此任务执行结果,其他任务取消,带超时时间
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

```java
public void execute(Runnable command) {
    // command为null，抛出NullPointerException
    if (command == null)
        throw new NullPointerException(); 
    // 获取ctl
    int c = ctl.get();
        // 判断当前启用的线程数是否小于核心线程数
    if (workerCountOf(c) < corePoolSize) {
        // 为该任务分配线程
        if (addWorker(command, true))
            // 分配成功就返回
            return;
        // 分配失败再次获取ctl
        c = ctl.get();
    }
    // 分配和信息线程失败以后
    // 如果池状态为RUNNING并且插入到任务队列成功
    if (isRunning(c) && workQueue.offer(command)) {
        // 双重检测，可能在添加后线程池状态变为了非RUNNING
        int recheck = ctl.get();
        // 如果池状态为非RUNNING，则不会执行新来的任务
        // 将该任务从阻塞队列中移除
        if (! isRunning(recheck) && remove(command))
            // 调用拒绝策略，拒绝该任务的执行
            reject(command);
        // 如果没有正在运行的线程
        else if (workerCountOf(recheck) == 0)
             // 就创建新线程来执行该任务
            addWorker(null, false);
    }// 阻塞队列已满，尝试创建新的线程，如果超过maximumPoolSize，执行handler.rejectExecution()
    else if (!addWorker(command, false))
        reject(command);
}
```

* addWorker()

  ```java
  private boolean addWorker(Runnable firstTask, boolean core) {
      retry:
      for (;;) {
          int c = ctl.get();
          int rs = runStateOf(c);
  
          // Check if queue empty only if necessary.
          // 如果池状态为非RUNNING状态、线程池为SHUTDOWN且该任务为空 或者阻塞队列中已经有任务
          if (rs >= SHUTDOWN &&
              ! (rs == SHUTDOWN &&
                 firstTask == null &&
                 ! workQueue.isEmpty()))
              // 创建新线程失败
              return false;
  
          for (;;) {
              // 获得当前工作线程数
              int wc = workerCountOf(c);
  
              // 参数中 core 为true
              // CAPACITY 为 1 << COUNT_BITS-1，一般不会超过
              // 如果工作线程数大于了核心线程数，则创建失败
              if (wc >= CAPACITY ||
                  wc >= (core ? corePoolSize : maximumPoolSize))
                  return false;
              // 通过CAS操作改变c的值
              if (compareAndIncrementWorkerCount(c))
                  // 更改成功就跳出多重循环，且不再运行循环
                  break retry;
              // 更改失败，重新获取ctl的值
              c = ctl.get();  // Re-read ctl
              if (runStateOf(c) != rs)
                  // 跳出多重循环，且重新进入循环
                  continue retry;
              // else CAS failed due to workerCount change; retry inner loop
          }
      }
  
      // 用于标记work中的任务是否成功执行
      boolean workerStarted = false;
      // 用于标记worker是否成功加入了线程池中
      boolean workerAdded = false;
      Worker w = null;
      try {
          // 创建新线程来执行任务
          w = new Worker(firstTask);
          final Thread t = w.thread;
          if (t != null) {
              final ReentrantLock mainLock = this.mainLock;
              // 加锁
              mainLock.lock();
              try {
                  // Recheck while holding lock.
                  // Back out on ThreadFactory failure or if
                  // shut down before lock acquired.
                  // 加锁的同时再次检测
                  // 避免在释放锁之前调用了shut down
                  int rs = runStateOf(ctl.get());
  
                  if (rs < SHUTDOWN ||
                      (rs == SHUTDOWN && firstTask == null)) {
                      if (t.isAlive()) // precheck that t is startable
                          throw new IllegalThreadStateException();
                      // 将线程添加到线程池中
                      workers.add(w);
                      int s = workers.size();
                      if (s > largestPoolSize)
                          largestPoolSize = s;
                      // 添加成功标志位变为true
                      workerAdded = true;
                  }
              } finally {
                  mainLock.unlock();
              }
              // 如果worker成功加入了线程池，就执行其中的任务
              if (workerAdded) {
                  t.start();
                  // 启动成功
                  workerStarted = true;
              }
          }
      } finally {
          // 如果执行失败
          if (! workerStarted)
              // 调用添加失败的函数
              addWorkerFailed(w);
      }
      return workerStarted;
  }
  ```

#### 关闭线程池

* #### shutdown()

  ```java
  /**
  * 将线程池的状态改为 SHUTDOWN
  * 不再接受新任务，但是会将阻塞队列中的任务执行完
  */
  public void shutdown() {
      final ReentrantLock mainLock = this.mainLock;
      mainLock.lock();
      try {
          checkShutdownAccess();
          
          // 修改线程池状态为 SHUTDOWN
          advanceRunState(SHUTDOWN);
          
    		// 中断空闲线程（没有执行任务的线程）
          // Idle：空闲的
          interruptIdleWorkers();
          onShutdown(); // hook for ScheduledThreadPoolExecutor
      } finally {
          mainLock.unlock();
      }
      // 尝试终结，不一定成功
      // 
      tryTerminate();
  }
  ```

  ```java
  final void tryTerminate() {
      for (;;) {
          int c = ctl.get();
          // 终结失败的条件
          // 线程池状态为RUNNING
          // 线程池状态为 RUNNING SHUTDOWN STOP （状态值大于TIDYING）
          // 线程池状态为SHUTDOWN，但阻塞队列中还有任务等待执行
          if (isRunning(c) ||
              runStateAtLeast(c, TIDYING) ||
              (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
              return;
          
          // 如果活跃线程数不为0
          if (workerCountOf(c) != 0) { // Eligible to terminate
              // 中断空闲线程
              interruptIdleWorkers(ONLY_ONE);
              return;
          }
  
          final ReentrantLock mainLock = this.mainLock;
          mainLock.lock();
          try {
              // 处于可以终结的状态
              // 通过CAS将线程池状态改为TIDYING
              if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                  try {
                      terminated();
                  } finally {
                      // 通过CAS将线程池状态改为TERMINATED
                      ctl.set(ctlOf(TERMINATED, 0));
                      termination.signalAll();
                  }
                  return;
              }
          } finally {
              mainLock.unlock();
          }
          // else retry on failed CAS
      }
  }
  ```

* #### shutdownNow

  ```java
  /**
  * 将线程池的状态改为 STOP
  * 不再接受新任务，也不会在执行阻塞队列中的任务
  * 会将阻塞队列中未执行的任务返回给调用者
  */
  public List<Runnable> shutdownNow() {
      List<Runnable> tasks;
      final ReentrantLock mainLock = this.mainLock;
      mainLock.lock();
      try {
          checkShutdownAccess();
          
          // 修改状态为STOP，不执行任何任务
          advanceRunState(STOP);
          
          // 中断所有线程
          interruptWorkers();
          
          // 将未执行的任务从队列中移除，然后返回给调用者
          tasks = drainQueue();
      } finally {
          mainLock.unlock();
      }
      // 尝试终结，一定会成功，因为阻塞队列为空了
      tryTerminate();
      return tasks;
  }
  ```

#### 任务调度线程池

* 在"任务调用线程池"功能加入之前,可以使用`java.util.Timer`来实现定时功能,Timer的优点在于简单易用,但由于所有任务都是有一个同一个线程来调度,因此所有任务都是串行执行的,同一时间只有一个任务执行,前一个任务的延迟或异常将会影响到之后的任务;

* **Timer实现方式**

  ```java
  @Slf4j
  public class TimerTest {
      public static void main(String[] args) {
          Timer timer = new Timer();
          TimerTask timerTask1 = new TimerTask() {
              @Override
              public void run() {
                  log.info("task 1");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          };
          TimerTask timerTask2 = new TimerTask() {
              @Override
              public void run() {
                  log.info("task 2");
              }
          };
          // 使用timer添加两个任务,希望它们都是1s执行
          // 但由于timer内只有一个线程来顺序执行队列中任务,因此[任务1]的延迟,影响了[任务2]的执行
          timer.schedule(timerTask1,1000);
          timer.schedule(timerTask2,1000);
      }
  }
  
  21:01:24.204 [Timer-0] INFO com.holelin.sundry.test.thread.TimerTest - task 1
  21:01:25.216 [Timer-0] INFO com.holelin.sundry.test.thread.TimerTest - task 2
  ```

*  **ScheduledExecutorService**

  ```java
          ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
          // 添加两个任务,希望它们都在1s后执行
          executor.schedule(() -> {
              log.info("任务1,执行时间:{}", new Date());
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }, 1000, TimeUnit.MILLISECONDS);
          executor.schedule(() -> {
              log.info("任务2,执行时间:{}", new Date());
          }, 1000, TimeUnit.MILLISECONDS);
  
  21:00:09.547 [pool-1-thread-1] INFO com.holelin.sundry.test.thread.TimerTest - 任务1,执行时间:Tue Jun 22 21:00:09 CST 2021
  21:00:09.547 [pool-1-thread-2] INFO com.holelin.sundry.test.thread.TimerTest - 任务2,执行时间:Tue Jun 22 21:00:09 CST 2021
      
  // public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);
  // 上个任务结束-->延迟-->下一个任务开始
  // public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit);
  ```

* 整个线程池表现为:线程数固定,任务数多于线程数时,会放入无界队列排队,任务执行完毕,这些线程也不会释放,用来执行延迟或反复执行的任务

#### 正确处理执行任务异常

* 主动捕获异常

* **Future**

  ```java
  @Slf4j
  public class ThreadExceptionTest {
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          ExecutorService pool = Executors.newFixedThreadPool(1);
          Future<Boolean> task = pool.submit(() -> {
              log.info("task1");
              int i = 1 / 0;
              return true;
          });
          log.info("result:{}", task.get());
      }
  }
  ```

### Fork/Join



  
