---
title: 网络基础(四)-UDP协议
date: 2022-03-26 15:58:12
tags:
- 传输层
categories:
- 网络
- UDP
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
* [UDP-RFC768](https://www.ietf.org/rfc/rfc0768)

### `UDP`

##### `UDP`头部格式

```
  							  0      7 8     15 16    23 24    31  
                 +--------+--------+--------+--------+ 
                 |     Source      |   Destination   | 
                 |      Port       |      Port       | 
                 +--------+--------+--------+--------+ 
                 |                 |                 | 
                 |     Length      |    Checksum     | 
                 +--------+--------+--------+--------+ 
                 |                                     
                 |          data octets ...            
                 +---------------- ...                 

                      User Datagram Header Format
```

* 源端口号(16位)和目标端口(16位): 主要是告诉UDP协议应该把报文发给哪个进程.
* 包长度: 该字段保存了UDP首部的长度跟数据的长度之和.
* 校验和: 校验和是为了提供可靠的UDP首部和数据而设计.
