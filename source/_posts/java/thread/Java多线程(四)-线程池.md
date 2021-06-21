---
title: Java多线程-线程池
mermaid: true
date: 2021-06-16 16:42:44
cover: /img/cover/Java.jpg
tags:
- 多线程
- 线程池
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

#### 线程池的工作流程

一个新的任务到线程池时，线程池的处理流程如下：

- 当一个任务通过submit或者execute方法提交到线程池的时候，如果当前池中线程数（包括闲置线程）小于`coolPoolSize`，则创建一个线程执行该任务。
- 如果当前线程池中线程数已经达到`coolPoolSize`，则将任务放入等待队列。
- 如果任务不能入队，说明等待队列已满，若当前池中线程数小于`maximumPoolSize`，则创建一个临时线程（非核心线程）执行该任务。
- 如果当前池中线程数已经等于`maximumPoolSize`，此时无法执行该任务，根据拒绝执行策略处理。

> **注意**：当池中线程数大于`coolPoolSize`，超过`keepAliveTime`时间的闲置线程会被回收掉。回收的是非核心线程，核心线程一般是不会回收的。如果设置`allowCoreThreadTimeOut(true)`，则核心线程在闲置`keepAliveTime`时间后也会被回收。

##### **工作线程（Worker)**

> 线程池在创建线程时，会将线程封装成工作线程Woker。Woker在执行完任务后，不是立即销毁而是循环获取阻塞队列里的任务来执行。

#### 线程池的创建（7个参数）

```java
new ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)
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
- **threadFactory**：构建线程的工厂类

如果使用的阻塞队列为无界队列，则永远不会调用拒绝策略，因为再多的任务都可以放在队列中。

#### 向线程池提交任务

> 使用ThreadPoolEXecutor.executor()方法来提交任务

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

#### 线程池的五种运行状态

* **RUNNING**：该状态的线程池既能接受新提交的任务，又能处理阻塞队列中任务。

* **SHUTDOWN**：该状态的线程池不能接收新提交的任务，但是能处理阻塞队列中的任务。（政府服务大厅不在允许群众拿号了，处理完手头的和排队的政务就下班。）
  * 处于 RUNNING 状态时，调用 shutdown()方法会使线程池进入到该状态。
  * 注意：finalize() 方法在执行过程中也会隐式调用shutdown()方法。

* **STOP**：该状态的线程池不接受新提交的任务，也不处理在阻塞队列中的任务，还会中断正在执行的任务。（政府服务大厅不再进行服务了，拿号、排队、以及手头工作都停止了。）
  * 在线程池处于 RUNNING 或 SHUTDOWN 状态时，调用 shutdownNow() 方法会使线程池进入到该状态；

* **TIDYING**：如果所有的任务都已终止，workerCount (有效线程数)=0 。
  * 线程池进入该状态后会调用 terminated() 钩子方法进入TERMINATED 状态。

* **TERMINATED**：在terminated()钩子方法执行完后进入该状态，默认terminated()钩子方法中什么也没有做

###### 线程池的关闭（shutdown或者shutdownNow方法）

* 可以通过调用线程池的shutdown或者shutdownNow方法来关闭线程池：遍历线程池中工作线程，逐个调用interrupt方法来中断线程。

###### shutdown方法与shutdownNow的特点：

- shutdown方法将线程池的状态设置为SHUTDOWN状态，只会中断空闲的工作线程。
- shutdownNow方法将线程池的状态设置为STOP状态，会中断所有工作线程，不管工作线程是否空闲。
- 调用两者中任何一种方法，都会使isShutdown方法的返回值为true；线程池中所有的任务都关闭后，isTerminated方法的返回值为true。
- 通常使用shutdown方法关闭线程池，如果不要求任务一定要执行完，则可以调用shutdownNow方法。

