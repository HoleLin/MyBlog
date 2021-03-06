---
title: JVM(一)-内存结构
date: 2021-05-31 22:30:31
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- 内存结构
categories:
- Java
- JVM
---

### 参考文献

* [黑马程序员JVM完整教程，全网超高评价，全程干货不拖沓](https://www.bilibili.com/video/BV1yE411Z7AP?p=1)
* 深入理解Java虚拟机:JVM高级特性与最佳实践(第3版)

### JVM简述

> Java Virtual Machine,Java程序的运行环境(Java二进制字节码的运行环境);
>
> 好处: 
>
> * 一次编写,到处运行
> * 自动内存管理,垃圾回收机制
> * 数组下标越界检查

### 内存结构

<img src="https://www.holelin.cn/img/jvm/JVM架构.png" alt="JVM整体架构图" style="zoom: 80%;" />

#### 程序计数器

> Program Counter Register是一块较小的内存空间,它可以看作是**当前线程锁执行的字节码的行号指示器**;

* 在JVM的概念模型里,字节码解释器工作是就是通过改变这个计数器的值来**选取下一条需要执行的字节码指令**,它是程序控制流的指示器,分支,循环,跳转,异常处理,线程恢复等基础功能都需要依赖这个计数器来完成.

 * 由于JVM的多线程是是通过线程轮流切换,分配处理器执行时间的方式来实现,在任何一个确定的时刻,一个处理器(对于多核处理器说是一个内核)都只会执行一条线程中的指令.因此,为了线程切换后能回复到正确的执行位置,**每条线程都需要有一个独立的程序计数器,各条线程之间计数器互不影响,独立存储,我们称这个类内存区域为"线程私有"的内存.**
 * 若线程正在执行的是一个Java方法,这个计数器记录的是正在执行的虚拟机字节码指令的地址;若正在执行的是本地(Native)方法,这个计数器值则应该为空(Undefined).
* **此内存区域是唯一一个在<<Java虚拟机规范>>中没有规定任何`OutOfMemoryError`情况的区域**

#### Java虚拟机栈

> Java Virtual Machine Stack 

* Java虚拟机栈也是**线程私有**的,它的生命周期也线程相同;

* 虚拟机栈描述的是Java方法执行的线程内存模型:

  * 每个方法被执行的时候,Java虚拟机都会同步创建**一个栈帧**(Stack Frame)用于存储**局部变量表**,**操作数栈**,**动态连接**,**方法出口**等信息;(**每个线程只能有一个活动栈帧,对应着当前正在执行的方法**);
    * 局部变量表存放了编译期可知的各种Java虚拟机基本数据类型(boolean,byte,char,short,int,float,long,double),引用对象(reference类型,它不等同与对象本身,可能是一个指向对象起始地址的引用指针,也可能是指向一个代表对象的句柄或者其他与此对象相关的位置)和`returnAddress`类型(指向了一条字节码的地址);
    * 这些数据类型在局部变量表中的存储空间以局部变量槽（ Slot）来表示，其中 64 位
      长度的 long 和 double 类型的数据会占用两个变量槽，其余的数据类型只占用一个。  
  * 每个方法被调用直至执行完毕的过程,对应着一个栈帧在虚拟机中从入栈到出栈的过程;

* 对于Java虚拟机栈这个内存区域规定了两类异常状况:

  * 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 `StackOverflowError` 异常；
  * 如果 Java 虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出 `OutOfMemoryError `异常  

*  栈桢大小缺省为1M，可用参数 –Xss调整大小，例如-Xss256k 

* 常见问题: 

  * 垃圾回收是否涉及栈内存?

    > 不需要通过垃圾回收来回收内存,因为虚拟机栈中是由一个个的栈帧组成的,在方法执行完毕后,对应的栈帧会被弹出栈,所以无需要通过来及回收机制去回收内存.

  * 栈内存的分配越大越好吗?

    > 不是,因为物理内存是一定的,栈内存越大,可以支持更多的递归调用,但是可执行的线程数就会越少.

  * 方法内的局部变量是否线程安全的?

    > * 若方法内局部变量没有逃离方法的作用范围,则是线程安全的;
    > * 若局部变量引用了对象,并逃离了方法的作用范围,则需要考虑线程安全问题;

#### 本地方法栈

> Native Method Stacks

* 本地方法栈和虚拟机栈锁发挥的作用非常相似,区别在于虚拟机栈执行Java方法(也就是字节码)服务,而本地方法栈则是虚拟机使用到的本地(Native)方法服务;
* 和虚拟机栈一样,本地方法栈也会在栈深度溢出或者扩展失败的时候分别抛出`StackOverflowError`和`OutOfMemoryError`异常

#### Java堆

> Java Heap

* Java堆是虚拟机所管理的内存中最大的一块.

* **Java堆是被所有线程共享的一块内存区域,在虚拟机启动时创建.**

* 此内存区域的唯一**目的就是存放对象实例**,**Java世界里"几乎"所有的对象实例都在这里分配内存.**

* **Java堆是垃圾收集器管理的内存区域**,因此也被称为GC堆(Garbage Collected Heap);

  * 可用以下参数调整： 
    * `-Xms`：堆的最小值；
    * `-Xmx`：堆的最大值；
    * ` -Xmn`：新生代的大小；
    * `-XX:NewSize`；新生代最小值；
    * ` -XX:MaxNewSize`：新生代最大值； 例如- Xmx256m 

* 若Java堆中没有内存完成实例分配,并且堆也无法再扩展是,Java虚拟机将会抛出`OutofMemoryError`

  ```java
  // 堆内存溢出
  java.lang.OutofMemoryError ：java heap space.
  ```

* 堆内存诊断工具

  * jps
  * jmap
  * jconsole
  * jvisualvm
  
* 辨析堆和栈

  > **功能** 
  >
  > * 以栈帧的方式存储方法调用的过程，并存储方法调用过程中基本数据类型的变量（int、short、long、byte、float、double、boolean、char等）以及对象的引用变量，其内存分配在栈上，变量出了作用域就会自动释放 
  >
  > * 而堆内存用来存储Java中的对象。无论是成员变量，局部变量，还是类变量，它们指向的对象都存储在堆内存中 
  >
  > **线程独享还是共享**
  >
  > * 栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存。 
  > *  堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问。 
  >
  > **空间大小** 
  >
  > * 栈的内存要远远小于堆内存

#### 方法区

> Method Area 别名"非堆"(Non-Heap)

* 方法区和Java堆一样,是各个线程共享的内存区域
* 用于存储已经被虚拟机加载的类型信息,常量,静态变量,即时编译器编译后的代码缓存等数据.
* 可用以下参数调整：
  *  jdk1.7及以前：`-XX:PermSize`；`-XX:MaxPermSize`；
  *  jdk1.8以后：
     *  `-XX:MetaspaceSize`: 指定元空间的初始空间大小,以字节为单位,达到该值就会触发垃圾收集进行类型卸载,同时收集器会对该值进行调整: 如果释放了大量的空间,就适当降低该值;如果释放了很少的空间,那么在不超过`-XX:MaxMetaspaceSize`(如果设置了话)的情况下,适当提高该值.
     *  ` -XX:MaxMetaspaceSize`: 设置元空间的最大值,默认为-1,即不限制或者说只受限于本地内存大小.
     *  `-XX:MinMetaspaceFreeRatio`: 作用是在垃圾收集之后控制最小的元空间剩余容量的百分比,可减少因为元空间不足导致的垃圾收集频率.类似的还有`-XX:MaxMetaspaceFreeRatio`用于控制最大的元空间剩余容量的百分比.
* 若方法区无法满足新的内存分配需求时,将抛出`OutOfMemoryError`异常
  * 1.8以前会导致**永久代**内存溢出
  * 1.8以后会导致**元空间**内存溢出

<img src="https://www.holelin.cn/img/jvm/方法区.png" alt="方法区" style="zoom: 80%;" />

#### 运行时常量池

> Runtime Constant Pool

* 运行是常量池是方法区的一部分.Class文件中出了有类的版本,字段,方法,接口等描述信息外,还有一项信息是常量池表(Constant Pool Table),用于存放编译期生成的各种字面量和符号引用,这部分内容将在类加载后存放到方法区的运行是常量池中.

* 除了保存Class文件中描述的符号引用外,还会把有符号引用翻译出来的直接引用也存储在运行时常量池中.
  * 运行时常量池相对于Class文件常量池的另外一个重要特征是具备**动态性**,Java语言并不要要求常量一定只有编译期才能产生,就是说,并非预置入Class文件中常量池的内容才能进入方法区运行时常量池,运行是期间也可以将新的常量放入池中,这种特性被开发人员利用比较多的便是String类的intern()方法;
  
* 既然运行时常量池是方法区的一部分,自然受到方法区内存的限制,当常量池无法再申请到内存时会抛出`OutOfMemoryError`异常

* **通过反编译来查看类的信息**

  > * 编译.java文件生成.class: `javac Test.java`
  > * 反编译.class文件,`javap -v Test.class`

* 常量池与串池的关系

  > 串池(StringTable)
  >
  > **特征**
  >
  > - 常量池中的字符串仅是符号，只有**在被用到时才会转化为对象**
  > - 利用串池的机制，来避免重复创建字符串对象
  > - 字符串**变量**拼接的原理是`StringBuilder`
  > - 字符串**常量**拼接的原理是**编译器优化**
  > - 可以使用**intern方法**，主动将串池中还没有的字符串对象放入串池中
  > - **注意**：无论是串池还是堆里面的字符串，都是对象
  > - 用来放字符串对象且里面的**元素不重复**
  > - 常量池中的信息，都会被加载到运行时常量池中，但这是a b ab 仅是常量池中的符号，**还没有成为Java字符串**
  > - **注意**：字符串对象的创建都是**懒惰的**，只有当运行到那一行字符串且在串池中不存在的时候时，该字符串才会被创建并放入串池中。
  > - 使用**拼接字符串常量**的方法来创建新的字符串时，因为**内容是常量，javac在编译期会进行优化，结果已在编译期确定**
  > - 使用**拼接字符串变量**的方法来创建新的字符串时，因为内容是变量，只能**在运行期确定它的值，所以需要使用`StringBuilder`来创建**

  > ##### intern方法 1.8
  >
  > 调用字符串对象的intern方法，会将该字符串对象尝试放入到串池中
  >
  > - 如果串池中没有该字符串对象，则放入成功
  > - 如果有该字符串对象，则放入失败
  >
  > 无论放入是否成功，都会返回**串池中**的字符串对象
  >
  > **注意**：此时如果调用intern方法成功，堆内存与串池中的字符串对象是同一个对象；如果失败，则不是同一个对象

  > ##### intern方法 1.6
  >
  > 调用字符串对象的intern方法，会将该字符串对象尝试放入到串池中
  >
  > - 如果串池中没有该字符串对象，会**将该字符串对象复制一份**，再放入到串池中
  > - 如果有该字符串对象，则放入失败
  >
  > 无论放入是否成功，都会返回**串池中**的字符串对象
  >
  > **注意**：此时无论调用intern方法成功与否，串池中的字符串对象和堆内存中的字符串对象**都不是同一个对象**

  > ##### StringTable 垃圾回收
  >
  > StringTable在内存紧张时，会发生垃圾回收
  >
  > ##### StringTable调优
  >
  > - 因为StringTable是由HashTable实现的，所以可以**适当增加HashTable桶的个数**，来减少字符串放入串池所需要的时间
  >
  >   ```
  >   -XX:StringTableSize=xxxx
  >   ```
  >
  > - 考虑是否需要将字符串对象入池
  >
  >   可以通过**intern方法减少重复入池**

#### 直接内存

> Direct Memory
>
> 不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域；如果使用了`NIO`,这块区域会被频繁使用，在Java堆内可以用`DirectByteBuffer`对象直接引用并操作； 
>
> 这块内存不受Java堆大小限制，但受本机总内存的限制，可以通过`-XX:MaxDirectMemorySize`来设置（默认与堆内存最大值(由-Xmx指定)一样），所以也会出现`OOM`异常。 
>
> - 属于操作系统，常见于`NIO`操作时，**用于数据缓冲区**
> - 分配回收成本较高，但读写性能高
> - 不受`JVM`内存回收管理

<img src="https://www.holelin.cn/img/jvm/不使用直接内存.png" alt="不使用直接内存" style="zoom:80%;" />

<img src="https://www.holelin.cn/img/jvm/使用直接内存.png" alt="使用直接内存" style="zoom:80%;" />

* 直接内存的使用

  ```java
  //通过ByteBuffer申请1M的直接内存
  ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1M);
  ```

* 直接内存的释放

  ```
  直接内存的回收不是通过JVM的垃圾回收来释放的，而是通过unsafe.freeMemory来手动释放
  ```

* ##### 直接内存的回收机制总结

  - 使用了Unsafe类来完成直接内存的分配回收，回收需要主动调用`freeMemory`方法
  - `ByteBuffer`的实现内部使用了Cleaner（虚引用）来检测`ByteBuffer`。一旦`ByteBuffer`被垃圾回收，那么会由`ReferenceHandler`来调用Cleaner的clean方法调用`freeMemory`来释放内存

### 对象的创建

#### 对象的创建过程

> 当 Java 虚拟机遇到一条字节码 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程 .
>
> 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定,为对象分配空间的任务实际上便等同于把一块确定大小的内存块从 Java 堆中划分出来。假设 Java 堆中内存是绝对规整的，所有被使用过的内存都被放在一边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（Bump The Pointer）。 但如果 Java 堆中的内存并不是规整的，已被使用的内存和空闲的内存相互交错在一起，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”（Free List）。选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理（Compact的能力决定。因此，当使用 Serial、 ParNew 等带压缩整理过程的收集器时，系统采用的分配算法是指针碰撞，既简单又高效；而当使用` CMS` 这种基于清除（Sweep）算法的收集器时，理论上就只能采用较为复杂的空闲列表来分配内存。  
>
> 除如何划分可用空间之外，还有另外一个需要考虑的问题：对象创建在虚拟机中是非常频繁的行为，即使仅仅修改一个指针所指向的位置，在并发情况下也并不是线程安全的，可能出现正在给对象 A 分配内存，指针还没来得及修改，对象 B 又同时使用了原来的指针来分配内存的情况。解决这个问题有两种可选方案：一种是对分配内存空间的动作进行同步处理——实际上虚拟机是采用 CAS 配上失败重试的方式保证更新操作的原子性；另外一种是把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在 Java 堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer， `TLAB`），哪个线程要分配内存，就在哪个线程的本地缓冲区中分配，只有本地缓冲区用完了，分配新的缓存区时才需要同步锁定。虚拟机是否使用 `TLAB`，可以通过`-XX： +/-UseTLAB` 参数来设定 .
>
> 内存分配完成之后，虚拟机必须将分配到的内存空间（但不包括对象头）都初始化为零值，如果使用了 `TLAB`的话，这一项工作也可以提前至 `TLAB `分配时顺便进行。这步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，使程序能访问到这些字段的数据类型所对应的零值 .
>
> 接下来， Java 虚拟机还要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码（实际上对象的哈希码会延后到真正调用`Object::hashCode()`方法时才计算）、对象的 `GC` 分代年龄等信息。这些信息存放在对象的对象头（Object Header）之中。根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式 .
>
> 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了。但是从Java 程序的视角看来， 对象创建才刚刚开始——构造函数，即 Class 文件中的<init>()方法还没有执行，所有的字段都为默认的零值，对象需要的其他资源和状态信息也还没有按照预定的意图构造好。一般来说（由字节码流中 new 指令后面是否跟随 `invokespecial`指令所决定， Java 编译器会在遇到 new 关键字的地方同时生成这两条字节码指令，但如果直接通过其他方式产生的则不一定如此）， new 指令之后会接着执行<init> ()方法，按照程序员的意愿对对象进行初始化，这样一个真正可用的对象才算完全被构造出来。  

#### 对象的内存布局

在 `HotSpot` 虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：

* 对象头(Header）

* 实例数据（Instance Data）
* 对齐填充（Padding）

##### 对象头

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

* HotSpot 虚拟机对象的对象头部分包括两类信息。

  * 第一类是**用于存储对象自身的运行时数据**.如哈希码（HashCode）、 GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等，这部分数据的长度在 32 位和 64 位的虚拟机（未开启压缩指针）中分别为 32 个比特和 64 个比特，官方称它为“**Mark Word**”。  

    | 存储类型                           | 标志位 | 状态             |
    | ---------------------------------- | ------ | ---------------- |
    | 对象哈希码,对象分代年龄            | 01     | 未锁定           |
    | 指向锁记录的指针                   | 00     | 轻量级锁定       |
    | 指向重量级锁的指针                 | 10     | 膨胀(重量级锁定) |
    | 空,不需要记录信息                  | 11     | GC标记           |
    | 偏向线程ID,偏向时间戳,对象分代年龄 | 01     | 可偏向           |

  * 对象头的另外一部分是**类型指针**，即对象指向它的类型元数据的指针， Java 虚拟机通过这个指针来确定该对象是哪个类的实例.并不是所有的虚拟机实现都必须在对象数据上保留类型指针,换句话说,查找对象的元数据信息并不一定要经过对象本身.此外如果对象是一个Java数据组,那么对象头中还必须有一块用于记录数组长度的数据,因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小,但是如果数据的长度是不确定的,将无法通过元数据中的信息推断出数组的大小.

##### 实例数据

* 实例数据部分是对象真正存储的有效信息,即在程序代码里面所定义的各种类型的字段内容,无论是从父类继承下来的,还是子类中定义的字段都必须记录起来.
  * 这部分的存储顺序会受到虚拟机分片策略参数`-XX:FieldAllocationStyle`参数和字段在Java源码中定义顺序影响
* HotSpot虚拟机默认的分配顺序为`longs/doubles,ints,shorts/chars,bytes/booleans,oops(Ordinary Object Pointers,OOPs)`.相同宽度的字段总是被分配到一起存放,在满足这个前提条件下,在父类中定义的变量会出现在子类的之前.
  * 如果HotSpot虚拟机的`-XX:CompactFields`参数值为true(默认就为true),那子类之中较窄的变量也允许插入父类变量的空隙之中,以节省出一点点空间.

##### 对齐填充

* 这并不是必然存在,也没有特别含义,它仅起到占位符的作用.由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍,即任何对象的大小都必须是8字节的整数倍.

### 对象的访问定位

* Java程序会通过栈上的`reference`数据来操作堆上的具体对象.由于`reference`类型在Java虚拟机规范里面只规定了它是一个指向对象的引用,并没有定义这个引用应该通过什么方式去定位,访问到堆中对象的具体位置,所以对象访问也是由虚拟机实现而定的.

* 主流的访问方式有**使用句柄**和**直接指针**两种.
* 使用句柄访问的话,Java堆中将可能会划分出一块内存作为**句柄池**,`reference`中存储的就是对象的句柄地址,而句柄中包含了**对象实例数据**和**类型数据**各自具体的地址信息.
  * 使用句柄来访问的最大好处就是`reference`中存储的是稳定句柄地址,在对象被移动(垃圾收集是移动对象是非常普遍的行为)时只会改变句柄中的实例数据指针,而`reference`本身不需要被修改.
* 使用直接指针访问的话,Java堆中对象的内存布局就必须考虑如何放置访问类型数据的相关信息,`reference`中存储的直接就是对象地址,如果只是访问对象本身的haul,就不需要多一次间接访问的开销.
  * 使用直接指针来访问的最大好处就是速度更快,它节省了一次指针定位的时间开销.HotSpot主要使用直接指针的方式进行对象访问.

