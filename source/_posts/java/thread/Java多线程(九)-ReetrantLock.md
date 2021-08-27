---
title: Java多线程(九)-ReentrantLock
mermaid: true
date: 2021-06-23 00:06:08
cover: /img/cover/Java.jpg
tags:
- 多线程
- ReentrantLock
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

### 特点

* 可中断; `reentrantLock.lockInterruptibly();`
* 可设置超时时间;`reentrantLock.ryLock(long timeout, TimeUnit unit);`
* 可以设置为公平锁;`new ReentrantLock(true);`
* 支持多个条件变量;
* 与`synchronized`一样都支持可重入;

#### 基本用法

```java
		ReentrantLock reentrantLock = new ReentrantLock();
        reentrantLock.lock();
        try {
            // 临界区
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 释放锁
            reentrantLock.unlock();
        }
```

#### 条件变量

> `synchronized`中也有条件变量,就是那个`WaitSet`休息室,当条件不满足进入`WaitSet`等待`ReentrantLock`的条件变量比`synchronized`强大之处在于,它是支持多个条件变量的,这就好比:
>
> * `synchronized`是哪些不满足的条件都在一个休息室等消息;
> * 而`ReentrantLock`支持多间休息室,有专门等烟的休息室,专门等早餐的休息室,唤醒也是按休息室来唤醒;

* 使用要点
  * `await`前需要获得锁;
  * `await`执行后,会释放锁,进入`conditionObject`等待;
  * `await`的线程被唤醒(或打断,或超时),会重新竞争lock锁;
  * 竞争lock锁成功后,从`await`后继续执行

```java
package com.holelin.sundry.test.thread;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j
public class ReentrantLockTest {
    private static ReentrantLock reentrantLock = new ReentrantLock();
    private static Condition waitCigaretteQueue = reentrantLock.newCondition();
    private static Condition waitBreakfastQueue = reentrantLock.newCondition();
    private static volatile boolean hasCigarette = false;
    private static volatile boolean hasBreakfast = false;

    public static void main(String[] args) {
        new Thread(() -> {
            reentrantLock.lock();
            try {
                while (!hasCigarette) {
                    try {
                        waitCigaretteQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.info("获取到烟");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                reentrantLock.unlock();
            }
        }).start();

        new Thread(() -> {
            reentrantLock.lock();
            try {
                while (!hasBreakfast) {
                    try {
                        waitBreakfastQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.info("获取到早饭");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                reentrantLock.unlock();
            }
        }).start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        sendBreakfast();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        sendCigarette();
    }

    private static void sendBreakfast() {
        reentrantLock.lock();
        try {
            log.info("送早餐");
            hasBreakfast = true;
            waitBreakfastQueue.signal();
        } finally {
            reentrantLock.unlock();
        }
    }

    private static void sendCigarette() {
        reentrantLock.lock();
        try {
            log.info("送烟");
            hasCigarette = true;
            waitCigaretteQueue.signal();
        } finally {
            reentrantLock.unlock();
        }
    }
}

22:40:43.703 [main] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 送早餐
22:40:43.704 [Thread-1] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 获取到早饭
22:40:44.712 [main] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 送烟
22:40:44.712 [Thread-0] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 获取到烟
```

### 原理

#### 非公平锁实现原理

##### 加锁/释放锁流程

* 先从构造器入手,默认为非公平锁实现

  ```java
      /**
       * Creates an instance of {@code ReentrantLock}.
       * This is equivalent to using {@code ReentrantLock(false)}.
       */
      public ReentrantLock() {
          sync = new NonfairSync();
      }
  ```

* `NonfairSync`继承自`AQS`

  ```java
          /**
           * Performs lock.  Try immediate barge, backing up to normal
           * acquire on failure.
           */
          final void lock() {
              // 首先用CAS尝试(仅尝试一次),将state从0改为1,如果成功表示获得了独占锁;
              if (compareAndSetState(0, 1))
                  // 初始状态state为0,没有竞争,直接获取锁
                  setExclusiveOwnerThread(Thread.currentThread());
              else
                  // state不为0,如果尝试失败,进入acquire
                  acquire(1);
          }
  ```
  ###### 加锁

* 没有出现竞争

    ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync1.png)

* 第一个竞争出现时

    ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync2.png)

    * `CAS`尝试将state由0改为1,结果失败;

    * 进入`acquire`逻辑,进而进入`tryAcquire`逻辑,这是`state`已经是1,结果仍然失败;

      ```java
           	/** Marker to indicate a node is waiting in exclusive mode */
              static final Node EXCLUSIVE = null;
      		 // ==> arg为1
              public final void acquire(int arg) {
                  // tryAcquire方法执行,当前state为1且当前线程不是获取锁的线程,方法返回false
                  if (!tryAcquire(arg) &&
                     // 当tryAcquire返回false时,先调用addWaiter,接着acquireQueued
                      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                      selfInterrupt();
              }
      		 // ==> 
              protected final boolean tryAcquire(int acquires) {
                  return nonfairTryAcquire(acquires);
              }
      		 // ==> 
              /**
               * Performs non-fair tryLock.  tryAcquire is implemented in
               * subclasses, but both need nonfair try for trylock method.
               */
              final boolean nonfairTryAcquire(int acquires) {
                  final Thread current = Thread.currentThread();
                  int c = getState();
                  // 如果还没有获得锁
                  if (c == 0) {
                      // 尝试CAS获得,这里体现了非公平性,不去检查AQS队列
                      if (compareAndSetState(0, acquires)) {
                          setExclusiveOwnerThread(current);
                          return true;
                      }
                  }
                  // 如果已经获得了锁,线程还是当前线程,表示发生了锁重入
                  else if (current == getExclusiveOwnerThread()) {
                      int nextc = c + acquires;
                      if (nextc < 0) // overflow
                          throw new Error("Maximum lock count exceeded");
                      setState(nextc);
                      return true;
                  }
                  // 获取失败,回到调用处
                  return false;
              }
      ```
    
    * 接下来进入`addWaiter`逻辑,构造`Node`队列
    
      ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync3.png)
    
      ```java
         /**
           * Creates and enqueues node for current thread and given mode.
           *
           * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
           * @return the new node
           */
          private Node addWaiter(Node mode) {
              // 将当前线程关联到一个Node对象上,模式为独占模式
              Node node = new Node(Thread.currentThread(), mode);
              // Try the fast path of enq; backup to full enq on failure
              // 如果tail不为null,CAS尝试将Node对象加入AQS队列尾部
              Node pred = tail;
              if (pred != null) {
                  node.prev = pred;
                  if (compareAndSetTail(pred, node)) {
                      // 双向链表
                      pred.next = node;
                      return node;
                  }
              }
              // 第一次进入,队列为空,直接入队 
              // 尝试将Node加入AQS队列
              enq(node);
              return node;
          }
      	 // ==>
          /**
           * Inserts node into queue, initializing if necessary. See picture above.
           * @param node the node to insert
           * @return node's predecessor
           */
          private Node enq(final Node node) {
              for (;;) {
                  Node t = tail;
                  // 队列中还没有元素tail为null
                  if (t == null) { // Must initialize
                      // 将head从null->dumny 设置head为哨兵节点(不对应线程,状态为0)
                      if (compareAndSetHead(new Node()))
                          tail = head;
                  } else {
                      // 将node的prev设置为原来的tail
                      node.prev = t;
                      // 将tail从原来的tail设置为node
                      if (compareAndSetTail(t, node)) {
                          // 原来的tail的next设置为node
                          t.next = node;
                          return t;
                      }
                  }
              }
          }
      ```
    
      * 图中黄色三角表示该Node的`waitSatus`状态,其中0为默认正常状态
      * `Node`的创建是懒惰的
      * 其中第一个`Node`称为`Dummy`(哑元)或哨兵,用来占位,并不关联线程;
    
    * 当前线程进入`acquireQueued`逻辑
    
      ```java
      
          /**
           * Acquires in exclusive uninterruptible mode for thread already in
           * queue. Used by condition wait methods as well as acquire.
           *
           * @param node the node
           * @param arg the acquire argument
           * @return {@code true} if interrupted while waiting
           */
          final boolean acquireQueued(final Node node, int arg) {
              boolean failed = true;
              try {
                  boolean interrupted = false;
                  for (;;) {
                      // 获取node的前驱节点
                      final Node p = node.predecessor();
                      // 若上个节点是head,表示轮到自己(当前线程对应的Node)了,尝试获取锁
                      // 若前驱节点p为head节点,则再次尝试获取锁
                      if (p == head && tryAcquire(arg)) {
                          // 获取成功,设置自己(当前线程对应的Node)为head
                          setHead(node);
                          p.next = null; // help GC
                          failed = false;
                          // 返回中断标记false
                          return interrupted;
                      }
                      // 判断是否应当park
                      if (shouldParkAfterFailedAcquire(p, node) &&
                          // park等待,此时Node的状态被设置为Node.SIGNAL
                          parkAndCheckInterrupt())
                          interrupted = true;
                  }
              } finally {
                  if (failed)
                      cancelAcquire(node);
              }
          }
          /**
           * Convenience method to park and then check if interrupted
           *
           * @return {@code true} if interrupted
           */
          private final boolean parkAndCheckInterrupt() {
              LockSupport.park(this);
              return Thread.interrupted();
          }
      ```
      
      * `acquireQueued`会在一个死循环中不断尝试获得锁,然后失败后进入`park`阻塞;
      
      * 如果自己是紧邻着`head`(排在第二位),那么在此`tryAcquire`尝试获取锁,当然此时`state`仍为1,获取锁失败;
      
      * 进入`shouldParkAfterFailedAcquire`逻辑,将前驱`node`,即`head`的`waitStatus`改为-1(`Node.SIGNAL`),这次返回`false`;
      
        ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync4.png)
      
        ```java
             /** waitStatus value to indicate successor's thread needs unparking */
             static final int SIGNAL    = -1;
        	/**
             * Checks and updates status for a node that failed to acquire.
             * Returns true if thread should block. This is the main signal
             * control in all acquire loops.  Requires that pred == node.prev.
             *
             * @param pred node's predecessor holding status
             * @param node the node
             * @return {@code true} if thread should block
             */
            private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
                // 获取上个节点的状态
                int ws = pred.waitStatus;
                if (ws == Node.SIGNAL)
                    // 第二次进入waitStatus==-1该分支
                    /*
                     * This node has already set status asking a release
                     * to signal it, so it can safely park.
                     */
                    // 上个节点阻塞,那么自己也阻塞好了
                    return true;
                // > 0 表示取消状态
                if (ws > 0) {
                    /*
                     * Predecessor was cancelled. Skip over predecessors and
                     * indicate retry.
                     */
                     // 上一个节点取消,那么重构删除前面所有取消的节点,返回到外层循环重试
                    do {
                        node.prev = pred = pred.prev;
                    } while (pred.waitStatus > 0);
                    pred.next = node;
                } else {
                    // 第一次进入waitStatus==0,执行该处逻辑
                    /*
                     * waitStatus must be 0 or PROPAGATE.  Indicate that we
                     * need a signal, but don't park yet.  Caller will need to
                     * retry to make sure it cannot acquire before parking.
                     */
                    // 这次还没有阻塞
                    // 但下次如果重试不成功,则需要阻塞,这是需要设置上个节点状态为Node.SIGNAL
                    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
                }
                return false;
            }
        ```
      
      * `shouldParkAfterFailedAcquire`执行完毕回到`acquireQueued`,返回`false`,再次`tryAcquire`尝试获取锁,当然此时`state`仍为-1(`Node.SIGNAL`),获取锁失败;
      
      * 当再次进入`shouldParkAfterFailedAcquire`时,这是因为其前驱node的`waitStatus`已经为-1,这次返回`true`;
      
      * 进入`parkAndCheckInterrupt`逻辑,`Thread-1 park` (深灰色表示)
      
        ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync5.png)
      
    * 再次多个线程经历上述过程竞争失败后,变成下图所示
    
      ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync6.png)


  ###### 释放锁

  ```java
      /**
       * Attempts to release this lock.
       *
       * <p>If the current thread is the holder of this lock then the hold
       * count is decremented.  If the hold count is now zero then the lock
       * is released.  If the current thread is not the holder of this
       * lock then {@link IllegalMonitorStateException} is thrown.
       *
       * @throws IllegalMonitorStateException if the current thread does not
       *         hold this lock
       */
      public void unlock() {
          sync.release(1);
      }
  	 // 	==>
      /**
       * Releases in exclusive mode.  Implemented by unblocking one or
       * more threads if {@link #tryRelease} returns true.
       * This method can be used to implement method {@link Lock#unlock}.
       *
       * @param arg the release argument.  This value is conveyed to
       *        {@link #tryRelease} but is otherwise uninterpreted and
       *        can represent anything you like.
       * @return the value returned from {@link #tryRelease}
       */
      public final boolean release(int arg) {
          // 尝试释放锁
          if (tryRelease(arg)) {
              // 队列头节点unpark
              Node h = head;
              // 队列不为null
              if (h != null && 
                  // waitStatus == Node.SIGNAL才需要unpark
                  h.waitStatus != 0)
                  // unpark AQS中等待的线程
                  unparkSuccessor(h);
              return true;
          }
          return false;
      }
  ```

* `Thread-0`释放锁,进入`tryRelease`流程,如果成功
  
    ```java
    
    		protected final boolean tryRelease(int releases) {
                // state--
                int c = getState() - releases;
                if (Thread.currentThread() != getExclusiveOwnerThread())
                    throw new IllegalMonitorStateException();
                boolean free = false;
                // 支持锁重入,只有state减为0,才能释放成功
                if (c == 0) {
                    free = true;
                    setExclusiveOwnerThread(null);
                }
                setState(c);
              return free;
            }
    ```
    
    *  设置`exclusiveOwnerThread`为null;
  *  `state`为0;
  
  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync7.png)
  
* 当队列不为null,并且`head`的`waitStatus`为-1,进入`unparkSuccessor`流程

    ```java
        /**
         * Wakes up node's successor, if one exists.
         *
         * @param node the node
         */
        private void unparkSuccessor(Node node) {
            /*
             * If status is negative (i.e., possibly needing signal) try
             * to clear in anticipation of signalling.  It is OK if this
             * fails or if status is changed by waiting thread.
             */
            // 当前node.waitStatus为head.waitStatus,即为-1
            int ws = node.waitStatus;
            if (ws < 0)
                // 执行完成后,head.waitStatus为0
                compareAndSetWaitStatus(node, ws, 0);
    
            /*
             * Thread to unpark is held in successor, which is normally
             * just the next node.  But if cancelled or apparently null,
             * traverse backwards from tail to find the actual
             * non-cancelled successor.
             */
            // 找到需要unpark的节点,但本节点从AQS队列中脱离,是由唤醒节点完成
            // 不考虑已取消的节点,从AQS队列从后至前找到队列最前面需要unpark的节点
            Node s = node.next;
            if (s == null || s.waitStatus > 0) {
                s = null;
                for (Node t = tail; t != null && t != node; t = t.prev)
                    if (t.waitStatus <= 0)
                        s = t;
            }
            // head的后继节点(Thread-1)不为空
            if (s != null)
                // 唤醒(Thread-1)
                LockSupport.unpark(s.thread);
        }
    ```

    * 找到队列中离`head`最近一个Node(没有取消),`unpark`恢复其运行,本例中即为`Thread-1`,回到`Thread-1`的`acquireQueued`流程

* 若加锁成功(没有竞争),会设置

    * `exclusiveOwnerThread`为`Thread-1`,`state`为1;
    * `head`指向`Thread-1`所在Node,该`Node`清空`Thread`;
    * 原来的`head`因为从链表断开,因而可以被垃圾回收;

    ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync8.png)

* 若此时又有其他线程来竞争(非公平的体现),例如此时`Thread-4`来了,又碰巧被`Thread-4`竞争获取锁

  * `Thread-4`被设置`exclusiveOwnerThread`,`state`为1;
  * `Thread-1`再次进入`acquireQueued`流程,获取锁失败,重新进入`park`阻塞

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/ReentrantLock-NonfairSync9.png)

#### 可打断模式

```java
    /**
     * Acquires in exclusive mode, aborting if interrupted.
     * Implemented by first checking interrupt status, then invoking
     * at least once {@link #tryAcquire}, returning on
     * success.  Otherwise the thread is queued, possibly repeatedly
     * blocking and unblocking, invoking {@link #tryAcquire}
     * until success or the thread is interrupted.  This method can be
     * used to implement method {@link Lock#lockInterruptibly}.
     *
     * @param arg the acquire argument.  This value is conveyed to
     *        {@link #tryAcquire} but is otherwise uninterpreted and
     *        can represent anything you like.
     * @throws InterruptedException if the current thread is interrupted
     */
    public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 如果没有获得锁
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
```

```java
    /**
     * Acquires in exclusive interruptible mode.
     * @param arg the acquire argument
     */
    private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                // 获取node的前驱节点
                final Node p = node.predecessor();
                // 若上个节点是head,表示轮到自己(当前线程对应的Node)了,尝试获取锁
                // 若前驱节点p为head节点,则再次尝试获取锁
                if (p == head && tryAcquire(arg)) {
                    // 获取成功,设置自己(当前线程对应的Node)为head
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                // 判断是否应当park
                if (shouldParkAfterFailedAcquire(p, node) &&
                        // park等待,此时Node的状态被设置为Node.SIGNAL
                        parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
![img](https://www.holelin.cn/img/java/thread/reentrantlock/acquireQueued%E5%92%8CdoAcquireInterruptibly%E5%AF%B9%E6%AF%94.png)

#### 公平锁实现原理

```java
    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 先检查AQS队列中是否有前驱节点,没有才去竞争
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

```java
	// java.util.concurrent.locks.AbstractQueuedSynchronizer#acquire
	public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E5%85%AC%E5%B9%B3%E9%94%81%E4%B8%8E%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81tryAcquire%E5%AF%B9%E6%AF%94.png)

```java
    /**
     * Queries whether any threads have been waiting to acquire longer
     * than the current thread.
     *
     * <p>An invocation of this method is equivalent to (but may be
     * more efficient than):
     *  <pre> {@code
     * getFirstQueuedThread() != Thread.currentThread() &&
     * hasQueuedThreads()}</pre>
     *
     * <p>Note that because cancellations due to interrupts and
     * timeouts may occur at any time, a {@code true} return does not
     * guarantee that some other thread will acquire before the current
     * thread.  Likewise, it is possible for another thread to win a
     * race to enqueue after this method has returned {@code false},
     * due to the queue being empty.
     *
     * <p>This method is designed to be used by a fair synchronizer to
     * avoid <a href="AbstractQueuedSynchronizer#barging">barging</a>.
     * Such a synchronizer's {@link #tryAcquire} method should return
     * {@code false}, and its {@link #tryAcquireShared} method should
     * return a negative value, if this method returns {@code true}
     * (unless this is a reentrant acquire).  For example, the {@code
     * tryAcquire} method for a fair, reentrant, exclusive mode
     * synchronizer might look like this:
     *
     *  <pre> {@code
     * protected boolean tryAcquire(int arg) {
     *   if (isHeldExclusively()) {
     *     // A reentrant acquire; increment hold count
     *     return true;
     *   } else if (hasQueuedPredecessors()) {
     *     return false;
     *   } else {
     *     // try to acquire normally
     *   }
     * }}</pre>
     *
     * @return {@code true} if there is a queued thread preceding the
     *         current thread, and {@code false} if the current thread
     *         is at the head of the queue or the queue is empty
     * @since 1.7
     */
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        // h != t 时表示队列中有Node
        return h != t &&
                //  (s = h.next) == null 表示队列中还有没有第二个节点
                // 或者队列中第二个节点不是当前线程
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

####  条件变量实现原理

> 每个条件变量其实就对应着一个等待队列,其实现类是`java.util.concurrent.locks.AbstractQueuedSynchronizer.ConditionObject`

##### `await`流程

* 开始`Thread-0`持有锁,调用`await`,进入`ConditionObject`的`addConditionWaiter`流程

  ```java
        /**
           * Implements interruptible condition wait.
           * <ol>
           * <li> If current thread is interrupted, throw InterruptedException.
           * <li> Save lock state returned by {@link #getState}.
           * <li> Invoke {@link #release} with saved state as argument,
           *      throwing IllegalMonitorStateException if it fails.
           * <li> Block until signalled or interrupted.
           * <li> Reacquire by invoking specialized version of
           *      {@link #acquire} with saved state as argument.
           * <li> If interrupted while blocked in step 4, throw InterruptedException.
           * </ol>
           */
          public final void await() throws InterruptedException {
              if (Thread.interrupted())
                  throw new InterruptedException();
              Node node = addConditionWaiter();
              int savedState = fullyRelease(node);
              int interruptMode = 0;
              while (!isOnSyncQueue(node)) {
                  LockSupport.park(this);
                  if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                      break;
              }
              if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                  interruptMode = REINTERRUPT;
              if (node.nextWaiter != null) // clean up if cancelled
                  unlinkCancelledWaiters();
              if (interruptMode != 0)
                  reportInterruptAfterWait(interruptMode);
          }
  ```

  * 创建新的Node状态为-2(`Node.CONDITION`),关联`Thread-0`,加入等待队列尾部;

  ```java
          /** waitStatus value to indicate thread is waiting on condition */
          static final int CONDITION = -2;
  		/**
           * Adds a new waiter to wait queue.
           * @return its new wait node
           */
          private Node addConditionWaiter() {
              Node t = lastWaiter;
              // If lastWaiter is cancelled, clean out.
              if (t != null && t.waitStatus != Node.CONDITION) {
                  unlinkCancelledWaiters();
                  t = lastWaiter;
              }
              Node node = new Node(Thread.currentThread(), Node.CONDITION);
              if (t == null)
                  firstWaiter = node;
              else
                  t.nextWaiter = node;
              lastWaiter = node;
              return node;
          }
  ```

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86-await1.png)

* 接下来进入AQS的`fullyRelease`流程,释放同步器上的锁;

  ```java
      /**
       * Invokes release with current state value; returns saved state.
       * Cancels node and throws exception on failure.
       * @param node the condition node for this wait
       * @return previous sync state
       */
      final int fullyRelease(Node node) {
          boolean failed = true;
          try {
              int savedState = getState();
              if (release(savedState)) {
                  failed = false;
                  return savedState;
              } else {
                  throw new IllegalMonitorStateException();
              }
          } finally {
              if (failed)
                  node.waitStatus = Node.CANCELLED;
          }
      }
  ```

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86-await2.png)

* `unpark` AQS队列中的下一个节点,竞争锁,假设没有其他竞争线程,那么`Thread-1`竞争成功

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86-await3.png)

* park阻塞`Thread-0`

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86-await4.png)

##### `signal`流程

* 假设`Thread-1`要唤醒`Thread-0`

  ![img](https://www.holelin.cn/img/java/thread/reentrantlock/%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86-signal1.png)

* 进入`ConditionObject`
