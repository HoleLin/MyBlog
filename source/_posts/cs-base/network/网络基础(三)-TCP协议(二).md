---
title: 网络基础-TCP协议(二)
date: 2021-10-22 11:45:21
index_img: /img/cover/Network.jpg
cover: /img/cover/Network.jpg
tags:
- TCP
categories:
- 网络
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

* 编程必备基础 大话HTTP协议[慕课]
* [趣谈网络协议](https://time.geekbang.org/column/intro/85)
* WireShark数据包分析实战(第三版)
* [TCP-RFC793](https://www.ietf.org/rfc/rfc793.txt)
* TCP/IP详解 卷1: 协议
* 图解网络-小林coding
* [UDP-RFC768]

#### `TCP`重传机制

* TCP实现可靠传输的方式之一是**通过序列号与确认应答**.
  * 在TCP中,当发送端的数据到达接收主机时,接收端主机会返回一个确认应答消息,表示已收到消息.
* 常见的重传机制有:
  * 超时重传
  * 快速重传
  * SACK
  * D-SACK

##### 超时重传

* 在发送数据时,设定一个定时器,当超过指定的时间后,没有收到对方的ACK确认应答报文,就会重发该数据.
* TCP会在以下两种情况发生超时重传:
  * 数据包丢失: 数据包未成功发送
  * 确认应答丢失: 确认应答未成功发送
* `RTT(Round-Trip 往返时延)`: 数据从网络一端传送到另一端所需的时间,也就是包的往返时间.
* 超时重传时间是以`RTO(Retransmission Timeout 超时重传时间)`表示
* 在重传的情况下,超时时间RTO较长或较短会出现不同的情况
  * 当超时时间RTO较大时,重发就慢,丢了很久才重发,没效率,性能差;
  * 当超时时间RTO较小时,会导致可能并没有丢就重发,于是重发的就快,会增加网络拥塞,导致更多的超时,更多的超时导致更多的重发.
* 超时重传RTO的值应该略大于报文往返RTT的值.

#### `TCP`滑动窗口

#### `TCP`流量控制

#### `TCP`拥塞控制
