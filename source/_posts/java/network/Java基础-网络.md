---
title: Java基础-网络
date: 2022-02-23 11:26:28
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- socket
categories:
- Java
- Socket
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

* Java语言程序设计-进阶版(第8版)
* Java核心技术 卷2 高级特性

### 引言

* Java支持基于流的通信(`stream-based communication`)和基于包通信(`packet-based  communication`).

* 基于流的通信使用传输控制协议TCP进行数据传输,而基于包的通信使用用户数据协议UDP.

* Java API提供用于创建套接字的类来便于程序的网络通信.套接字(scoket)是两台主机之间逻辑连接的端点,可以用来发送和接收数据.

* 网络程序设计通常涉及一个服务器和一个或多个客户端,客户端向服务器发送请求,而服务器响应请求.客户端尝试建立与服务器的连接开始,服务器可能接受或拒绝这个连接.一旦建立连接,客户端和服务器就可以通过套接字进行通信.

* 当客户端尝试连接服务器时,服务器必须正在运行.服务器等待来自客户端的连接请求.

  ```java
  // 服务端
  				// 创建服务器套接字
          final ServerSocket serverSocket = new ServerSocket(8000);
          // 创建连接到客户的套接字
          final Socket accept = serverSocket.accept();
  // 客户端
  				// 创建连接到服务器
  				// serverHost 支持IP/域名格式
          final Socket socket = new Socket("serverHost", 8000);
  
  ```

### 套接字超时

* 从套接字读取信息时,在有数据可访问之前,读操作将会被阻塞.如果此时主机不可达,那么应用将要等待很长的时间,并且因为底层操作系统的限制而最终会导致超时.

* 对于不同的应用,应该确定合理的超时值.然后调用`setSoTimeout`方法设置这个超时值(单位:毫秒)

  ```java
          final Socket socket = new Socket("serverHost", port);
  				socket.setSoTimeout(10000);
  				// 继而设置了超时时间后 需要捕获SocketTimeoutException异常
  				try{
            	InputStream in = socket.getInputStream();
            	// ...
          } catch(InterruptedIOException exception){
            // react to timeout
          }
  
  ```

* 还有个超时问题是必须解决的.`Socket socket = new Socket("serverHost", port)`会一致无限期地阻塞下去,直到建立了到达主机的初始连接为止.

  * 可以通过先构建一个无连接的套接字,然后再使用一个超时来进行连接的方法解决这个问题

  ```java
          final Socket socket = new Socket(host, port);
  				socket.connect(new InetSocketAddress(host,port),timeout);
  ```

### `InetAdress`类

* `InetAdress`类对IP地址建模.

  ```java
  	final InetAddress inetAddress = socket.getInetAddress();
  	// 获取本地地址
  	inetAddress.getLocalHost();
  ```
  

### 半关闭

* 半关闭(half-close)提供这样的能力: 套接字连接的一端可以终止其输出,同时任旧可以接收来自另一端的数据.

  * 例如我们在想服务器传输数据,但是并不知道要传输多少数据.在向文件写数据时,我们只需要在数据写入后关闭文件即可.但是,如果关闭一个套接字,那么与服务器的连接立刻断开,因而也就读取服务器的响应了.
  * 使用半关闭的方法就可以解决上诉问题.可以通过关闭一个套接字的输出流来表示发送给服务器的请求数据已经结束,但必须保持输入流处于打开状态.

  ```java
          final Socket socket = new Socket(host, port);
          Scanner in = new Scanner(socket.getInputStream());
          final PrintWriter writer = new PrintWriter(socket.getOutputStream());
          writer.println("");
          writer.flush();
          // half-closed
          socket.shutdownOutput();
          while (in.hasNextLine()){
              final String next = in.next();
          }
          socket.close();
  ```

### 可中断套接字

* 当连接到一个套接字,当前线程将会被阻塞直到建立连接或产生超时为止.同样地,当套接字读写数据时,当前线程也会被阻塞直到操作成功或产生超时为止.

* 在交互式的应用中,也许会考虑为用户提供一个选项,用以取消那些看似不会产生结果的连接.但是,当线程因套接字无法响应而发送阻塞时,则无法通过调用`interrupt`来解除阻塞.

* 为了中断套接字,可以使用`java.nio`包提供了一个特性`SocketChanel`类.

  ```java
          final SocketChannel channel = SocketChannel.open(new InetSocketAddress(host, port));
  ```

  * 通道(Channel)并没有与之相关联的流.实际上,它所拥有的的`read`和`write`方法都是通过使用`Buffer`对象来实现的`ReadableByteChannel`接口和`WritableByteChannel`接口都声明了这两个方法.

    * 若不想处理缓冲区,可以使用`Scanner`类从`SocketChannel`中读取信息,因为`Scanner`有一个带`ReadableByteChannel`参数的构造器.

      ```java
      	Scanner in = new Scanner(channel);
      ```

    * 通过调用静态方法`Channels.newOutputStream`可以将通道转换成输出流.

      ```java
      	OutputStream outStream = Channels.newOutputStream(channel);
      ```

  * 假设现场正在执行打开,读取或写入操作,此时如果线程发生中断,那么这些操作将不会陷入阻塞,而是以抛出异常的方式结束.
