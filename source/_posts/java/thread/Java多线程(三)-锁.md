---
title: Java多线程(三)-锁
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 多线程
- 锁
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

* [锁的简单应用](https://www.cnblogs.com/dj3839/p/6580765.html)
* [图解Java中那18 把锁](https://mp.weixin.qq.com/s/l6ee7k0n7CCVFgBS4tI2kQ)

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


#### 死锁

#####  产生死锁的原因

> 当前线程拥有其他线程需要的资源,当前线程等待其他线程已拥有的资源,同时不放弃自己拥有的资源;

##### 避免死锁的方式

* 固定加锁的顺序: 可以使用Hash值的大小来确定加锁的先后;
* 尽可能缩减加锁的范围,等到操作共享变量的时候才加锁;
* 使用可释放的定时锁(一段时间申请不到锁的权限,直接释放掉);

#### 管程(`Monitor`)

> 每个Java对象都可以关联一个Monitor对象,如果使用`synchronized`给对象上锁(重量级锁)之后,该对象的Mark Word中就被设置指向Monitor对象的指针.

![img](https://www.holelin.cn/img/java/thread/Monitor%E7%BB%93%E6%9E%84.png)

##### 流程说明

* 刚开始`Monitor`中`Owner`为null,当Thread-2执行`synchronized(object)`就会将`Monitor`的所有者`Owner`设置为Thread-2,**Monitor中只能有一个Owner**;
* 在Thread-2上锁的过程中,如果Thread-3,Thread-4,Thread-5也来执行`synchroinzed(object)`就会进入`EntryList`中被阻塞,线程进入`BLOCKED`状态
* Thread-2执行完成同步代码块的内容,然后唤醒`EntryList`中等待的线程来进行竞争锁,竞争是非公平的;
* 上图中`WaitSet`中Thread-0,Thread-1是之前获得过锁,但条件不满足进入`WAITING`状态;
  * **tips:**`synchronized`必须是进入同一个对象的`Monitor`才有上述效果,不加`synchronized`的对象不会关联`Monitor`,不遵从以上规则;

##### Java实现管程的方式: `synchronized`

> `synchronized`是一种互斥锁,一次只能允许一个线程进入被锁住的代码块;
>
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

<img src="https://www.holelin.cn/img/java/thread/lock.png" alt="锁的分类" style="zoom:67%;" />

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

#### `CAS`

> Compare And Swap 比较与交换,是一种无锁算法.在不适用锁(没有线程被阻塞)的情况下实现多线程之间的变量同步.`java.util.concurrent`包中的原子类就是通过`CAS`来实现了乐观锁。
>
> `CAS`的底层是`lock cmpxing`指令(x86架构),在单核CPU和多核CPU下都能保证比较-交换的原子性.
>
> 在多核状态下,某个执行到待`lock`的指令,CPU会让总线锁住,当这个核把此执行执行完毕,再开启总线,这个过程中不会被线程调度机制打断,保证了多线程对内存操作的准确性.

* `CAS`算法涉及到三个操作数

  * 需要读写的内存值V
  * 进行比较的值A
  * 要写入的新值B

  > 当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

`CAS`虽然很高效，但是它也存在三大问题，这里也简单说一下：

* **ABA问题**

  > `CAS`需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。
  >
  > - JDK从1.5开始提供了`AtomicStampedReference`类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

* **循环时间长开销大**。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

* **只能保证一个共享变量的原子操作**

  > * 对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
  >
  > - Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

#### 可重入锁与不可重入锁

##### 可重入锁

> **可重入锁又名递归锁**，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞;当一个线程请求其他线程已经占有的锁时,请求线程将被阻塞,然而内部锁`synchronized`是可重入的,因此线程在试图获取它自己占有的锁时,请求会成功.

* 可重入意味着所有的请求是基于"每个线程(per-thread)",而不是基于"每个调用(pre-invocation)"的.
* 可重入实现是通过每个锁关联**一个请求计数器(acquisition count)**和**一个占有它的线程**.当计数为0时,认为锁是未被占有的.线程请求一个未被占有的锁时,JVM将记录锁的占有者,并且将请求计数置为1.若同一线程再次请求这个锁,计数将递增,每次占用线程退出同步块,计数器值将递减,直到计数器达到0时,锁被释放.

* **可重入锁优点** 
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

##### 不可重入锁

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

#### 自旋锁,适应性自旋锁

##### 自旋锁

* **重量级锁**竞争时，还可以使用自选来优化，如果当前线程在**自旋成功**（使用锁的线程退出了同步块，**释放了锁**），这时就可以避免线程进入阻塞状态;

* `Java6`之后自旋锁是自适应的,比如对象刚刚一次的自旋操作成功过,那么认为这次自旋成功的可能性会高,就多自旋几次,反之就会少自旋升值不自旋,总之比较智能,自旋会占用CPU时间,单核CPU自旋就是浪费,多核CPU自旋才能发挥作用;

* `Java7`之后不能控制是否开启自旋功能;

  <img src="https://www.holelin.cn/img/java/thread/%E8%87%AA%E6%97%8B%E6%88%90%E5%8A%9F.png" alt="img" style="zoom:67%;" />

  <img src="https://www.holelin.cn/img/java/thread/%E8%87%AA%E6%97%8B%E5%A4%B1%E8%B4%A5.png" alt="img" style="zoom:67%;" />

##### 适应性自选锁

#### 公平锁和非公平锁

* 公平锁指的是:在竞争环境下,先到临界区的线程比后道的线程一定更快地获取得到锁;
* 非公平锁指的是: 在竞争环境下,先到临界区的线程未必比厚道的线程更快地获取到锁;
* 如果会尝试获取锁,那就是非公平的;如果不会尝试获取锁,直接进队列,再等待唤醒,那就是公平的;

##### 公平锁的实现

* 公平锁可以把竞争的线程放到一个先进先出的队列上;
* 只有持有锁的线程执行完了,唤醒队列的下一个线程去获取锁;

##### 非公平锁的实现

* 线程先尝试能不能获取得到锁,如果获取得到锁了就执行同步代码
* 如果获取不到锁,那就再把这个线程放到队列

### 锁升级(无锁|偏向锁|轻量级锁|重量级锁)

#### 偏向锁(Biased Locking)

> JVM会认为只有某个线程才会执行同步代码(没有竞争的环境)
>
> Java偏向锁(Biased Locking)是指它会偏向于第一个访问锁的线程，如果在运行过程中，只有一个线程访问加锁的资源，不存在多线程竞争的情况，那么线程是不需要重复获取锁的，这种情况下，就会给线程加一个偏向锁。
>
> 偏向锁的实现是通过控制对象`Mark Word`的标志位来实现的，如果当前是`可偏向状态`，需要进一步判断对象头存储的线程 ID 是否与当前线程 ID 一致，如果一致直接进入。
>
> **只有一个线程进入临界区,偏向锁**

* 轻量级锁在没有竞争时(就自己这个线程),每次重入任需要`CAS`操作;
* Java 6引入偏向锁来做优化,只有第一次使用`CAS`将线程ID设置到对象的`Mark Word`头之后,发现这个线程ID是自己的就表示没有竞争,不用重新`CAS`,以后只要不发生竞争,这个对象就该线程所有;
* 一个对象创建时,如果开启了偏向锁（默认开启）,对象的Mark Word的值为`Ox05`即最后三位为101,此时他的thread,epoch,age都是0;
* 但是偏向锁默认是**有延迟**的，不会再程序一启动就生效，而是会在程序运行一段时间（几秒之后），才会对创建的对象设置为偏向状态;如果想避免延迟,可加VM参数`-XX:BasicLockingStartupDelay=0`来禁用延迟;
* 如果没有开启偏向锁，那么对象创建后,`Mark Word`值为`Ox01`即最后三位为001,这时它的`hashcode`,age都为0,第一次用到`hashcode`时赋值;

##### 禁用偏向锁

* 添加VM参数:`-XX:UserBaisedLocking`

##### 撤销偏向锁

*  调用对象`hashCode`方法
  * 调用了对象的`hashCode`但偏向锁的对象`Mark Word`中存储的是线程id,如果调用`hashCode`会导致偏向锁被撤销 ;
    * 轻量级锁会在锁记录中记录`hashCode`
    * 重量级锁会在锁记录中记录`hashCode`
* 其他线程使用对象
  * 当有其他线程使用偏向锁对象时,会将偏向锁升级为轻量级锁;
* 调用`wait`-`notify`

##### 批量重偏向

##### 批量撤销

##### `wait`-`notify`

> `Owner`线程发现条件不满足,调用`wait`方法,即可进入`WaitSet`变为`WAITING`状态;
>
> `BLCOKED`和`WAITING`的线程都处于阻塞状态,不占用CPU时间片;
>
> `BLOCKED`线程会在`Owner`线程释放锁时唤醒;
>
> `WAITING`线程会`Owner`线程调用`notify`或`notifyAll`时唤醒,但唤醒后并不意味者立即获得锁,仍需进入`EntryList`重新竞争

![img](https://www.holelin.cn/img/java/thread/Monitor%E7%BB%93%E6%9E%84.png)

###### API

* `obj.wait()/obj.wait(long timeout)`让进入object监视器的线程到`WaitSet`中等待;
* `obj.notify()`在object上正在`WaitSet`等待的线程挑一个唤醒;
* `obj.notifyAll()`在object上正在`WaitSet`等待的线程全部唤醒;
* 他们都是线程进行协作的手段,都属于Object对象的方法,必须获得对象的锁,才能调用这些方法

#### 轻量级锁与重量级锁

##### 轻量级锁

> 当线程竞争变得比较激烈时，偏向锁就会升级为`轻量级锁`，轻量级锁认为虽然竞争是存在的，但是理想情况下竞争的程度很低，通过`自旋方式`等待上一个线程释放锁。
>
> **多个线程交替进入临界区,轻量级锁**

* 使用场景: 如果一个对象虽然有多线程访问,但多线程访问的时间是错开的(也就是没有竞争),那么就可以使用轻量级说来优化

* 轻量级锁对使用者是透明的,即语法任然是`synchronized`

* 说明

  * 创建**锁记录**（`Lock Record`）对象，每个线程的栈帧都会包含一个锁记录对象，内部可以存储锁定对象的`mark word`（不再一开始就使用`Monitor`）;

    <img src="https://www.holelin.cn/img/java/thread/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%811.png" alt="img" style="zoom:80%;" />

  * 让锁记录中的`Object reference`指向锁对象（Object），并尝试用`cas`去替换Object中的`mark word`，将此`mark word`放入`lock record`中保存;

    <img src="https://www.holelin.cn/img/java/thread/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%812.png" alt="img" style="zoom:80%;" />

  * 如果`CAS`替换成功，则将Object的对象头替换为**锁记录的地址**和**状态 00（轻量级锁状态）**，并由该线程给对象加锁;

    <img src="https://www.holelin.cn/img/java/thread/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%813.png" alt="img" style="zoom:80%;" />

##### 锁膨胀

* 如果一个线程在给一个对象加轻量级锁时，**`CAS`替换操作失败**（因为此时其他线程已经给对象加了轻量级锁），此时该线程就会进入**锁膨胀**过程;

  <img src="https://www.holelin.cn/img/java/thread/%E9%94%81%E8%86%A8%E8%83%801.png" alt="img" style="zoom:80%;" />

* 此时便会给对象加上重量级锁（使用Monitor）

  - 将对象头的`Mark Word`改为`Monitor`的地址，并且状态改为**01**(重量级锁)
  - 并且该线程放入入`EntryList`中，并进入阻塞状态**BLOCKED**

  <img src="https://www.holelin.cn/img/java/thread/%E9%94%81%E8%86%A8%E8%83%802.png" alt="img" style="zoom: 67%;" />

* 当Thread-0退出同步块解锁时,使用`CAS`将`Mark Word`的值恢复给对象头,失败,这时会进入重量级解锁流程,即按照`Monitor`地址找到`Monitor`对象设置Owner为null,唤醒`EntryList`中`BLOCKED`线程;

##### 重量级锁

> 如果线程并发进一步加剧，线程的自旋超过了一定次数，或者一个线程持有锁，一个线程在自旋，又来了第三个线程访问时（反正就是竞争继续加大了），轻量级锁就会膨胀为`重量级锁`，重量级锁会使除了此时拥有锁的线程以外的线程都阻塞。
>
> 升级到重量级锁其实就是互斥锁了，一个线程拿到锁，其余线程都会处于阻塞等待状态。
>
> 在 Java 中，synchronized 关键字内部实现原理就是锁升级的过程：无锁 --> 偏向锁 --> 轻量级锁 --> 重量级锁
>
> **多个线程同时进入临界区,重量级锁**

### 锁优化技术（锁粗化、锁消除）

#### **锁粗化**

* `锁粗化`就是将多个同步块的数量减少，并将单个同步块的作用范围扩大，本质上就是将多次上锁、解锁的请求合并为一次同步请求。

举个例子，一个循环体中有一个代码同步块，每次循环都会执行加锁解锁操作。

```java
private static final Object LOCK = new Object();

for(int i = 0;i < 100; i++) {
    synchronized(LOCK){
        // do some magic things
    }
}
```

经过`锁粗化`后就变成下面这个样子了：

```java
 synchronized(LOCK){
     for(int i = 0;i < 100; i++) {
        // do some magic things
    }
}
```

#### **锁消除**

* `锁消除`是指虚拟机编译器在运行时检测到了共享数据没有竞争的锁，从而将这些锁进行消除。

举个例子让大家更好理解。

```java
public String test(String s1, String s2){
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append(s1);
    stringBuffer.append(s2);
    return stringBuffer.toString();
}
```

上面代码中有一个 test 方法，主要作用是将字符串 s1 和字符串 s2 串联起来。

test 方法中三个变量s1, s2, stringBuffer， 它们都是局部变量，局部变量是在栈上的，栈是线程私有的，所以就算有多个线程访问 test 方法也是线程安全的。

我们都知道 StringBuffer 是线程安全的类，append 方法是同步方法，但是 test 方法本来就是线程安全的，为了提升效率，虚拟机帮我们消除了这些同步锁，这个过程就被称为`锁消除`。

```java
StringBuffer.class

// append 是同步方法
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```
