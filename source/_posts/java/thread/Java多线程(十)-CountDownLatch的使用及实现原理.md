---
title: Java多线程(十)-CountDownLatch
date: 2021-06-30 00:21:17
cover: /img/cover/Java.jpg
tags:
- 多线程
- CountDownLatch
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

* [CountDownLatch的使用及实现原理](https://juejin.cn/post/6971661675543920677?utm_source=gold_browser_extension)
* [CountDownLatch的两种常用场景](https://zhuanlan.zhihu.com/p/148231820)
* [CountDownLatch的理解和使用](https://www.cnblogs.com/Lee_xy_z/p/10470181.html)

#### CountDownLatch概念

> A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
>
> 一种同步辅助，允许一个或多个线程等待，直到在其他线程中执行的一组操作完成。

* CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）。
* CountDownLatch能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行。使用一个计数器进行实现。计数器初始值为线程的数量。当每一个线程完成自己任务后，计数器的值就会减一。当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务。
* 用给定的计数 初始化 `CountDownLatch`。由于调用了 `countDown() `方法，所以在当前计数到达零之前，`await` 方法会一直受阻塞。之后，会释放所有等待的线程，`await` 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 `CyclicBarrier`。
* 与`CyclicBarrier`区别
  * `CountDownLatch`的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待
  * `CountDownLatch`的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier
* 内部采用共享锁来实现

#### CountDownLatch使用场景

##### **让多个线程等待：模拟并发，让并发线程一起执行**

* 为了模拟高并发，让一组线程在指定时刻(秒杀时间)执行抢购，这些线程在准备就绪后，进行等待(CountDownLatch.await())，直到秒杀时刻的到来，然后一拥而上;

* 在这个场景中，CountDownLatch充当的是一个`发令枪`的角色；
  就像田径赛跑时，运动员会在起跑线做准备动作，等到发令枪一声响，运动员就会奋力奔跑。和上面的秒杀场景类似，代码实现如下

  ```java
  @Slf4j
  public class CountDownLatchTest {
  
      public static final int N = 5;
  
      public static void main(String[] args) {
          CountDownLatch latch = new CountDownLatch(1);
          for (int i = 0; i < N; i++) {
              new Thread(() -> {
                  try {
                      latch.await();
                      log.info("[{}] 开始执行", Thread.currentThread().getName());
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }).start();
          }
          latch.countDown();
      }
  }
  
  22:01:40.133 [Thread-2] INFO com.holelin.sundry.demo.CountDownLatchTest - [Thread-2] 开始执行
  22:01:40.133 [Thread-1] INFO com.holelin.sundry.demo.CountDownLatchTest - [Thread-1] 开始执行
  22:01:40.133 [Thread-3] INFO com.holelin.sundry.demo.CountDownLatchTest - [Thread-3] 开始执行
  22:01:40.133 [Thread-4] INFO com.holelin.sundry.demo.CountDownLatchTest - [Thread-4] 开始执行
  22:01:40.133 [Thread-0] INFO com.holelin.sundry.demo.CountDownLatchTest - [Thread-0] 开始执行
  ```

##### **让单个线程等待：多个线程(任务)完成后，进行汇总合并**

* 很多时候，我们的并发任务，存在前后依赖关系；比如数据详情页需要同时调用多个接口获取数据，并发请求获取到数据后、需要进行结果合并；或者多个数据操作完成后，需要数据check；这其实都是：在多个线程(任务)完成后，进行汇总合并的场景

  ```java
        private static void scenes2() {
          CountDownLatch latch = new CountDownLatch(5);
          for (int i = 0; i < N; i++) {
              int finalI = i;
              new Thread(() -> {
                  try {
                      Thread.sleep(1000 + ThreadLocalRandom.current().nextInt(1000));
                      log.info("Finish {},[{}]", finalI, Thread.currentThread().getName());
                      latch.countDown();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }).start();
          }
          try {
              latch.await();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          log.info("主线程:在所有任务运行完成后，进行结果汇总");
      }
  
  22:11:49.178 [Thread-2] INFO com.holelin.sundry.demo.CountDownLatchTest - Finish 2,[Thread-2]
  22:11:49.257 [Thread-1] INFO com.holelin.sundry.demo.CountDownLatchTest - Finish 1,[Thread-1]
  22:11:49.422 [Thread-4] INFO com.holelin.sundry.demo.CountDownLatchTest - Finish 4,[Thread-4]
  22:11:49.454 [Thread-0] INFO com.holelin.sundry.demo.CountDownLatchTest - Finish 0,[Thread-0]
  22:11:49.476 [Thread-3] INFO com.holelin.sundry.demo.CountDownLatchTest - Finish 3,[Thread-3]
  22:11:49.476 [main] INFO com.holelin.sundry.demo.CountDownLatchTest - 主线程:在所有任务运行完成后，进行结果汇总
  ```

#### **CountDownLatch的不足**

* CountDownLatch是一次性的，计算器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用

#### CountDownLatch源码

##### 构造函数

```java
    /**
     * Constructs a {@code CountDownLatch} initialized with the given count.
     *
     * @param count the number of times {@link #countDown} must be invoked
     *        before threads can pass through {@link #await}
     * @throws IllegalArgumentException if {@code count} is negative
     */
    public CountDownLatch(int count) {
        // count小于0抛出异常
        if (count < 0) throw new IllegalArgumentException("count < 0");
        // 创建Sync对象
        this.sync = new Sync(count);
    }
```

```java
  		Sync(int count) {
            // 设置state字段的值为count
            setState(count);
        }
```

#### `await`方法

```java
    public void await() throws InterruptedException {
        // 调用AQS中的方法，在该方法中会调用tryAcquireShared方法
        sync.acquireSharedInterruptibly(1);
    }

    public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
        // 线程被中断过抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        // tryAcquireShared也是一个模板方法 需要使用的子类去实现
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

    // CountDownLatch.Sync中的实现
    protected int tryAcquireShared(int acquires) {
        // state = 0代表获取成功
        return (getState() == 0) ? 1 : -1;
    }
```

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    // 在阻塞队列中插入共享模式的节点
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        // 自旋
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                // 尝试获取共享锁
                int r = tryAcquireShared(arg);
                // 大于0代表获取成功
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            // 会将前置节点的状态值修改为 -1  并调用LockSupport.park方法阻塞线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

```

##### `countDown`方法

> 这个方法用于减少count值，当值降低为0时代表释放锁成功，唤醒主线程

```java
public void countDown() {
    // 调用AQS中的释放共享锁的方法，在releaseShared方法中会调用tryReaseShared方法
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    // 模板方法  需要在CountDownLatch中实现
    if (tryReleaseShared(arg)) {
       
        doReleaseShared();
        return true;
    }
    return false;
}

protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    // 自旋
    for (;;) {
        // 当前的count
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        // CAS操作修改state的值
        if (compareAndSetState(c, nextc))
            // 修改后的值为0代表释放成功
            return nextc == 0;
    }
}

private void doReleaseShared() {
    // 自旋
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                // 唤醒下个节点  LockSupport.unpark
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```





