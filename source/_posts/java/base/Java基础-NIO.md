---
title: Java基础-NIO
mermaid: true
date: 2021-07-01 17:09:01
cover: /img/cover/Java.jpg
tags:
- NIO
categories:
- Java
- Base
updated:
type:
comments:
descriptI/On:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [【对线面试官】 Java NIO](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247483854&idx=1&sn=aa450a03ac0d6e8cf12cf13d4719ede3&scene=21#wechat_redirect)

### IO读写的基本原理

* 为了避免用户进程直接操作内核,保证内核安全,操作系统将内存(虚拟内存)划分为两个部分:
  * **内核空间(Kernel-Space)**
  * **用户空间(User-Space)**
* 在Linux系统中,内核模块运行在内核空间,对应的进程处于**内核态**;用户程序运行在用户空间,对应的进程处于**用户态**
* 用户进程不能访问内核空间中的数据,也不能直接调用内核函数,因此需要将进程切换到内核状态才能进行系统调用.**用户态进程必须通过系统调用(System Call)向内核发出指令,完成调用系统资源之类的操作**
* 用户程序进行IO的读写依赖于底层的IO读写,基本上会用到底层的`read`和`write`两大系统调用.但`read/write`系统调用都不是直接在物理设备做读取/写入操作.
  * 即上层应用通过操作系统的`read`系统调用把数据从**内核缓冲区**复制到应用程序的**进程缓冲区**,通过操作系统的`write`系统调用把数据从应用程序的进程缓冲区复制到操作系统的内核缓冲区.
  * **内核缓冲区<==>进程缓冲区**

### I/O模型 

* 在Unix下I/O模型分别有
  * **同步阻塞I/O**
  * **同步非阻塞I/O**
  * **I/O多路复用**
  * **信号驱动**
  * **异步I/O**

#### 同步阻塞I/O

* 阻塞I/O指的是内核IO操作彻底完成后才返回到用户空间执行用户程序的操作指令.
  * 阻塞指的是用户程序(发起IO请求的进程或者线程)的执行状态.
  * 传统的IO模型都是阻塞IO模型,并且在Java中默认创建的`socket`都属于阻塞IO模型;
* 同步和异步
  * 同步IO是值用户空间(进程或者线程)是主动发起请求的一方,系统内核是杯中接收方;
  * 异步则反过来,系统内核是主动发起IO请求的一方,用户空间是被动接收方.
* 同步阻塞IO(Blocking IO)指的是用户空间(或者线程)主动发起,需要等待内核IO操作彻底完成后才返回到用户空间的IO操作.在IO操作过程中,发起IO请求的用户进程(或线程)处于阻塞状态.

#### 同步非阻塞IO

* 非阻塞IO(Non-Blocking IO,NIO)指的是用户空间的程序不需要等待IO操作彻底完成,可以立即返回用户空进去执行后续的指令,即发起IO请求的用户进程(或线程)处于非阻塞状态,与此同时,内核会立即返回给用户一个IO状态值.

* 在Java中,非阻塞IO的socket被设置为`NONBLOCK`模式

  >  同步非阻塞IO也可以称为NIO,但是它不是Java编程中的NIO.Java编程中的NIO(New IO)类库组件归属的不是基础IO模型中的NIO模型,而是IO多路复用模型.

#### **I/O多路复用**

* Linux对文件的操作实际上就是通过**文件描述符(fd)**;
* I/O复用模型指的是: 通过一个进程(或者线程)监听多个文件描述符,一旦某个文件描述准备就绪(一般是内核缓冲区可读/可写),内核就能够将文件描述符的就绪状态返回给用户进程(或线程),用户空间可以根据文件描述符的就绪状态进行相应的IO系统调用.
* 这种以通知的方式,优势并不是对于单个连接能处理得更快,而是在于它能处理更多的连接;
* IO多路复用(IO Multiplexing)属于一种经典的Reactor模式实现,有时也称为异步阻塞IO,Java中的Selector属于这种模型.
* 在Linux下I/O复用模型的函数有`select/poll`和`epoll`
  * `select`函数
  * `select`函数它支持最大的连接数是1024或2048,因为在`select`函数下要传入`fd_set`参数,这个`fd_set`的大小要么1024或2048(其实就是看操作系统的位数)
  * `fd_set`就是`bitmap`的数据结构,可以简单理解为只要位为0,那就说明还没有到缓冲区,只要位为1,那就说明数据已经到缓冲区;
  * 而`select`函数做的就是每次将`fd_set`遍历,判断标志位有没有发现变化,如果有变化则通知程序做中断处理;
  * `epoll`是在Linux2.6内核正式提出,完善了`select`的一些缺点;
    * 它定义`epoll_event`结构体来处理,不存在最大连接数的限制;
    * 并且它不想`select`函数每次把所有的文件描述符(fd),都遍历,简单理解就是`epoll`把就绪的文件描述符(fd)专门维护了一块空间,每次从就绪列表里边拿就好了,不再进行对所有文件描述符(fd)进行遍历;


#### 异步IO

* 异步IO(Asynchronous IO,AIO)指的是用户空间的线程变成被动接收者,而内核空间称为主动调用者.在异步IO模型中,当用户线程收到通知时,数据已经被内核读取完毕并放在用户缓冲区内,内核在IO完成后通知用户现场直接使用接口.
* 异步IO类似Java中典型的回调模式.用户进程(或线程)向内核空间注册了各种IO事件的回调函数,由内核去主动调用.

#### 零拷贝

* 以读操作为例,假设用户程序发起一次读请求,
  * 其实会调用`read`相关的**系统函数**,然后会从**用户态**切换到**内核态**,随后CPU会告诉DMA会磁盘把数据拷贝到内核空间;
  * 等到**内核缓冲区**真正有数据之后,CPU会把**内核缓冲区**拷贝到**用户缓冲区**,最终用户程序才能获取到数据;
* 为了保证内核安全,操作系统将虚拟空间划分为**用户空间**和**内核空间**,所以在读取系统数据的时候会有状态切换,因为应用程序不能直接去读取硬盘的数据.从上面的描述可知读写需要依赖**内核缓冲区**.一次读操作会让DMA将磁盘数据拷贝到内核缓冲区,CPU将内核缓冲区数据拷贝到用户缓冲区;
* 所谓的零拷贝就是将**CPU将内核缓冲区数据拷贝到用户缓冲区**,这次CPU拷贝省去,来提高效率和性能;
* 常见的零拷贝技术有**mmap(内核缓冲区与用户缓冲区的共享)**,**sendfile(系统底层函数支持)**

### Java NIO

#### NIO和传统I/O的区别

* 传统I/O是一次一个字节的处理数据,NIO是以块(缓冲区)的形式处理数据.
* 最主要的是,NIO可以实现非阻塞,而传统I/O只能阻塞;
* I/O的实际场景是文件I/O和网络I/O,NIO在网络I/O场景下提升就是尤其明显;

#### NIO的组成部分(三个部分)

* **Buffer(缓冲区)**: Buffer是存储数据的地方;
* **Channel(管道)**: Channel是运输数据的载体;
* **Selector(选择器)**: Selector用于检查多个Channel的状态变更情况;

#### NIO示例

* 服务端

  ```java
  public class NoBlockServer {
  
      public static void main(String[] args) throws I/OExceptI/On {
  
          // 1.获取通道
          ServerSocketChannel server = ServerSocketChannel.open();
  
          // 2.切换成非阻塞模式
          server.configureBlocking(false);
  
          // 3. 绑定连接
          server.bind(new InetSocketAddress(6666));
  
          // 4. 获取选择器
          Selector selector = Selector.open();
  
          // 4.1将通道注册到选择器上，指定接收“监听通道”事件
          server.register(selector, SelectionKey.OP_ACCEPT);
  
          // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
          while (selector.select() > 0) {
              // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
              Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
  
              // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
              while (iterator.hasNext()) {
  
                  SelectionKey SelectionKey = iterator.next();
  
                  // 接收事件就绪
                  if (SelectionKey.isAcceptable()) {
  
                      // 8. 获取客户端的链接
                      SocketChannel client = server.accept();
  
                      // 8.1 切换成非阻塞状态
                      client.configureBlocking(false);
  
                      // 8.2 注册到选择器上-->拿到客户端的连接为了读取通道的数据(监听读就绪事件)
                      client.register(selector, SelectionKey.OP_READ);
  
                  } else if (SelectionKey.isReadable()) { // 读事件就绪
  
                      // 9. 获取当前选择器读就绪状态的通道
                      SocketChannel client = (SocketChannel) SelectionKey.channel();
  
                      // 9.1读取数据
                      ByteBuffer buffer = ByteBuffer.allocate(1024);
  
                      // 9.2得到文件通道，将客户端传递过来的图片写到本地项目下(写模式、没有则创建)
                      FileChannel outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOptI/On.WRITE, StandardOpenOptI/On.CREATE);
  
                      while (client.read(buffer) > 0) {
                          // 在读之前都要切换成读模式
                          buffer.flip();
  
                          outChannel.write(buffer);
  
                          // 读完切换成写模式，能让管道继续读取文件的数据
                          buffer.clear();
                      }
                  }
                  // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                  iterator.remove();
              }
          }
  
      }
  }
  ```

* 客户端

  ```java
  public class NoBlockClient {
  
      public static void main(String[] args) throws I/OExceptI/On {
  
          // 1. 获取通道
          SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 6666));
  
          // 1.1切换成非阻塞模式
          socketChannel.configureBlocking(false);
  
          // 1.2获取选择器
          Selector selector = Selector.open();
  
          // 1.3将通道注册到选择器中，获取服务端返回的数据
          socketChannel.register(selector, SelectionKey.OP_READ);
  
          // 2. 发送一张图片给服务端吧
          FileChannel fileChannel = FileChannel.open(Paths.get("X:\\Users\\ozc\\Desktop\\面试造火箭\\1.png"), StandardOpenOptI/On.READ);
  
          // 3.要使用NI/O，有了Channel，就必然要有Buffer，Buffer是与数据打交道的呢
          ByteBuffer buffer = ByteBuffer.allocate(1024);
  
          // 4.读取本地文件(图片)，发送到服务器
          while (fileChannel.read(buffer) != -1) {
  
              // 在读之前都要切换成读模式
              buffer.flip();
  
              socketChannel.write(buffer);
  
              // 读完切换成写模式，能让管道继续读取文件的数据
              buffer.clear();
          }
  
  
          // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
          while (selector.select() > 0) {
              // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
              Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
  
              // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
              while (iterator.hasNext()) {
  
                  SelectionKey SelectionKey = iterator.next();
  
                  // 8. 读事件就绪
                  if (SelectionKey.isReadable()) {
  
                      // 8.1得到对应的通道
                      SocketChannel channel = (SocketChannel) SelectionKey.channel();
  
                      ByteBuffer responseBuffer = ByteBuffer.allocate(1024);
  
                      // 9. 知道服务端要返回响应的数据给客户端，客户端在这里接收
                      int readBytes = channel.read(responseBuffer);
  
                      if (readBytes > 0) {
                          // 切换读模式
                          responseBuffer.flip();
                          System.out.println(new String(responseBuffer.array(), 0, readBytes));
                      }
                  }
  
                  // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                  iterator.remove();
              }
          }
      }
  
  }
  ```
