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

#### `InetAdress`类

* `InetAdress`类对IP地址建模.

  ```java
  	final InetAddress inetAddress = socket.getInetAddress();
  ```

  
