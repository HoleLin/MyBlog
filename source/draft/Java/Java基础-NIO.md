---
title: Java基础-NIO
mermaid: true
date: 2021-07-01 17:09:01
cover: /img/cover/Java.jpg
tags:
- NIO
categories:
- Java
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

* [【对线面试官】 Java NI/O](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247483854&idx=1&sn=aa450a03ac0d6e8cf12cf13d4719ede3&scene=21#wechat_redirect)

#### NI/O和传统I/O的区别

* 传统I/O是一次一个字节的处理数据,NI/O是以块(缓冲区)的形式处理数据.
* 最主要的是,NI/O可以实现非阻塞,而传统I/O只能阻塞;
* I/O的实际场景是文件I/O和网络I/O,NI/O在网络I/O场景下提升就是尤其明显;

#### NI/O的组成部分(三个部分)

* **Buffer(缓冲区)**: Buffer是存储数据的地方;
* **Channel(管道)**: Channel是运输数据的载体;
* **Selector(选择器)**: Selector用于检查多个Channel的状态变更情况;

#### NI/O示例

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
          server.register(selector, SelectI/OnKey.OP_ACCEPT);
  
          // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
          while (selector.select() > 0) {
              // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
              Iterator<SelectI/OnKey> iterator = selector.selectedKeys().iterator();
  
              // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
              while (iterator.hasNext()) {
  
                  SelectI/OnKey selectI/OnKey = iterator.next();
  
                  // 接收事件就绪
                  if (selectI/OnKey.isAcceptable()) {
  
                      // 8. 获取客户端的链接
                      SocketChannel client = server.accept();
  
                      // 8.1 切换成非阻塞状态
                      client.configureBlocking(false);
  
                      // 8.2 注册到选择器上-->拿到客户端的连接为了读取通道的数据(监听读就绪事件)
                      client.register(selector, SelectI/OnKey.OP_READ);
  
                  } else if (selectI/OnKey.isReadable()) { // 读事件就绪
  
                      // 9. 获取当前选择器读就绪状态的通道
                      SocketChannel client = (SocketChannel) selectI/OnKey.channel();
  
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

  ```
  public class NoBlockClient {
  
      public static void main(String[] args) throws I/OExceptI/On {
  
          // 1. 获取通道
          SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 6666));
  
          // 1.1切换成非阻塞模式
          socketChannel.configureBlocking(false);
  
          // 1.2获取选择器
          Selector selector = Selector.open();
  
          // 1.3将通道注册到选择器中，获取服务端返回的数据
          socketChannel.register(selector, SelectI/OnKey.OP_READ);
  
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
              Iterator<SelectI/OnKey> iterator = selector.selectedKeys().iterator();
  
              // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
              while (iterator.hasNext()) {
  
                  SelectI/OnKey selectI/OnKey = iterator.next();
  
                  // 8. 读事件就绪
                  if (selectI/OnKey.isReadable()) {
  
                      // 8.1得到对应的通道
                      SocketChannel channel = (SocketChannel) selectI/OnKey.channel();
  
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

#### I/O模型 

* 在Unix下I/O模型分别有
  * **阻塞I/O**
  * **非阻塞I/O**
  * **I/O复用**
  * **信号驱动**
  * **异步I/O**

#### **I/O复用**

* Linux对文件的操作实际上就是通过**文件描述符(fd)**;
* I/O复用模型指的是: 通过一个进程监听多个文件描述符,一旦某个文件描述准备就绪,就会通知程序做相对应的处理;
* 这种以通知的方式,优势并不是对于单个连接能处理得更快,而是在于它能处理更多的连接;
* 在Linux下I/O复用模型的函数有`select/poll`和`epoll`
* `select`函数
  * `select`函数它支持最大的连接数是1024或2048,因为在`select`函数下要传入`fd_set`参数,这个`fd_set`的大小要么1024或2048(其实就是看操作系统的位数)
  * `fd_set`就是`bitmap`的数据结构,可以简单理解为只要位为0,那就说明还没有到缓冲区,只要位为1,那就说明数据已经到缓冲区;
  * 而`select`函数做的就是每次将`fd_set`遍历,判断标志位有没有发现变化,如果有变化则通知程序做中断处理;
* `epoll`是在Linux2.6内核正式提出,完善了`select`的一些缺点;
  * 它定义`epoll_event`结构体来处理,不存在最大连接数的限制;
  * 并且它不想`select`函数每次把所有的文件描述符(fd),都遍历,简单理解就是`epoll`把就绪的文件描述符(fd)专门维护了一块空间,每次从就绪列表里边拿就好了,不再进行对所有文件描述符(fd)进行遍历;

#### 零拷贝

* 以读操作为例,假设用户程序发起一次读请求,
  * 其实会调用`read`相关的**系统函数**,然后会从**用户态**切换到**内核态**,随后CPU会告诉DMA会磁盘把数据拷贝到内核空间;
  * 等到**内核缓冲区**真正有数据之后,CPU会把**内核缓冲区**拷贝到**用户缓冲区**,最终用户程序才能获取到数据;
* 为了保证内核安全,操作系统将虚拟空间划分为**用户空间**和**内核空间**,所以在读取系统数据的时候会有状态切换,因为应用程序不能直接去读取硬盘的数据.从上面的描述可知读写需要依赖**内核缓冲区**.一次读操作会让DMA将磁盘数据拷贝到内核缓冲区,CPU将内核缓冲区数据拷贝到用户缓冲区;
* 所谓的零拷贝就是将**CPU将内核缓冲区数据拷贝到用户缓冲区**,这次CPU拷贝省去,来提高效率和性能;
* 常见的零拷贝技术有**mmap(内核缓冲区与用户缓冲区的共享)**,**sendfile(系统底层函数支持)**
