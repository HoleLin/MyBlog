---
title: Java多线程-ReentrantLock
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

    ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync1.png)

* 第一个竞争出现时

    ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync2.png)

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
    
      ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync3.png)
    
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
                          // 上个节点为null
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
    
        ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync4.png)
    
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
    
        ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync5.png)
    
    * 再次多个线程经历上述过程竞争失败后,变成下图所示
    
      ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync6.png)


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
          if (tryRelease(arg)) {
              Node h = head;
              if (h != null && h.waitStatus != 0)
                  unparkSuccessor(h);
              return true;
          }
          return false;
      }
  ```

* `Thread-0`释放锁,进入`tryRelease`流程,如果成功
  
    ```java
    		protected final boolean tryRelease(int releases) {
                int c = getState() - releases;
                if (Thread.currentThread() != getExclusiveOwnerThread())
                    throw new IllegalMonitorStateException();
                boolean free = false;
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
    
    ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync7.png)

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

    ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync8.png)

* 若此时又有其他线程来竞争(非公平的体现),例如此时`Thread-4`来了,又碰巧被`Thread-4`竞争获取锁

  * `Thread-4`被设置`exclusiveOwnerThread`,`state`为1;
  * `Thread-1`再次进入`acquireQueued`流程,获取锁失败,重新进入`park`阻塞

  ![img](http://www.chenjunlin.vip/img/java/thread/reentrantlock/ReentrantLock-NonfairSync9.png)
