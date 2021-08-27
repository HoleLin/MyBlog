---
title: Java多线程(十一)-CyclicBarrier
cover: /img/cover/Java.jpg
date: 2021-06-30 22:33:12
tags:
- 多线程
- CyclicBarrier
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

* [CyclicBarrier 原理（秒懂)](https://www.cnblogs.com/crazymakercircle/p/13906379.html)

#### `CyclicBarrier`概念

* `CyclicBarrier`是Java提供的同步辅助类。

* 它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)，才得以继续执行。阻塞子线程，当阻塞数量到达定义的参与线程数后，才可继续向下执行;
* 通俗讲：让一组线程到达一个屏障时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活
* 底层采用`ReentrantLock` + `Condition`实现

#### `CyclicBarrier`使用场景

* 可以用于多线程计算数据，最后合并计算结果的场景。

#### `CyclicBarrier`示例

```java
package com.holelin.sundry.demo;

import java.util.concurrent.CyclicBarrier;

class CyclicBarrierTest {
    static class TaskThread extends Thread {

        CyclicBarrier barrier;

        public TaskThread(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(1000);
                System.out.println(getName() + " 到达栅栏 A");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 A");

                Thread.sleep(2000);
                System.out.println(getName() + " 到达栅栏 B");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 B");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        int threadNum =3;
        CyclicBarrier barrier = new CyclicBarrier(threadNum, () -> System.out.println(Thread.currentThread().getName() + " 完成最后任务"));

        for (int i = 0; i < threadNum; i++) {
            new TaskThread(barrier).start();
        }
    }
}
Thread-1 到达栅栏 A
Thread-2 到达栅栏 A
Thread-0 到达栅栏 A
Thread-0 完成最后任务
Thread-0 冲破栅栏 A
Thread-2 冲破栅栏 A
Thread-1 冲破栅栏 A
Thread-0 到达栅栏 B
Thread-2 到达栅栏 B
Thread-1 到达栅栏 B
Thread-1 完成最后任务
Thread-1 冲破栅栏 B
Thread-0 冲破栅栏 B
Thread-2 冲破栅栏 B
```

#### `CyclicBarrier` 原理

* 在`CyclicBarrier`类的内部有一个计数器，每个线程在到达屏障点的时候都会调用await方法将自己阻塞，此时计数器会减1，当计数器减为0的时候所有因调用await方法而被阻塞的线程将被唤醒。这就是实现一组线程相互等待的原理;

  <img src="https://www.holelin.cn/img/java/thread/CyclicBarrier/CyclicBarrier.png" alt="img" style="zoom: 50%;" />
  
  ```java
      /**
       * Each use of the barrier is represented as a generation instance.
       * The generation changes whenever the barrier is tripped, or
       * is reset. There can be many generations associated with threads
       * using the barrier - due to the non-deterministic way the lock
       * may be allocated to waiting threads - but only one of these
       * can be active at a time (the one to which {@code count} applies)
       * and all the rest are either broken or tripped.
       * There need not be an active generation if there has been a break
       * but no subsequent reset.
       */
      // 静态内部类Generation
      private static class Generation {
          boolean broken = false;
      }
  
      /** The lock for guarding barrier entry */
      // 同步操作锁
      private final ReentrantLock lock = new ReentrantLock();
      /** Condition to wait on until tripped */
      // 线程拦截器
      private final Condition trip = lock.newCondition();
      /** The number of parties */
      // 每次拦截的线程数
      private final int parties;
      /* The command to run when tripped */
      // 换代前执行的任务
      private final Runnable barrierCommand;
      /** The current generation */
      // 表示栅栏的当前代
      private Generation generation = new Generation();
  
      /**
       * Number of parties still waiting. Counts down from parties to 0
       * on each generation.  It is reset to parties on each new
       * generation or when broken.
       */
      // 计数器
      private int count;
  ```

  ##### 构造器
  
  ```java
     /**
       * Creates a new {@code CyclicBarrier} that will trip when the
       * given number of parties (threads) are waiting upon it, and which
       * will execute the given barrier action when the barrier is tripped,
       * performed by the last thread entering the barrier.
       *
       * @param parties the number of threads that must invoke {@link #await}
       *        before the barrier is tripped
       * @param barrierAction the command to execute when the barrier is
       *        tripped, or {@code null} if there is no action
       * @throws IllegalArgumentException if {@code parties} is less than 1
       */
      public CyclicBarrier(int parties, Runnable barrierAction) {
          if (parties <= 0) throw new IllegalArgumentException();
          this.parties = parties;
          this.count = parties;
          this.barrierCommand = barrierAction;
      }
      /**
       * Creates a new {@code CyclicBarrier} that will trip when the
       * given number of parties (threads) are waiting upon it, and
       * does not perform a predefined action when the barrier is tripped.
       *
       * @param parties the number of threads that must invoke {@link #await}
       *        before the barrier is tripped
       * @throws IllegalArgumentException if {@code parties} is less than 1
       */
      public CyclicBarrier(int parties) {
          this(parties, null);
      }
  ```

  ##### 等待的方法

  > `CyclicBarrier`类最主要的功能就是使先到达屏障点的线程阻塞并等待后面的线程，其中它提供了两种等待的方法，分别是定时等待和非定时等待。
  
  ```java
  //非定时等待
  public int await() throws InterruptedException, BrokenBarrierException {
    try {
      return dowait(false, 0L);
    } catch (TimeoutException toe) {
      throw new Error(toe);
    }
  }
  
  //定时等待
  public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException {
    return dowait(true, unit.toNanos(timeout));
  }
  ```
  
  ```java
      /**
       * Main barrier code, covering the various policies.
       */
      private int dowait(boolean timed, long nanos)
          throws InterruptedException, BrokenBarrierException,
                 TimeoutException {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              final Generation g = generation;
  			// 检查当前栅栏是否被打翻
              if (g.broken)
                  throw new BrokenBarrierException();
      		// 检查当前线程是否被中断
              if (Thread.interrupted()) {
                  // 如果当前线程被中断会做以下三件事
                  // 1.打翻当前栅栏
                  // 2.唤醒拦截的所有线程
                  // 3.抛出中断异常
                  breakBarrier();
                  throw new InterruptedException();
              }
     			// 每次都将计数器的值减1
              int index = --count;
              // 计数器的值减为0则需唤醒所有线程并转换到下一代
              if (index == 0) {  // tripped
                  boolean ranAction = false;
                  try {
                      // 唤醒所有线程前先执行指定的任务
                      final Runnable command = barrierCommand;
                      if (command != null)
                          command.run();
                      ranAction = true;
                      // 唤醒所有线程并转到下一代
                      nextGeneration();
                      return 0;
                  } finally {
                      // 确保在任务未成功执行时能将所有线程唤醒
                      if (!ranAction)
                          breakBarrier();
                  }
              }
  
              // loop until tripped, broken, interrupted, or timed out
              // 如果计数器不为0则执行此循环
              for (;;) {
                  try {
                      // 根据传入的参数来决定是定时等待还是非定时等待
                      if (!timed)
                          trip.await();
                      else if (nanos > 0L)
                          nanos = trip.awaitNanos(nanos);
                  } catch (InterruptedException ie) {
                      // 若当前线程在等待期间被中断则打翻栅栏唤醒其他线程
                      if (g == generation && ! g.broken) {
                          breakBarrier();
                          throw ie;
                      } else {
                          // We're about to finish waiting even if we had not
                          // been interrupted, so this interrupt is deemed to
                          // "belong" to subsequent execution.
                          // 若在捕获中断异常前已经完成在栅栏上的等待, 则直接调用中断操作
                          Thread.currentThread().interrupt();
                      }
                  }
    				// 如果线程因为打翻栅栏操作而被唤醒则抛出异常
                  if (g.broken)
                      throw new BrokenBarrierException();
    				// 如果线程因为换代操作而被唤醒则返回计数器的值
                  if (g != generation)
                      return index;
    				// 如果线程因为时间到了而被唤醒则打翻栅栏并抛出异常
                  if (timed && nanos <= 0L) {
                      breakBarrier();
                      throw new TimeoutException();
                  }
              }
          } finally {
              lock.unlock();
          }
      }
  
  ```

  ##### 重置一个栅栏
  
  ```java
  public void reset() {
      final ReentrantLock lock = this.lock;
      lock.lock();
      try {
          breakBarrier();   // break the current generation
          nextGeneration(); // start a new generation
      } finally {
          lock.unlock();
      }
  }
  ```
  
  
