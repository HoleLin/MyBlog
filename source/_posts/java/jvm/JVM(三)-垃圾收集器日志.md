---
title: JVM(三)-垃圾收集器日志分析
date: 2022-01-04 13:40:57
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- 垃圾收集器日志
- JVM
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

* 深入理解Java虚拟机:JVM高级特性与最佳实践(第3版)
* [JEP 158: Unified JVM Logging](https://openjdk.java.net/jeps/158)
* [Tools Reference](https://docs.oracle.com/en/java/javase/11/tools/java.html#GUID-3B1CE181-CD30-4178-9602-230B800D4FAE)
* [Java HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

### 垃圾收集器日志

* HotSpot 所有功能的日志都收归到了`-Xlog`参数

  ```
  Xlog[:[selector][:[output][:[decorators][:output-options
  ```

  * 命令行中最关键的参数是选择器（Selector），它由标签（Tag）和日志级别 （Level）共同组成.标签可理解为虚拟机中某个功能模块的名字，它告诉日志框架用户希望得到虚拟机哪些功能的日志输出。垃圾收集器的标签名称为“gc”，由此可见，垃圾收集器日志只是 HotSpot 众多功能日志的其中一项，全部支持的功能模块标签名如下所示

    ```
    add,age,alloc,annotation,aot,arguments,attach,barrier,biasedlocking,blocks,bot,break 
    point,bytecode
    ```

  * 日志级别从低到高,共有`Trace`,`Debug`,`Warning`,`Error`,`Off`六种级别,日志级别决定了输出信息的详细程序,默认级别为`Info`.

  * 还可以使用修饰器(Decorator)来要求每行日志输出都附加上额外的内容,支持附加在日志行上的信息包括:

    ```
    time: 当前时间;
    uptime: 虚拟机启动到现在经过的时间,以秒为单位;
    timemillis: 当前时间毫秒数,相当于System.currentTimeMillis()的输出;
    uptimemillis: 虚拟机启动到现在经过的毫秒数;
    timenanos: 当前时间的纳秒数,相当于System.nanoTime()的输出;
    pid: 进程ID
    tid: 线程ID
    level: 日志级别
    tags: 日志输出的标签集.如果不指定,默认值为uptime,level,tags这三个,此时日志输出类似于以下形式:
    			[3.080s][info][gc,cpu] GC(5) User=0.03s Sys=0.00s Real=0.01s
    ```

#### **查看GC基本信息**

  * 在JDK9之前使用`-XX: +PrintGC`
  * 在JDK9之后使用`-Xlog: gc`

#### **查看GC详细信息**

  * 在JDK9之前使用`-XX: +PrintGCDetails`
  * 在JDK9之后使用`-Xlog: gc*`, 用通配符*将GC标签下所有细分过程都打印出来.

#### **查看GC过程中用户现场并发时间以及停顿的时间**

  * 在JDK9之前使用`-XX:+PrintGCApplicationConcurrentTime`以及`-XX: +PrintFCApplicationStoppedTime`
  * 在JDK9之后使用`-Xlog: safepoint`

#### 查看收集器Ergonmics机制自动调节的相关信息

* Ergonmics机制(自动设置对空间各分代区域大小,收集目标等内容,从Parallel收集器开始支持)

* 在JDK9之前使用`-XX: PrintAdaptiveSizePolicy`
* 在JDK9之后使用`-Xlog: gc+ergo*=trace`

#### 查看熬过收集后剩余对象的年龄分布信息

* 在JDK9之前使用`-XX: +PrintTenuringDistribution`
* 在JDK9之后使用`-Xlog: gc+age=trace`

### 内存分配与回收策略

#### **对象优先在Eden分配**:

* 大多数情况下,对象在新生代Eden区分配.当Eden区没有足够空间进行分配时,虚拟机将发起一次Minor GC.

#### **大对象直接进入老年代**

* 大对象就是指需要大量连续内存空间的 Java 对象，最典型的大对象便是那种很长的字符串，或者元素数量很庞大的数组.

  * 在 Java 虚拟机中要避免大对象的原因是，在分配空间时，它容易导致内存明明还有不少空间时就提前触发垃圾收集，以获取足够的连续空间才能安置好它们，而当复制对象时，大对象就意味着高额的内存复制开销。
  * HotSpot 虚拟机提供了`-XX: PretenureSizeThreshold` 参数，指定大于该设置值的对象直接在老年代分配，这样做的目的就是避免在 Eden 区及两个 Survivor 区之间来回复制， 产生大量的内存复制操作。

#### **长期存活的对象将进入老年代**

* HotSpot 虚拟机中多数收集器都采用了分代收集来管理堆内存，那内存回收时就必须能决策哪些存活对象应当放在新生代，哪些存活对象放在老年代中。

  * 为做到这点，虚拟机给每个对象定义了一个对象年龄（Age）计数器，存储在对象头中。对象通常在 Eden 区里诞生，如果经过第一次 Minor GC 后仍然存活，并且能被 Survivor 容纳的话，该对象会被移动到  Survivor 空间中，并且将其对象年龄设为  1 岁。对象在Survivor 区中每熬过一次  Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为  15），就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数`-XX：  MaxTenuringThreshold`设置。

#### **动态对象年龄判定**

* 为了能更好地适应不同程序的内存状况，HotSpot 虚拟机并不是永远要求对象的年龄必须达到 `-XX：MaxTenuringThreshold`才能晋升老年代.

  * **如果在 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到`-XX： MaxTenuringThreshold` 中要求的年龄。**

#### **空间分配担保**

* 在发生  Minor GC 之前，虚拟机必须先检查**老年代最大可用的连续空间是否大于新生代所有对象总空间**，如果这个条件成立，那这一次  Minor 
  GC 可以确保是安全的。如果不成立，则虚拟机会先查看`-XX:HandlePromotionFailure` 参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次  Minor GC 是有风险的；如果小于，或者`-XX:HandlePromotionFailure `设置不允许冒险，那这时就要改为进行一次  Full GC。
  * 但为了内存利用率，只使用其中一个 Survivor 空间来作为轮换备份，因此当出现大量对象在 Minor GC 后仍然存活的情况 ——最极端的情况就是内存回收后新生代中所有对象都存活，需要老年代进行分配担保,把 Survivor 无法容纳的对象直接送入老年代，这与生活中贷款担保类似.老年代要进行这样的担保，前提是老年代本身还有容纳这些对象的剩余空间，但一共有多少对象会在这次回收中活下来在实际完成内存回收之前是无法明确知道的，所以只能取之前每一次回收晋升到老年代对象容量的平均大小作为经验值，与老年代的剩余空间进行比较，决定是否进行  Full GC 来让老年代腾出更多空间。
  * 取历史平均值来比较其实仍然是一种赌概率的解决办法，也就是说假如某次  Minor GC 存活后的对象突增，远远高于历史平均值的话，依然会导致担保失败。如果出现了担保失败，那就只好老老实实地重新发起一次Full GC，这样停顿时间就很长了。虽然担保失败时绕的圈子是最大的，但通常情况下都还是会将`-XX：HandlePromotionFailure` 开关打开，避免Full GC 过于频繁.
* 在  JDK 6 Update 24 之后，这个测试结果就有了差异，`-XX:HandlePromotionFailure` 参数不会再影响到虚拟机的空间分配担保策略.
* JDK 6 Update 24 之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行  Minor GC，否则将进行 
  Full GC。



