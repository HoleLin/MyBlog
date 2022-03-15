---
title: Java多线程(八)-AQS
date: 2021-06-23 00:06:28
cover: /img/cover/Java.jpg
tags:
- 多线程
- AQS
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

### `AQS`原理

> 全称`AbstractQueuedSynchronizer`,是阻塞式锁和相关的同步器工具的框架;

#### 特点

* 用`status`属性来表示资源的状态(分为独占模式和共享模式),子类需要定义如何维护这个状态,控制如何获取和释放锁;
  * `getState`: 获取`state`状态;
  * `setState`: 设置`state`状态;
  * `compareAndSetState`: `CAS`机制设置`state`状态;
  * 独占模式是只有一个线程能够访问资源,而共享模式可以允许多个线程访问资源;
* 提供了基于`FIFO`的等待队列,类似于`Monitor`的`EntryList`;
* 条件变量来实现等待,唤醒机制,支持多个条件变量,类似于`Monitor`的`WaitSet`;

#### 子类主要实现的方法(默认抛出`UnsupportedOperationException`)

* **`tryAcquire`**

  ```java
      /**
       * Attempts to acquire in exclusive mode. This method should query
       * if the state of the object permits it to be acquired in the
       * exclusive mode, and if so to acquire it.
       *
       * <p>This method is always invoked by the thread performing
       * acquire.  If this method reports failure, the acquire method
       * may queue the thread, if it is not already queued, until it is
       * signalled by a release from some other thread. This can be used
       * to implement method {@link Lock#tryLock()}.
       *
       * <p>The default
       * implementation throws {@link UnsupportedOperationException}.
       *
       * @param arg the acquire argument. This value is always the one
       *        passed to an acquire method, or is the value saved on entry
       *        to a condition wait.  The value is otherwise uninterpreted
       *        and can represent anything you like.
       * @return {@code true} if successful. Upon success, this object has
       *         been acquired.
       * @throws IllegalMonitorStateException if acquiring would place this
       *         synchronizer in an illegal state. This exception must be
       *         thrown in a consistent fashion for synchronization to work
       *         correctly.
       * @throws UnsupportedOperationException if exclusive mode is not supported
       */
      protected boolean tryAcquire(int arg) {
          throw new UnsupportedOperationException();
      }
  ```

*  **`tryRelease`**

  ```java
      /**
       * Attempts to set the state to reflect a release in exclusive
       * mode.
       *
       * <p>This method is always invoked by the thread performing release.
       *
       * <p>The default implementation throws
       * {@link UnsupportedOperationException}.
       *
       * @param arg the release argument. This value is always the one
       *        passed to a release method, or the current state value upon
       *        entry to a condition wait.  The value is otherwise
       *        uninterpreted and can represent anything you like.
       * @return {@code true} if this object is now in a fully released
       *         state, so that any waiting threads may attempt to acquire;
       *         and {@code false} otherwise.
       * @throws IllegalMonitorStateException if releasing would place this
       *         synchronizer in an illegal state. This exception must be
       *         thrown in a consistent fashion for synchronization to work
       *         correctly.
       * @throws UnsupportedOperationException if exclusive mode is not supported
       */
      protected boolean tryRelease(int arg) {
          throw new UnsupportedOperationException();
      }
  ```

*  **`tryAcquireShared`**

  ```java
      /**
       * Attempts to acquire in shared mode. This method should query if
       * the state of the object permits it to be acquired in the shared
       * mode, and if so to acquire it.
       *
       * <p>This method is always invoked by the thread performing
       * acquire.  If this method reports failure, the acquire method
       * may queue the thread, if it is not already queued, until it is
       * signalled by a release from some other thread.
       *
       * <p>The default implementation throws {@link
       * UnsupportedOperationException}.
       *
       * @param arg the acquire argument. This value is always the one
       *        passed to an acquire method, or is the value saved on entry
       *        to a condition wait.  The value is otherwise uninterpreted
       *        and can represent anything you like.
       * @return a negative value on failure; zero if acquisition in shared
       *         mode succeeded but no subsequent shared-mode acquire can
       *         succeed; and a positive value if acquisition in shared
       *         mode succeeded and subsequent shared-mode acquires might
       *         also succeed, in which case a subsequent waiting thread
       *         must check availability. (Support for three different
       *         return values enables this method to be used in contexts
       *         where acquires only sometimes act exclusively.)  Upon
       *         success, this object has been acquired.
       * @throws IllegalMonitorStateException if acquiring would place this
       *         synchronizer in an illegal state. This exception must be
       *         thrown in a consistent fashion for synchronization to work
       *         correctly.
       * @throws UnsupportedOperationException if shared mode is not supported
       */
      protected int tryAcquireShared(int arg) {
          throw new UnsupportedOperationException();
      }
  ```

*  **`tryReleaseShared`**

  ```java
      /**
       * Attempts to set the state to reflect a release in shared mode.
       *
       * <p>This method is always invoked by the thread performing release.
       *
       * <p>The default implementation throws
       * {@link UnsupportedOperationException}.
       *
       * @param arg the release argument. This value is always the one
       *        passed to a release method, or the current state value upon
       *        entry to a condition wait.  The value is otherwise
       *        uninterpreted and can represent anything you like.
       * @return {@code true} if this release of shared mode may permit a
       *         waiting acquire (shared or exclusive) to succeed; and
       *         {@code false} otherwise
       * @throws IllegalMonitorStateException if releasing would place this
       *         synchronizer in an illegal state. This exception must be
       *         thrown in a consistent fashion for synchronization to work
       *         correctly.
       * @throws UnsupportedOperationException if shared mode is not supported
       */
      protected boolean tryReleaseShared(int arg) {
          throw new UnsupportedOperationException();
      }
  ```

*  **`isHeldExclusively`**

  ```java
      /**
       * Returns {@code true} if synchronization is held exclusively with
       * respect to the current (calling) thread.  This method is invoked
       * upon each call to a non-waiting {@link ConditionObject} method.
       * (Waiting methods instead invoke {@link #release}.)
       *
       * <p>The default implementation throws {@link
       * UnsupportedOperationException}. This method is invoked
       * internally only within {@link ConditionObject} methods, so need
       * not be defined if conditions are not used.
       *
       * @return {@code true} if synchronization is held exclusively;
       *         {@code false} otherwise
       * @throws UnsupportedOperationException if conditions are not supported
       */
      protected boolean isHeldExclusively() {
          throw new UnsupportedOperationException();
      }
  ```

#### 格式

* 获取锁

  ```java
  // 如果获取锁失败
  if(!tryAcquire(arg)){
  	// 入队,可以选择阻塞当前线程 park unpark
  }
  ```

* 释放锁

  ```java
  // 如果释放锁成功
  if(tryRelease(arg)){
  	// 让阻塞线程恢复运行
  }
  ```

#### `AQS`要实现的功能目标

* 阻塞版本获取锁`acquire`和非阻塞版本尝试获取锁`tryAcquire`
* 获取锁超时机制;
* 通过打断取消机制;
* 独占机制及共享机制;
* 条件不满足时的等待机制;

#### 实现不可重入锁

* 自定义同步器

  ```java
  import java.util.concurrent.locks.AbstractQueuedSynchronizer;
  import java.util.concurrent.locks.Condition;
  
  public class MySync extends AbstractQueuedSynchronizer {
      @Override
      protected boolean tryAcquire(int acquires) {
          if (acquires == 1) {
              if (compareAndSetState(0, 1)) {
                  setExclusiveOwnerThread(Thread.currentThread());
                  return true;
              }
          }
          return false;
      }
  
      @Override
      protected boolean tryRelease(int acquires) {
          if (acquires == 1) {
              if (getState() == 0) {
                  throw new IllegalMonitorStateException();
              }
              setExclusiveOwnerThread(null);
              setState(0);
              return true;
          }
          return false;
      }
  
      protected Condition newCondition() {
          return new ConditionObject();
      }
  
      @Override
      protected boolean isHeldExclusively() {
          return getState() == 1;
      }
  }
  ```

* 自定以锁

  ```java
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.locks.Condition;
  import java.util.concurrent.locks.Lock;
  
  public class MyLock implements Lock {
      private static MySync sync = new MySync();
  
      @Override
      public void lock() {
          sync.acquire(1);
      }
  
      @Override
      public void lockInterruptibly() throws InterruptedException {
          sync.acquireInterruptibly(1);
      }
  
      @Override
      public boolean tryLock() {
          return sync.tryAcquire(1);
      }
  
      @Override
      public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
          return sync.tryAcquireNanos(1, unit.toNanos(time));
      }
  
      @Override
      public void unlock() {
          sync.release(1);
      }
  
      @Override
      public Condition newCondition() {
          return sync.newCondition();
      }
  }
  ```

#### `AQS`基本思想

* 获取锁的逻辑

  ```java
  while(state状态不允许获取){
  	if(队列中还没有此线程){
  		// 入队并阻塞
  	}
  }
  // 当前线程出队
  ```

* 释放锁的逻辑

  ```java
  if(state状态允许了){
  	// 回复阻塞的线程(s)
  }
  ```

* 要点

  * 原子维护`state`状态
  * 阻塞以及恢复现场
  * 维护队列

##### `state`设计

* `state`使用`volatile`配合`CAS`保证其修改时的原子性;

* `state`使用了32`bit` `int`来维护同步状态,因为当时使用`long`在很多平台下测试的结果并不理想;

  ```java
      /**
       * The synchronization state.
       */
      private volatile int state;
  ```

##### 阻塞恢复设计

* 早起的控制线程暂停和恢复的`API`有`suspend`和`resume`,但它们是不可用的,因为如果先调用的`resume`那么`suspend`将感知不到;
* 解决办法是使用`park`&`unpark`来实现线程的暂停和恢复;
* `park`&`unpark`是针对线程的,而不是针对同步器的,因此控制粒度更为精细;
* `park`线程还可以通过`interrupt`打断

##### 队列设计

* 使用`FIFO`先入先出队列,并不支持优先队列;

* 设计是借鉴`CLH`队列,它是一种单向无锁队列;

  > `CLH`是一种基于单向链表的高性能、公平的自旋锁
  >
  > `CLH`好处:
  >
  > * 无锁,使用自旋;
  > * 快速,无阻塞;

* 队列中有`head`和`tail`两个指针节点,都是用`volatile`修饰配合`CAS`使用,每个节点都有`state`维护节点状态

  ```java
      /**
       * Head of the wait queue, lazily initialized.  Except for
       * initialization, it is modified only via method setHead.  Note:
       * If head exists, its waitStatus is guaranteed not to be
       * CANCELLED.
       */
      private transient volatile Node head;
  
      /**
       * Tail of the wait queue, lazily initialized.  Modified only via
       * method enq to add new wait node.
       */
      private transient volatile Node tail;
  ```

  * 入队伪代码,只需要考虑`tail`赋值的原子性

    ```java
    do{
    	// 原来的tail
    	Node prev = tail;
    	// 用CAS在原来的tail的基础上改为node
    }while(tail.compareAndSet(prev,node))
    ```

  * 出队伪代码

    ```java
    // prev是上一个节点
    while((Node prev=node.prev).state != 唤醒状态){
    }
    // 设置头节点
    head = node;
    ```

* `AQS`在一些方面改进了`CLH`

  ```java
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
              	// 将head从null->dumny
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

  
