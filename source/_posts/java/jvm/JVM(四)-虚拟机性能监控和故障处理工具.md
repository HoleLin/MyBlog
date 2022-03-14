---
title: JVM(四)-虚拟机性能监控和故障处理工具
date: 2022-01-05 18:20:18
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- 故障处理
- 性能监控
categories:
- Java
- JVM
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

### 基础故障处理工具

#### jps：虚拟机进程状况工具

* jps（JVM Process Status Tool）
* 可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一 ID（LVMID，Local Virtual Machine Identifier）

* 命令格式

  ```sh
  jps [options] [hostid]
  ```

  * -q: 只输出LVMID,省略主类的名称;
  * -m: 输出虚拟机进程启动时传递给主类main()函数的参数;
  * -l: 输出主类的全名,如果进程执行的JAR包,则输出JAR路径;
  * -v: 输出虚拟机进程启动时的JVM参数

#### jstat：虚拟机统计信息监视工具

* jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具.

* 它可以显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据，在没有  GUI 图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。

* 命令格式

  ```sh
  jstat [option vmid [interval[s|ms]] [count]]
  ```

  * 对于命令格式中的  VMID 与  LVMID 需要特别说明一下：如果是本地虚拟机进程，VMID 与  LVMID 是一致的；如果是远程虚拟机进程，那VMID 的格式应当是：

    ```
    protocol:][//]lvmid[@hostname[:port]/servername
    ```

  * 参数 interval 和 count 代表查询间隔和次数，如果省略这 2 个参数，说明只查询一次。假设需要每 250 毫秒查询一次进程 2764 垃圾收集状况,一共查询 20 次，那命令应当是

    ```sh
    jstat -gc 2764 250 20
    ```

  * 选项  option 代表用户希望查询的虚拟机信息，主要分为三类：类加 
    载、垃圾收集、运行期编译状况

    | 选项              | 作用                                                         |
    | ----------------- | ------------------------------------------------------------ |
    | -class            | 监视类加载,卸载数量,总空间以及类装在所耗费的时间             |
    | -gc               | 监视Java堆状况,包括Eden区,2个Survivor区,老年代,永久代等容量已用空间,垃圾收集时间合计等信息 |
    | -gccapacity       | 监视内容与-gc基本相同,但输出主要关注Java堆各个区域使用到的最大,最小空间 |
    | -gcutil           | 监视内容与-gc基本相同,但输出主要关注已使用空间占总空间的百分比 |
    | -gccause          | 与-gcutil功能一样,但是会额外输出导致上一次垃圾收集产生的原因 |
    | -gcnew            | 监视新生代垃圾收集状况                                       |
    | -gcnewcapacity    | 监视内容与-gcnew基本相同,输出主要关注使用到的最大,最小空间   |
    | -gcold            | 监视老年代垃圾收集状态                                       |
    | -gcoldcapacity    | 监视内容与-gcold基本相同,输出主要关注使用到的最大,最小空间   |
    | -gcpermcapacity   | 输出永久代使用到的最大,最小空间                              |
    | -complier         | 输出即使编译器编译过的方法,耗时等信息                        |
    | -printcompilation | 输出已经被即时编译的方法                                     |

  * 示例

    ```
    jstat -gcutil 1335
      S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT
      0.00 100.00  36.45  78.33  87.59  78.01    285    2.547     0    0.000   138    1.121    3.668
    ```

    * E表示新生代Eden区
    * S0,S1表示Survivor0,Survivor1
    * O表示Old,老年代
    * P表示Permanent,永久代
    * M表示元空间

#### Jinfo: Java配置信息工具

* Jinfo(Configuration Info for Java)的作用是实时查看和调整虚拟机各项参数.

* 使用`jps -v`可以查看虚拟机启动时显式指定的参数列表,但是想知道未被显式指定的参数的系统默认值,可以使用`jinfo -flag`

* Jinfo还可以使用`-sysprops`选项把虚拟机进程的`System.getProperties()`的内容打印出来

* 命令格式:

  ```
  jinfo [option] pid
  ```

#### jmap: Java内存映像工具

* Jmap(Memory Map for Java)命令用于生成堆转储快照(一般为heapdump或dump文件).

* 如果不适用jmap命令,要想获取Java堆转储快照也可以使用`-XX:+HeapDumpOnOutOfMemoryError`参数,可以让虚拟机在内存溢出异常出现之后自动生成堆存储快照文件,也通过`-XX:+HeapDumpOnCtrlBreak`参数使用`Ctrl+Break`键让虚拟机生成堆转储快照文件,也可以在Linux系统下通过`kill -3`命令发送进程退出信号来获取堆转储快照.

* jmap的作用不仅是为了获取堆转储快照,它还可以查询 finalize执行队列、Java 堆和方法区的详细信息，如空间使用率、当前用的是哪种收集器等。

* 命令格式

  ```
  jmap [option] vmid		
  ```

  | 选项           | 作用                                                         |
  | -------------- | ------------------------------------------------------------ |
  | -dump          | 生成Java堆转储快照.格式为`-dump:[live,]format=b,file=\<filename>`,其中live子参数说明是否是dump出存活的对象 |
  | -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize方法的对象.只在Linux/Solaris平台下有效 |
  | -heap          | 显示Java堆详细信息,让使用哪种回收器,参数配置,分代状况等,只在Linux/Solaris平台下有效 |
  | -histo         | 显示堆中对象统计信息,包括类,实例数量,合计容量                |
  | -permstat      | 以ClassLoader为统计口径显示永久代内存状态,只在Linux/Solaris平台下有效 |
  | -F             | 当虚拟机进程对-dump选项没有响应时,可以使用这个选项强制生成dump快照.只在Linux/Solaris平台有效 |

#### jhat: 虚拟机堆转储快照分析工具

* jhat(JVM Heap Analysis Tool)命令与jmap搭配使用,来分析jmap生成的堆转储快照.
* jhat内置了一个微型HTTP/Web服务,生成堆转储快照的分析结果后,可以在浏览器中查看
* 替代品: VisualVM,Eclipse Memory Analyzer.

#### jstack: Java堆栈跟踪工具

* Jstack(Stack Trace for Java)命令用于生成虚拟机当前时刻的线程快照(一般称为threaddump或者javacore文件).

* 线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合,生成线程快照的目的通常是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂起等，都是导致线程长时间停顿的常见原因。线程出现停顿 时通过 jstack 来查看各个线程的调用堆栈，就可以获知没有响应的线程到底在后台做些什么事情，或者等待着什么资源。

* 命令格式:

  ```
  jstack [option] vmid
  ```

  | 选项 | 作用                                        |
  | ---- | ------------------------------------------- |
  | -F   | 当正常输出的请求不被响应时,强制输出线程堆栈 |
  | -l   | 除堆栈外,显示关于锁的附加信息               |
  | -m   | 如果调用到本地方法的话,可以显示C/C++的堆栈  |

  
