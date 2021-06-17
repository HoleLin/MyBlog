---
title: Java多线程-锁
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 多线程
- 锁
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

* [锁的简单应用](https://www.cnblogs.com/dj3839/p/6580765.html)

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

> 每个Java对象都可以关联一个Monitor对象,如果使用`synchronized`给对象上锁(重量级锁)之后,该对象的Mark Word中就被设置指向Monitor对象的指针
>
> 
>
> **Moitor内部属性**
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

##### **优点** 

* 可重入锁的一个优点是可一定程度避免死锁

```java
public class Lock {
    boolean isLocked = false;
    Thread lockedBy = null;
    int lockedCount = 0;

    public synchronized void lock()
            throws InterruptedException {
        Thread thread = Thread.currentThread();
        while (isLocked && lockedBy != thread) {
            wait();
        }
        isLocked = true;
        lockedCount++;
        lockedBy = thread;
    }

    public synchronized void unlock() {
        if (Thread.currentThread() == this.lockedBy) {
            lockedCount--;
            if (lockedCount == 0) {
                isLocked = false;
                notify();
            }
        }
    }
}
```

#### 不可重入锁

```java
public class Lock{
    private boolean isLocked = false;
    public synchronized void lock() throws InterruptedException{
        while(isLocked){
            wait();
        }
        isLocked = true;
    }
    public synchronized void unlock(){
        isLocked = false;
        notify();
    }
}

public class Count{
    Lock lock = new Lock();
    public void print(){
        lock.lock();
        doAdd();
        lock.unlock();
    }
    public void doAdd(){
        lock.lock();
        //do something
        lock.unlock();
    }
}
```

#### 互斥锁(mutual exclusion local)

> 也称为mutex.
>
> 意味着至多只有一个线程可以拥有锁,当线程A尝试请求一个被线程B占有的锁时,线程A必须等待或者阻塞,直到B线程释放这个锁,.若线程B永远不释放锁,A将永远等待下去.

#### 自旋锁和适应性自旋锁

##### 自旋锁

##### 适应性自选锁

