---
title: Redis-单线程模型
date: 2021-07-02 21:41:24
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags:
- 单线程模型
categories:
- Redis
updated: 2021-07-13 22:41:24
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

* [了解redis的单线程模型工作原理？一篇文章就够了](https://blog.csdn.net/qq_38601777/article/details/91325622)
* [REDIS 单线程模型介绍](https://www.cnblogs.com/yrjns/p/12517784.html)
* [redis单线程模型](https://www.cnblogs.com/ryjJava/p/14268079.html)
* [3w字深度好文|Redis面试全攻略，读完这个就可以和面试官大战几个回合了](https://mp.weixin.qq.com/s?__biz=Mzg2NzA4MTkxNQ==&mid=2247487073&idx=2&sn=28f48fc6de98b2a5c8382ad8e234ad5b&chksm=ce4045b5f937cca3450268ee8ad769a6818bbe97e0328a1dd065773a7a07a60eab9646a4c1fd&scene=126&sessionid=1583410944#rd)
* [极客时间专栏: Redis核心技术与实战](https://time.geekbang.org/column/intro/100056701)

#### Redis单线程

* Redis是单线程，主要是指**Redis的网络IO和键值对读写是由一个线程来完成的，这也是Redis对外提供键值存储服务的主要流程**。但Redis的其他功能，比如持久化、异步删除、集群数据同步等，其实是由额外的线程执行的。

#### Redis 为什么使用单线程?

* **从性能角度来看**
  * 多线程的开销
    * **多线程编程模式面临的共享资源的并发访问控制问题**。
    * 并发访问控制一直是多线程开发中的一个难点问题，如果没有精细的设计，比如说，只是简单地采用一个粗粒度互斥锁，就会出现不理想的结果：即使增加了线程，大部分线程也在等待获取访问共享资源的互斥锁，并行变串行，系统吞吐率并没有随着线程的增加而增加。
    * 而且，采用**多线程开发一般会引入同步原语来保护共享资源的并发访问**，这也会降低系统代码的易调试性和可维护性。为了避免这些问题，Redis直接采用了单线程模式。
  * **单线程避免了线程切换和竞态产生的消耗。**

    * 单线程能带来几个好处：
      - 第一，单线程可以简化数据结构和算法的实现。并发数据结构实现不但困难而且开发测试比较麻烦;

      - 第二，单线程避免了线程切换和竞态产生的消耗，对于服务端开发来说，锁和线程切换通常是性能杀手;

    * 单线程的问题：对于每个命令的执行时间是有要求的。如果某个命令执行过长，会造成其他命令的阻塞，所以Redis 适用于那些需要快速执行的场景。
* **从内部结构设计角度来看**
  * Redis是基于Reactor模式开发了自己的网络事件处理器,这个处理器被称为**文件处理器(File Event Handler)**.而这个文件事件处理器是单线程的,所以才叫Redis的单线程模型,这也决定了Redis的单线程;

#### 单线程Redis为什么那么快?

* 一方面,Redis的大部分操作是**在内存中完成**,再加上它采用了**高效的数据结构**.
* 另一方面,Redis采用了**多路复用机制**,使其在网络IO操作中能并发处理大量的客户端请求,实现高吞吐率.

#### 基本IO模型与阻塞点

* 以Get请求为例，SimpleKV为了处理一个Get请求，需要监听客户端请求（`bind/listen`），和客户端建立连接（`accept`），从Socket中读取请求（`recv`），解析客户端发送请求（`parse`），根据请求类型读取键值数据（g`et`），最后给客户端返回结果，即向Socket中写回数据（`send`）。

* 下图显示了这一过程，其中，bind/listen、accept、recv、parse和send属于网络IO处理，而get属于键值数据操作。既然Redis是单线程，那么，最基本的一种实现是在一个线程中依次执行上面说的这些操作。

  <img src="https://www.holelin.cn/img/redis/%E5%9F%BA%E6%9C%ACIO%E6%A8%A1%E5%9E%8B%E4%B8%8E%E9%98%BB%E5%A1%9E%E7%82%B9.jpg" alt="img" style="zoom:67%;" />

  * 在这里的网络IO操作中，有潜在的阻塞点，分别是**`accept()`**和**`recv()`**。
    * 当Redis监听到一个客户端有连接请求，但一直未能成功建立起连接时，会阻塞在accept()函数这里，导致其他客户端无法和Redis建立连接。	
    * 类似的，当Redis通过`recv()`从一个客户端读取数据时，如果数据一直没有到达，Redis也会一直阻塞在`recv()`。
  * 这就导致Redis整个线程阻塞，无法处理其他客户端请求，效率很低。不过，幸运的是，**Socket网络模型本身支持非阻塞模式**

##### 非阻塞模式

* Socket网络模型的非阻塞模式设置，主要体现在三个关键的函数调用上，如果想要使用socket非阻塞模式，就必须要了解这三个函数的调用返回类型和设置模式;

* 在Socket模型中，不同操作调用后会返回不同的套接字类型。`socket()`方法会返回主动套接字，然后调用`listen()`方法，将主动套接字转化为监听套接字，此时，可以监听来自客户端的连接请求。最后，调用`accept()`方法接收到达的客户端连接，并返回已连接套接字。

  ![img](https://www.holelin.cn/img/redis/%E9%9D%9E%E9%98%BB%E5%A1%9E%E6%A8%A1%E5%BC%8F%E4%B8%89%E4%B8%AA%E5%85%B3%E9%94%AE%E5%87%BD%E6%95%B0.jpg)

  * 针对监听套接字，我们可以设置非阻塞模式：当Redis调用`accept()`但一直未有连接请求到达时，Redis线程可以返回处理其他操作，而不用一直等待。但是，你要注意的是，调用`accept()`时，已经存在监听套接字了。
  * 虽然Redis线程可以不用继续等待，但是总得有机制继续在监听套接字上等待后续连接请求，并在有请求时通知Redis。

##### 基于多路复用的高性能I/O模型

* Linux中的IO多路复用机制是指一个线程处理多个IO流，就是我们经常听到的`select/epoll`机制。

* 简单来说，在Redis只运行单线程的情况下，**该机制允许内核中，同时存在多个监听套接字和已连接套接字**。内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给Redis线程处理，这就实现了一个Redis线程处理多个IO流的效果。

* 下图就是基于多路复用的Redis IO模型。图中的多个`FD`就是刚才所说的多个套接字。Redis网络框架调用epoll机制，让内核监听这些套接字。此时，Redis线程不会阻塞在某一个特定的监听或已连接套接字上，也就是说，不会阻塞在某一个特定的客户端请求处理上。正因为此，Redis可以同时和多个客户端连接并处理请求，从而提升并发性。

  ![img](https://www.holelin.cn/img/redis/%E5%9F%BA%E4%BA%8E%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E7%9A%84Redis%20IO%E6%A8%A1%E5%9E%8B.jpg)

* 为了在请求到达时能通知到Redis线程，`select/epoll`提供了**基于事件的回调机制**，即**针对不同事件的发生，调用相应的处理函数**。

  * `select/epoll`一旦监测到FD上有请求到达时，就会触发相应的事件。
  * 这些事件会被放进一个事件队列，Redis单线程对该事件队列不断进行处理。这样一来，Redis无需一直轮询是否有请求实际发生，这就可以避免造成CPU资源浪费。同时，Redis在对事件队列中的事件进行处理时，会调用相应的处理函数，这就实现了基于事件的回调。因为Redis一直在对事件队列进行处理，所以能及时响应客户端请求，提升Redis的响应性能。

  > 为了方便你理解，我再以连接请求和读数据请求为例，具体解释一下。
  >
  > * 这两个请求分别对应Accept事件和Read事件，Redis分别对这两个事件注册accept和get回调函数。当Linux内核监听到有连接请求或读数据请求时，就会触发Accept事件和Read事件，此时，内核就会回调Redis相应的accept和get函数进行处理。
  >
  > * 这就像病人去医院瞧病。在医生实际诊断前，每个病人（等同于请求）都需要先分诊、测体温、登记等。如果这些工作都由医生来完成，医生的工作效率就会很低。所以，医院都设置了分诊台，分诊台会一直处理这些诊断前的工作（类似于Linux内核监听请求），然后再转交给医生做实际诊断。这样即使一个医生（相当于Redis单线程），效率也能提升。

* 文件描述符(FD)：内核（Kernel）利用文件描述符（File Descriptor）来访问文件。文件描述符是非负整数。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。

![img](https://www.holelin.cn/img/linux/IO%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8.png)

* 多路 I/O 复用模型是利用`select`、`poll`、`epoll`可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有I/O事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（`epoll`是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络IO的时间消耗），且Redis在内存中操作数据的速度非常快（内存的操作不会成为这里的性能瓶颈），主要以上两点造就了Redis具有很高的吞吐量。

* `IO`复用只需要阻塞在`select`，`poll`或者`epoll`，可以同时处理和管理多个连接。缺点是当`select`、`poll`或者`epoll` 管理的连接数过少时，这种模型将退化成阻塞`IO` 模型。并且还多了一次系统调用：一次`select`、`poll`或者`epoll` 一次`recvfrom`。

* `select`、`poll`、`epoll` 区别：

  |          | 最大连接数                                                   | FD剧增后带来的IO效率问题                                     | 消息传递方式                                                 |
  | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | `select` | 单个进程所能打开的最大连接数有FD_SETSIZE宏定义，其大小是32个整数的大小（在32位的机器上，大小就是32\*32，同理64位机器上FD_SETSIZE为32\*64) | 因为每次调用时都会对连接进行线性遍历，所以随着FD的增加会造成遍历速度慢的“线性下降性能问题”。 | 内核需要将消息传递到用户空间，都需要内核拷贝动作             |
  | `poll`   | 基于链表来存储的，没有最大连接数的限制                       | 同上                                                         | 内核需要将消息传递到用户空间，都需要内核拷贝动作             |
  | `epoll`  | 接数有上限，但是很大，1G内存的机器上可以打开10万左右的连接   | 因为`epoll`内核中实现是根据每个`fd`上的`callback`函数来实现的，只有活跃的socket才会主动调用callback，所以在活跃socket较少的情况下，使用`epoll`没有前面两者的线性下降的性能问题，但是所有socket都很活跃的情况下，可能会有性能问题。 | 利用`mmap()`文件映射内存加速与内核空间的消息传递；即`epoll`使用`mmap`减少复制开销 |

* **使用`epoll`模型**

  > `epoll`是Linux提供的系统实现，核心方法只有三个`epoll_create`、`epoll_ctl`、`epoll_wait`。`epoll`效率高，是因为基于红黑树、双向链表、事件回调机制。

  ![img](https://www.holelin.cn/img/linux/epoll%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

  * `epoll_create`

    ```
    epoll_create(int size)
    核心功能：
    1.创建一个epoll文件描述符
    2.创建eventpoll，其中包含红黑树cache和双向链表
    ```

    * 参数size并不是限制了`epoll`所能监听的文件描述符最大个数，只是对内核初始分配内部数据结构的一个建议。在Linux 2.6.8后，size 参数被忽略，但是必须传一个比 0 大的数。调用`epoll_create`后，会占用一个`fd`值。在Linux下可以查看`/proc/$$/fd/` 文件描述符。使用完，需要调用close关闭。

  * `epoll_ctl`

    ```
    int epoll_ctl(int epfd, int op, int fd, struct epollevent *event)；
    核心功能：
    1.对指定描述符fd执行op的绑定操作
    2.把fd写入红黑树，同时在内核注册回调函数
    ```

    * op操作类型，用三个宏`EPOLL_CTL_ADD`，`EPOLL_CTL_DEL`，`EPOLL_CTL_MOD`，来分别表示增删改对`fd`的监听。

  * `epoll_wait`

    ```
    int epoll_wait(int epfd, struct epollevent *events, int maxevents, int timeout);
    核心功能：
    1.获取epfd上的io事件
    ```

    * 参数`events`是就绪事件，用来得到想要获得的事件集合。`maxevents`表示的events有多大，`maxevents`的值必须大于0，参数`timeout`是超时时间。`epollwait`会阻塞，直到一个文件描述符触发了事件，或者被一个信号处理函数打断，或者timeout超时。返回值是需要处理的`fd`数量。

  * **使用`epoll`模型优点**

    * `epoll`创建的红黑树保存所有`fd`，没有大小限制，且增删查的复杂度`O(logN)`
    * 基于callback，利用系统内核触发感兴趣的事件
    * 就绪列表为双线链表时间复杂度O(1)
    * 应用获取到的`fd`都是真实发生IO的`fd`，与select 和 poll 需要不断轮询判断是否可用相比，能避免无用的内存拷贝

#### Redis的文件事件和时间事件

> Redis是事件驱动的服务器，主要的事件类型就是：**文件事件类型**和**时间事件类型**，其中时间事件是理解单线程逻辑模型的关键。

##### 时间事件

* Redis的时间事件分为两类:
  * **定时事件**: 任务在等待指定大小的等待时间之后就执行，执行完成就不再执行，只触发一次；
  * **周期事件**: 任务每隔一定时间就执行，执行完成之后等待下一次执行，会周期性的触发；
    * Redis中大部分是周期事件，周期事件主要是服务器定期对自身运行情况进行检测和调整，从而保证稳定性，这项工作主要是ServerCron函数来完成的，周期事件的内容主要包括:
      * 删除数据库的key
      * 触发RDB和AOF持久化
      * 主从同步
      * 集群化保活
      * 关闭清理死客户端链接
      * 统计更新服务器的内存、key数量等信息
    * 可见 Redis的周期性事件虽然主要处理辅助任务，但是对整个服务的稳定运行，起到至关重要的作用。

##### 时间事件的无序链表

* Redis的每个时间事件分为三个部分：
  * 事件ID 全局唯一 依次递增;
  * 触发时间戳 ms级精度
  * 事件处理函数 事件回调函数
* Redis的时间事件是存储在链表中的，并且是**按照ID存储**的，**新事件在头部旧事件在尾部，但是并不是按照即将被执行的顺序存储的**。
  * 也就是第一个元素50ms后执行，但是第三个可能30ms后执行，这样的话Redis每次从链表中获取最近要执行的事件时，都需要进行O(N)遍历，显然性能不是最好的，最好的情况肯定是类似于最小栈MinStack的思路，然而Antirez大佬却选择了无序链表的方式。

#### 单线程模式中事件调度和执行

* Redis服务器会轮流处理文件事件和时间事件，这两种事件的处理都是同步、有序、原子地执行的，服务器也不会终止正在执行的事件，也不会对事件进行抢占。

* **事件执行调度规则**

  * 文件事件是随机出现的，如果处理完成一次文件事件后，仍然没有其他文件事件到来，服务器将继续等待，在文件事件的不断执行中，时间会逐渐向最早的时间事件所设置的到达时间逼近并最终来到到达时间，这时服务器就可以开始处理到达的时间事件了。
  * 由于时间事件在文件事件之后执行，并且事件之间不会出现抢占，所以时间事件的实际处理时间一般会比设定的时间稍晚一些

* **事件执行和调度的伪码**

  ```c
  def aeProcessEvents()
    #获取当前最近的待执行的时间事件
    time_event = aeGetNearestTimer()
    #计算最近执行事件与当前时间的差值
    remain_gap_time = time_event.when - uinx_time_now()
    #判断时间事件是否已经到期 则重置 马上执行
    if remain_gap_time < 0:
      remain_gap_time = 0
    #阻塞等待文件事件 具体的阻塞等待时间由remain_gap_time决定
    #如果remain_gap_time为0 那么不阻塞立刻返回
    aeApiPoll(remain_gap_time)
    #处理所有文件事件
    ProcessAllFileEvent()
    #处理所有时间事件
    ProcessAllTimeEvent()
  ```

  * 可以看到Redis服务器是边阻塞边执行的，具体的阻塞事件由最近待执行时间事件的等待时间决定的，在阻塞该最小等待时间返回之后，开始处理事件任务，并且先执行文件事件、再执行时间事件，所有即使时间事件要即刻执行，也需要等待文件事件完成之后再执行时间事件，所以比预期的稍晚。

* **事件调度和执行流程**

  <img src="https://www.holelin.cn/img/redis/Redis%E4%BA%8B%E4%BB%B6%E8%B0%83%E5%BA%A6%E5%92%8C%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png" alt="img" style="zoom:67%;" />

#### Redis的单线程模型

> Redis单线程模型中最核心的就是**文件事件处理器**

<img src="https://www.holelin.cn/img/redis/Redis%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.png" alt="img" style="zoom:67%;" />

##### 文件事件处理器

* 文件事件处理器结构包含4个部分:**Socket**,**I/O多路复用程序**,**文件事件分派器(`Dispather`)**,**事件处理器(`Handler`)**;
* `I/O`多路复用程序会同时监听多个`socket`，当被监听的`socket`准备好执行`accept`、`read`、`write`、`close`等操作时，与这些操作相对应的文件事件就会产生。`IO`多路复用程序会把所有产生事件的`socket`压入一个队列中，然后有序地每次仅一个`socket`的方式传送给文件事件分派器，文件事件分派器接收到`socket`之后会根据`socket`产生的事件类型调用对应的事件处理器进行处理。
* 事件处理器分为以下几种:
  * **连接应答处理器**: 用于处理客户端的连接请求;
  * **命令请求处理器**: 用于执行客户端传递过来的命令,比如常见的`set`,`lpush`等;
  * **命令回复处理器**: 用于返回客户端命令的执行结果,比如`set`,`get`等命令的结果;
* 事件的种类: 
  * **AE_READABLE**: 与两个事件处理器结合使用.
    * 当客户端连接服务器端时,服务端会将**连接应答处理器**与`Socket`的**AE_READABLE**事件关联起来;
    * 当客户端向服务端发送命令的时候,服务器端将**命令请求处理器**与**AE_READABLE**事件关联起来;
  * **AE_WRITABLE **: 当服务端有数据需要回传给客户端时,服务端将**命令回复处理器**与`Socket`的**AE_WRITABLE **事件关联起来;

##### Redis的客户端与服务器端的交互过程

![img](https://www.holelin.cn/img/redis/Redis%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E7%9A%84%E4%BA%A4%E4%BA%92%E8%BF%87%E7%A8%8B.jpg)

* 在Redis启动及初始化的时候,Redis会(预先)将**连接应答处理器**跟**AE_READABLE事件**关联起来,接着如果一个客户端向Redis发起连接,此时就会产生一个**AE_READABLE事件**,然后由**连接应答处理器**来处理跟客户端建立连接,创建客户端对应的socket,同时将这个Socket的**AE_READABLE事件**跟**命令请求处理器**关联起来;
* 当客户端向Redis发起请求的时候(不管是读请求还是写请求,都一样),首先就会在之前创建的客户端对应的Socket上产生一个**AE_READABLE事件**,然后**I/O多路复用程序**会监听到在之前创建的客户端对应的Socket上产生了一个**AE_READABLE事件**,接着把这个Socket放入一个队列中排队,然后由**文件事件分派器**从队列中获取Socket交给对应的**命令请求处理器**来处理(因为之前在Redis启动并进行初始化的时候就已经预先将**AE_READABLE**事件跟**命令请求处理器**关联起来了).之后**命令请求处理器**就会从之前创建的客户端对应的Socket中读取请求相关的数据,然后在自己的内存中进行执行和处理;
*  当客户端请求处理完成,Redis这边也准备好了给客户端的响应数据之后,就会(预先)将Socket的**AE_WRITABLE**事件跟**命令回复处理器**关联起来,当客户端这边准备好读取响应数据时,就会在之前创建的客户端对应的Socket上产生一个**AE_WRITABLE事件**,然后**IO多路复用程序**会监听到在之前创建的客户端对应的Socket上产生了一个**AE_WRITABLE事件**,接着把这个Socket放入一个队列中排队,然后由**文件事件分派器**从队列中获取Socket交给对应的**命令回复处理器**来处理(因为之前在Redis这边准备好给客户端的响应数据之后就已经预先将**AE_WRITABLE事件**跟**命令回复处理器**关联起来了),之后**命令回复处理器**就会向之前创建的客户端对应的Socket输出/写入准备好的响应数据,最终返回给客户端,供客户端来读取;

#### 为什么Redis使用单线程模型还能保证高性能？

* **纯内存访问**: Redis将所有数据放在内存中,内存的响应时长大约为100纳秒,这是Redis的QPS过万的重要基础;
* **采用高效的数据结构**,如哈希表和跳表等;
* **使用非阻塞式I/O**
  * 什么是阻塞式I/O
    * 当我们调用 Scoket 的读写方法，默认它们是阻塞的。
    * `read()` 方法要传递进去一个参数 n，表示读取这么多字节后再返回，如果没有读够 n 字节线程就会阻塞，直到新的数据到来或者连接关闭了,`read()` 方法才可以返回,线程才能继续处理。
    * `write()` 方法会首先把数据写到系统内核为Socket 分配的写缓冲区中，当写缓存区满溢，即写缓存区中的数据还没有写入到磁盘，就有新的数据要写道写缓存区时,`write()` 方法就会阻塞，直到写缓存区中有空闲空间。
  * 什么是非阻塞式 I/O
    * 非阻塞 I/O 在 Socket 对象上提供了一个选项`Non_Blocking` ，当这个选项打开时，读写方法不会阻塞，而是能读多少读多少，能写多少写多少。
    * 能读多少取决于内核为 Socket 分配的读缓冲区的大小，能写多少取决于内核为 Socket 分配的写缓冲区的剩余空间大小。读方法和写方法都会通过返回值来告知程序实际读写了多少字节数据。
    * 有了非阻塞 IO 意味着线程在读写 IO 时可以不必再阻塞了，读写可以瞬间完成然后线程可以继续干别的事了。
* **使用I/O多路复用**


#### Redis单线程处理IO请求性能瓶颈

* **主要两方面:**
  * 任意一个请求在server中一旦发生耗时，都会影响整个server的性能，也就是说后面的请求都要等前面这个耗时请求处理完成，自己才能被处理到。耗时的操作包括以下几种：
    * **操作bigkey**：写入一个bigkey在分配内存时需要消耗更多的时间，同样，删除bigkey释放内存同样会产生耗时；
    * **使用复杂度过高的命令**：例如`SORT`/`SUNION`/`ZUNIONSTORE`，或者O(N)命令，但是N很大，例如`lrange key 0 -1`一次查询全量数据；
    * **大量key集中过期**：Redis的过期机制也是在主线程中执行的，大量key集中过期会导致处理一个请求时，耗时都在删除过期key，耗时变长；
    * **淘汰策略**：淘汰策略也是在主线程执行的，当内存超过Redis内存上限后，每次写入都需要淘汰一些key，也会造成耗时变长；
    * **AOF刷盘开启always机制**：每次写入都需要把这个操作刷到磁盘，写磁盘的速度远比写内存慢，会拖慢Redis的性能；
    * **主从全量同步生成RDB**：虽然采用fork子进程生成数据快照，但fork这一瞬间也是会阻塞整个线程的，实例越大，阻塞时间越久；
  * 并发量非常大时，单线程读写客户端IO数据存在性能瓶颈，虽然采用IO多路复用机制，但是读写客户端数据依旧是同步IO，只能单线程依次读取客户端的数据，无法利用到CPU多核。
* **解决方法:**
  * 针对问题1，一方面需要业务人员去规避，一方面Redis在4.0推出了lazy-free机制，把bigkey释放内存的耗时操作放在了异步线程中执行，降低对主线程的影响。
  * 针对问题2，Redis在6.0推出了多线程，可以在高并发场景下利用CPU多核多线程读写客户端数据，进一步提升server性能，当然，只是针对客户端的读写是并行的，每个命令的真正操作依旧是单线程的;

