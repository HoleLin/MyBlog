---
title: 网络基础(五)-IP协议
date: 2022-03-26 15:58:17
index_img: /img/cover/Network.jpg
cover: /img/cover/Network.jpg
tags:
- 网络层
categories:
- 网络
- IP
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
  enable: true
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

### IP

* 主要目的是解决寻址和路由问题,以及如何在两点之间传送数据包.
  * IP 协议使用“**IP 地址**”的概念来定位互联网上的每一台计算机。
* IPV4地址是用四个**"."**分隔数字,总数有$2^{32}$个,大约42亿个可以分配的地址.
* IPV6地址是用八个**":"**分隔数字,总数有$2^{128}$个.

#### IP数据报首部格式

![img](https://www.holelin.cn/img/cs-base/http/IP%E6%95%B0%E6%8D%AE%E6%8A%A5%E9%A6%96%E9%83%A8%E6%A0%BC%E5%BC%8F.jpg)

* 版本号: 指IP协议所使用的版本.通信双方使用的IP协议版本必须一致.目前广泛使用的IP协议版本号为4,即IPv4.
* 首部长度:IP的首部长度,可表示的最大十进制数值是15.注意,该字段所表示的单位是32位字长(4字节).因此,当IP首部长度为1111(即十进制的15)时,首部长度就达到60字节.当IP分组的首部长度不是4字节的整数倍时,必须利用最后的填充字段加以填充.
* 服务类型:优先级标志位和服务类型标志位.被路由器用来进行流量的优先排序.
* 总长度:指IP首部和数据报中数据之后的长度,单位为字节.总长度字段为16位,因此数据报的最大长度为216-1=65535字节.
* 标识符:一个唯一的标识数字,用来识别一个数据报或者被分片数据包的次序.
* 标记:用来标记一个数据报是否是一组分片数据报的一部分.标志字段中的最低位记为MF(More Fragment).MF=1即表示后面"还有分片"的数据报.MF=0表示这个已是若干数据包分片中最后一个.标志字段中间的一位记为DF(Don't Fragment),意思是"不能分片".只有当DF=0时,才允许分片.
* 分片偏移:一个数据报是一个分片,这个域中的值就会被用来将数据报以正确的顺序重新组装.
* 存活时间:用来定义数据报的生成周期,以经过路由器的条数/秒数进行描述.
* 协议:用来识别在数据包序列中上层协议数据报的类型.
* 首部校验和:一个错误检测机制,用来确认IP首部的内容有没有被损坏或者被篡改.
* 源IP地址:发出数据报的主机的IP地址.
* 目的IP地址:数据报目的地IP地址.
* 选项:保留作额外的IP选项,它包含着源站选路和时间戳的一些选项.
* 数据:使用IP传递的实际参数.

##### `TTL`

* 存活时间(TTL)值定义了在该数据报被丢弃之前,所能经历的时间,或者能经过的最大路由数目.TTL在数据报被创建时就会被定义,而且通常在每次被发往一个路由器的时候减1.

##### IP分片

* 数据报分片是将一个数据流分为更小的片段,是IP用于解决跨域不同类型网络时可靠的一个特性.一个数据报的分片主要是基于第二层数据链路层所使用的最大传输单元(Maximum Transmission Unit MTU)的大小,以及使用这些二层协议的设备情况.在多数情况下,第二层所使用的最大数据报大小是1500字节(不包括14字节的以太网头本身).
* 当一个设备这边传输一个IP数据报时,它将会比较这个数据报的大小,以及将要把这个数据报传送出去的网络接口MTU,用于决定是否需要将这个数据报分片.如果数据报的大小大于MTU,那么这个数据报将会被分片.将一个数据报分片包括下列几个步骤:
  * 设备将数据分为若干个可成功进行传输的数据报.
  * 每个IP首部的总长度域会被设置为每个分片的片段长度.
  * 更多分片标志将会在数据流的所有数据报中设置为1,除了最后一个数据报.
  * IP头中分片部分的分片偏移将会被设置.
  * 数据报被发送出去.
