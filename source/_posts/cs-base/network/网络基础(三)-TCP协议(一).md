---
title: 网络基础-TCP协议
date: 2021-10-20 14:27:32
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
* [UDP-RFC768](https://www.ietf.org/rfc/rfc0768)

### `TCP`协议

#### 为什么需要TCP协议?TCP工作在哪一层?

* IP层是[不可靠],它不保证网络包的交付,不保证网络包的按序交付,也不保证网络中的数据完整性.
  * 如果需要保证网络数据包的可靠性,那么就需要由上层(传输层)的TCP协议来负责.
  * 因为TCP是一个工作在传输层的可靠数据传输的服务,它能确保接收端接收的网络包是无损坏,无间隔,非冗余和按序的.

* TCP是面**向连接,可靠的,基于字节流**的传输层通信协议.
  * 面向连接:一定是一对一才能连接,不能想UDP协议可以一个主机同时向多个主机发送消息.
  * 可靠的:无论网络链路中出现了怎样的链路变化,TCP都可以保证一个报文一定能够到达接收端.
  * 字节流:消息是没有边界的,所以无论消息有多大都可以进行传输,并且消息是有序的,当前一个消息没有收到的时候,即使它先收到了后面的字节,那么也不能扔给应用层去处理,同时对**重复**的报文会自动放弃.

#### `TCP`连接

* 用于保证可靠性和流量控制维护的某些状态信息,这些信息的组合,包括**Socket,序列号和窗口大小**称为连接.
  * Socket: 由IP地址和端口号组成
  * 序列号(Sequence number): 用来解决乱序问题等
  * 窗口大小(Windows sizes): 用来做流量控制

##### `TCP`连接的确定--`TCP`四元组

* 源地址
* 源端口
* 目的地址
* 目的端口
* 源地址和目的地址的字段(32位)是在IP头部中,作用是通过IP协议发送报文给对方主机.
* 源端口和目的端口的字段(16位)是在TCP头部中,作用是告诉TCP协议应该把报文给哪个进程.

##### `TCP`最大连接数

* 服务器通常固定在某个端口上监听,等待客户端的连接请求.

* 客户端IP和端口是可变的,其理论值计算公式如下:

  ```
  最大TCP连接数 = 客户端的IP数 * 客户端的端口数
  ```

  * 对于IPv4,客户端的IP数最多为2的32次方,客户端的端口数最多为2的16次方,也就是服务端单机最大TCP连接数,约为2的48次方.
  * 当然,服务端最大并发TCP连接数远不能达到理论上限.
    * 首先主要是**文件描述符限制**,Socket都是文件,所以首先要通过`ulimit`配置文件描述符的数目;
    * 另一个是**内存限制**,每个TCP连接都要占用一定内存,操作系统的内存是有限的.

##### `TCP`和`UDP`的区别

###### 连接

* TCP是面向连接的传输层协议,传输数据前先要建立连接.
* UDP是不需要连接,即刻传输数据.

###### 服务对象

* TCP是一对一的两点服务,即一条连接只有两个端点.
* UDP支持一对一,一对多,多对多的交互通信.

###### 可靠性

* TCP是可靠交付数据的,数据可以无差错,不丢失,不重复,按需到达.
* UDP是尽最大努力交付,不保证可靠交互数据.

###### 拥塞控制,流量控制

* TCP有拥塞控制和流量控制机制,保证数据传输的安全性.
* UDP则没有,即使网络非常拥堵了,也不会影响UDP的发送速率.

###### 首部开销

* TCP首部长度较长,会有一定的开销,首部在没有使用选项字段时是20个字节,如果使用了选项字段则会变长的.
* UDP首部只有8个字节,并且是固定不变的,开销较小.

###### 传输方式

* TCP是流式传输,没有边界,保证顺序和可靠.
* UDP是一个包一个包的发送,是有边界的,但可能会丢包和乱序.

###### 分片不同

* TCP的数据大小如果大于MSS大小,则会在传输层进行分片,目标主机收到后,也同样在传输层组装TCP数据包,如果中途丢失了一个分片,只需要传输丢失的这个分片.
* UDP的数据大小如果小于MTU大小,则会在IP层进行分片,目标主机收到后,在IP层组装完数据,接着再传给传输层,但是如果中途丢失了一分片,在实现可靠传输的UDP时则就需要重传所有的数据包,这样传输效率非常差,所有通常UDP的报文应该小于MTU.

`TCP`和`UDP`应用场景:

* 由于TCP是面向连接,能保证数据的可靠性交付,因此长用于
  * FTP文件传输
  * HTTP/HTTPS
* 由于UDP面向无连接,它可以随时发送数据,再机上UDP本身的处理既简单又高效,因此常用于:
  * 包总量较少的通信,如DNS,SNMP等
  * 视频,音频等多媒体通信
  * 广播通信

#### `TCP`协议头

```sh
    0                   1                   2                   3   
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                            TCP Header Format
```

* 源端口号 Source Port:  16 bits The source port number.

* 目标端口号 Destination Port:  16 bits The destination port number.

* 序列号 Sequence Number:  32 bits 

  > The sequence number of the first data octet in this segment (except when SYN is present). If SYN is present the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1.

* 确认应答号 Acknowledgment Number:  32 bits

  > If the ACK control bit is set this field contains the value of the next sequence number the sender of the segment is expecting to receive.  Once a connection is established this is always sent.

* 首部长度 Data Offset:  4 bits

  > The number of 32 bit words in the TCP Header.  This indicates where the data begins.  The TCP header (even one including options) is an integral number of 32 bits long.

* 保留 Reserved:  6 bits

  > Reserved for future use.  Must be zero.

* 控制位 Control Bits:  6 bits (from left to right)

  ```
      URG:  Urgent Pointer field significant
      ACK:  Acknowledgment field significant,该位为1时,[确认应答]的字段变为有效,TCP规定除了最初建立连接时的SYN包之外该位必须设置为1.
      PSH:  Push Function
      RST:  Reset the connection,该位为1时,表示TCP连接中出现异常必须强制断开连接.
      SYN:  Synchronize sequence numbers,该位为1,表示希望建立连接,并在[序列号]的字段进行序列号初始值的设定.
      FIN:  No more data from sender,该位为1时,表示今后不会再有数据发送,希望断开连接.当通信结束希望断开连接时,通信双方的主机之间就可以相互交换FIN位为1的TCP段
  ```

* 窗口大小 Window:  16 bits

  > The number of data octets beginning with the one indicated in the acknowledgment field which the sender of this segment is willing to accept.

* 校验和 Checksum:  16 bits

* 紧急指针 Urgent Pointer  16bits

#### `TCP`连接运行过程中状态

* LISTEN - represents waiting for a connection request from any remote TCP and port.
* SYN-SENT - represents waiting for a matching connection requestafter having sent a connection request.
* SYN-RECEIVED - represents waiting for a confirming connection request acknowledgment after having both received and sent a connection request.
* ESTABLISHED - represents an open connection, data received can be delivered to the user.  The normal state for the data transfer phase of the connection.
* FIN-WAIT-1 - represents waiting for a connection termination request from the remote TCP, or an acknowledgment of the connection termination request previously sent.
* FIN-WAIT-2 - represents waiting for a connection termination request from the remote TCP.
* CLOSE-WAIT - represents waiting for a connection termination request from the local user.
* CLOSING - represents waiting for a connection termination request acknowledgment from the remote TCP.
* LAST-ACK - represents waiting for an acknowledgment of the connection termination request previously sent to the remote TCP (which includes an acknowledgment of its connection termination request).
* TIME-WAIT - represents waiting for enough time to pass to be sure the remote TCP received the acknowledgment of its connection termination request.
* CLOSED - represents no connection state at all.

#### `TCP`三次握手

<img src="https://www.holelin.cn/img/cs-base/http/TCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.png" alt="img" style="zoom:67%;" />

```
                               TCP A                                                TCP B

                            1.  CLOSED                                               LISTEN

                            2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

                            3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

                            4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

                            5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED

                                    Basic 3-Way Handshake for Connection Synchronization

                                                          Figure 7.
```

* 使用`TCP`协议进行通信的双方必须先建立连接,然后才能开始传输数据.为了确保连接双方可靠性,在双方建立连接时,`TCP`协议采用了三次握手策略;

###### 握手过程

* 一开始,客户端和服务端都处于`CLOSED`状态.先是服务端主动鉴定某个端口,处于`LISTEN`状态.

* 客户端会随机初始化序号(`client_isn`),将此序号置于TCP首部的序号字段中,同时把`SYN`标志位置为1,表示SYN报文.接着把第一个SYN报文发送给服务端,表示向服务端发起连接,该报文不包含应用层数据,之后客户端处于`SYN-SENT`状态

  <img src="https://www.holelin.cn/img/cs-base/http/TCP%E6%8F%A1%E6%89%8B%E7%AC%AC%E4%B8%80%E6%AC%A1%E6%8A%A5%E6%96%87.png" alt="img" style="zoom: 33%;" />

* 服务端收到客户端的`SYN`报文后,首先服务端也随机初始化自己的序号(`server_isn`),将此序号填入TCP首部的序号字段,其次把TCP首部的确认应答号字段填入`client_isn + 1`,接着把`SYN`和`ACK`标志位置为1.最后把该报文发给客户端,该报文也不包含应用层数据,之后服务端处于`SYN-RCVD`状态.

  <img src="https://www.holelin.cn/img/cs-base/http/TCP%E6%8F%A1%E6%89%8B%E7%AC%AC%E4%BA%8C%E6%AC%A1%E6%8A%A5%E6%96%87.png" alt="img" style="zoom:33%;" />

* 客户端收到服务端报文后,还要向服务端回应最后一个应答报文,首先该应答报文TCP首部`ACK`标志位置为1,其次确认应答号字段填入`server_isn + 1`,最后把报文发送给服务端,这次报文可以携带客户端到服务器的数据,之后客户端处于`ESTABLISHED`状态.

  <img src="https://www.holelin.cn/img/cs-base/http/TCP%E6%8F%A1%E6%89%8B%E7%AC%AC%E4%B8%89%E6%AC%A1%E6%8A%A5%E6%96%87.png" alt="img" style="zoom:33%;" />

* 服务器收到客户端的应答报文后,也进入`ESTABLISHED`状态.

* 上述过程中可以发现第三次握手是可以携带数据的,前两次握手是不可用携带数据的.

* 一旦完成三次握手,双方处于`ESTABLISHED`状态,此时连接就已建立完成.客户端和服务端可以相互发送数据了.

* 可以使用`netstat -napt`查看TCP连接状态.

##### 采用三次握手的原因

###### 三次握手才能保证双方具有接收和发送的能力.

###### 三次握手才可以阻止重复历史连接的初始化(主要原因).

  * 为了防止旧的重复连接初始化造成混乱.
  * 三次握手则可以在客户端(发送方)准备发送第三次报文时,客户端因有足够的上下文判断当前连接是否是历史连接:
    * 如果是历史连接(序列号过期或超时),则第三次握手发送的报文是`RST`报文,以此中止历史连接.
    * 如果不是历史连接,则第三次发送的报文是`ACK`,通信双方就会成功建立连接.

###### 三次握手才可以同步双方的初始序列号.

* TCP协议的通信双方,都必须维护一个序列号,序列号时可靠传输的一个关键因素,它的作用:
  * 接收方可以去除重复的数据.
  * 接收方可以根据数据包的序列号按序接收.
  * 可以标识发送出去的数据包中,哪些是已经被对方收到的.
* 序列号在TCP连接中占据着非常重要的作用,所以当客户端发送携带初始化序列号的`SYN`报文的时候,需要服务端回一个`ACK`应答报文,表示客户端的`SYN`报文已被服务端成功接收,那当服务端发送初始序列号给客户端的时候,依然也要得到客户端的应答,这样一来一回,才能确保双方的初始化序列号能被可靠的同步.

###### 三次握手才可以避免资源浪费.

###### 总结

* 三次握手能防止历史连接的建立,能减少双方不必要的资源开销,能帮助双方同步初始化序列号.序列号能保证数据包不重复,不丢弃和按序传输.
* 两次握手:无法防止历史连接的建立,会造成双方资源的浪费,也无法可靠的同步双方序列号.
* 四次握手:三次握手就已经理论上最少可靠连接建立,所以不需要使用更多的通信次数.

##### MTU和MSS

<img src="https://www.holelin.cn/img/cs-base/http/MTU%E5%92%8CMSS.png" alt="img" style="zoom:50%;" />

* MTU:一个网络包的最大长度,以太网中一般为1500字节.
  * 当IP层数据超过MTU大小要发送,则IP层就要进行分片,把数据分片成
* MSS:除去IP和TCP头部之后,一个网络包所能容纳的TCP数据的最大长度.
  * 当TCP层发现数据超过MSS时,则就先会进行分片
  * 当一个TCP分片丢失后,进行重发时也是以MSS为单位,而不用重传所有的分片,大大增加了重传的效率.

#### `SYN`攻击

* TCP连接建立是需要三次握手,假设攻击者短时间伪造不同IP地址的`SYN`报文,服务端每接收到一个`SYN`报文,就进入`SYN_RCVD`状态,但服务端发送出去的`ACK + SYN`报文,无法得到未知IP主机的`ACK`应答,久而久之就会占满服务端的`SYN`接收队列(未连接队列),使得服务器不能为正常用户服务.

##### 避免SYN攻击方式

* 修改Linux内核参数,控制队列大小和当队列满时应做什么处理.

  ```sh
  # 当网卡接收数据包的速度大于内核处理的速度时,会有一个队列保存这些数据包.控制该队列的最大值
  net.core.netdev_max_backlog
  # SYN_REVD状态连接的最大个数
  net.ipv4.tcp_max_syn_backlog
  # 超出处理能力时,对新的SYN直接回复RST,丢弃连接
  net.ipv4.tcp_abort_onoverflow
  ```

* Linux内核的SYN(未完成连接建立)队列与Accpet(已完成连接建立)队列如何工作的?

  * 正常流程

    * 当服务端收到客户端的SYN报文时,会将其加入到内核的SYN队列;
    * 接着发送SYN+ACK给客户端,等待客户端回应ACK报文;
    * 服务端接收到ACK报文后,从SYN队列移除放入到Accept队列;
    * 应用通过调用accept() socket接口,从Accpet队列取出连接.

  * 应用程序过慢

    * 如果应用程序过慢时,就会导致Accpet队列被占满.

  * 受到SYN攻击

    * 如果不断受到SYN攻击,就会导致SYN队列被占满

      ```sh
      # tcp_syncookies的方式可以应对SYN攻击的方法
      net.ipv4.tcp_syncookies = 1
      ```

  * `SYN`队列占满,启动`cookie`

    * 当`SYN`队列满之后,后续服务器收到SYN包,不进入`SYN`队列
    * 计算出一个`cookie`值,再以`SYN + ACK`中的序列号返回客户端
    * 服务端收到客户端的应答报文时,服务器会检查这个`ACK`包的合法性.如果合法,直接放入到Accpet队列.
    * 最后应用通过accpet()socket接口,从Accpet队列取出连接

#### `TCP`四次挥手

![img](https://www.holelin.cn/img/cs-base/http/TCP%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B.png)

```
         
                                TCP A                                                TCP B

                            1.  ESTABLISHED                                          ESTABLISHED

                            2.  (Close)
                                FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  --> CLOSE-WAIT

                            3.  FIN-WAIT-2  <-- <SEQ=300><ACK=101><CTL=ACK>      <-- CLOSE-WAIT

                            4.                                                       (Close)
                                TIME-WAIT   <-- <SEQ=300><ACK=101><CTL=FIN,ACK>  <-- LAST-ACK

                            5.  TIME-WAIT   --> <SEQ=101><ACK=301><CTL=ACK>      --> CLOSED

                            6.  (2 MSL)
                                CLOSED

                                                   Normal Close Sequence

                                                         Figure 13.             
```



* 主动关闭连接的一方，调用close()；协议层发送FIN包
* 被动关闭的一方收到FIN包后，协议层回复ACK；然后被动关闭的一方，进入CLOSE_WAIT状态，主动关闭的一方等待对方关闭，则进入FIN_WAIT_2状态；此时，主动关闭的一方 等待 被动关闭一方的应用程序，调用close操作
* 被动关闭的一方在完成所有数据发送后，调用close()操作；此时，协议层发送FIN包给主动关闭的一方，等待对方的ACK，被动关闭的一方进入LAST_ACK状态；
* 主动关闭的一方收到FIN包，协议层回复ACK；此时，主动关闭连接的一方，进入TIME_WAIT状态；而被动关闭的一方，进入CLOSED状态
* 等待2MSL时间，主动关闭的一方，结束TIME_WAIT，进入CLOSED状态

##### 结论

* 主动关闭连接的一方 - 也就是主动调用socket的close操作的一方，最终会进入`TIME_WAIT`状态
* 被动关闭连接的一方，有一个中间状态，即`CLOSE_WAIT`，因为协议层在等待上层的应用程序，主动调用close操作后才主动关闭这条连接
* `TIME_WAIT`会默认等待`2MSL`时间后，才最终进入`CLOSED`状态；
* 在一个连接没有进入`CLOSED`状态之前，这个连接是不能被重用的！

#### `TCP`连接状态转换图

```
                              +---------+ ---------\      active OPEN  
                              |  CLOSED |            \    -----------  
                              +---------+<---------\   \   create TCB  
                                |     ^              \   \  snd SYN    
                   passive OPEN |     |   CLOSE        \   \           
                   ------------ |     | ----------       \   \         
                    create TCB  |     | delete TCB         \   \       
                                V     |                      \   \     
                              +---------+            CLOSE    |    \   
                              |  LISTEN |          ---------- |     |  
                              +---------+          delete TCB |     |  
                   rcv SYN      |     |     SEND              |     |  
                  -----------   |     |    -------            |     V  
 +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
 |         |<-----------------           ------------------>|         |
 |   SYN   |                    rcv SYN                     |   SYN   |
 |   RCVD  |<-----------------------------------------------|   SENT  |
 |         |                    snd ACK                     |         |
 |         |------------------           -------------------|         |
 +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
   |           --------------   |     |   -----------                  
   |                  x         |     |     snd ACK                    
   |                            V     V                                
   |  CLOSE                   +---------+                              
   | -------                  |  ESTAB  |                              
   | snd FIN                  +---------+                              
   |                   CLOSE    |     |    rcv FIN                     
   V                  -------   |     |    -------                     
 +---------+          snd FIN  /       \   snd ACK          +---------+
 |  FIN    |<-----------------           ------------------>|  CLOSE  |
 | WAIT-1  |------------------                              |   WAIT  |
 +---------+          rcv FIN  \                            +---------+
   | rcv ACK of FIN   -------   |                            CLOSE  |  
   | --------------   snd ACK   |                           ------- |  
   V        x                   V                           snd FIN V  
 +---------+                  +---------+                   +---------+
 |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
 +---------+                  +---------+                   +---------+
   |                rcv ACK of FIN |                 rcv ACK of FIN |  
   |  rcv FIN       -------------- |    Timeout=2MSL -------------- |  
   |  -------              x       V    ------------        x       V  
    \ snd ACK                 +---------+delete TCB         +---------+
     ------------------------>|TIME WAIT|------------------>| CLOSED  |
                              +---------+                   +---------+

                      TCP Connection State Diagram
```

![img](https://www.holelin.cn/img/cs-base/http/TCP%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2%E5%9B%BE.png)

##### `TIME_WAIT`状态

* `TIME_WAIT`状态也被称为`2MSL`等待状态.在该状态将会等待两倍于最大段生存期(Maximum Segment Lifetime,MSL)的时间

  * `TIME_WAIT`等待2倍的MSL的原因为:网络中可能存在来自发送的数据包,当这些发送方的数据包被接收方处理后又会向对方发送响应,所以一来一回响应等待2倍的时间
  * `2MSL`的时间是从客户端接收到`FIN`后发送`ACK`开始计时的.如果在`TIME_WAIT`时间内,因为客户端的`ACK`没有传输到服务端,客户端又接收到了服务端重发的`FIN`报文,那么`2MSL`时间将重新计时.

* 在Linux系统中,`net.ipv4.tcp_fin_timeout`的数值记录了`2MSL`状态需要等待的超时时间(以秒为单位)

  ```properties
  net.ipv4.tcp_fin_timeout = 2
  # 为1表示允许将TIME-WAIT的句柄重新用于新的TCP连接
  net.ipv4.tcp_tw_reuse = 1
  # 为1表示开启TCP连接中TIME-WAIT的快速回收，NAT环境可能导致DROP掉SYN包（回复RST）
  net.ipv4.tcp_tw_recycle = 1
  # 为1时SYN Cookies，当SYN等待队列溢出时启用cookies来处理，可防范少量SYN攻击
  net.ipv4.tcp_syncookies = 1
  #  端口最大backlog内核限制，防止占用过大内核内存
  net.ipv4.ip_local_port_range = 4000 65000
  net.ipv4.tcp_max_syn_backlog = 16384
  # 保持TIME_WAIT套接字的最大个数，超过这个数字TIME_WAIT套接字将立刻被清除并打印警告信息
  net.ipv4.tcp_max_tw_buckets = 36000
  net.ipv4.route.gc_timeout = 100
  #  对一个新建连接，内核要发送多少个SYN连接请求才决定放弃，不应该大于255
  net.ipv4.tcp_syn_retries = 1
  net.ipv4.tcp_synack_retries = 1
  net.core.somaxconn = 16384
  net.core.netdev_max_backlog = 16384
  # 不属于任何进程（已经从进程上下文中删除）的sockets最大个数，超过这个值会被立即RESET，并同时显示警告信息
  net.ipv4.tcp_max_orphans = 16384
  ```

* 假设已设定MSL的数值,按照规则:当TCP执行一个主动关闭并发送最终的ACK时,连接必须处于`TIME_WAIT`状态并持续两倍于最大生存周期(`2MSL`)的时间.这样能够让TCP重新发送最终的ACK以避免出现丢失的情况.重新发送最终的ACK并不是因为重传了ACK(它们并不消耗序列号,也不会被TCP重传),而是因为通信另一方重传了它的FIN(它消耗一个序列号).事实上,TCP总是重传FIN,直到它收到一个最终的ACK.

* 影响`2MSL`等待状态的因素是当TCP处于等待状态时,通信双方将该连接(客户端IP地址,客户端端口号,服务器IP地址,服务器端口号)(四元组)定义为不可重新使用.只有当`2MSL`等待结束时,或一条新连接使用的初始化序列号超过了连接之前的实例使用的最高序列号时,或者允许使用时间戳选项来区分之前连接实例的报文段以避免混淆时,这条连接才能被再次使用.不幸的是,一些实现施加了更加严格的约束,**在这些系统中,如果一个端口号被处于`2MSL`等待状态的任何通信端所用,那么该端口号将不能被再次使用.**

* 使用`ss -tan state time-wait | wc -l ` 查看`TIME_WAIT`的连接数量

##### 为什么需要`TIME_WAIT`状态

* 主动发起关闭连接的一方,才会有`TIME_WAIT`状态
* 需要`TIME_WAIT`状态,主要是两个原因:
  * 防止具有相同四元组的旧数据包被收到;
    * 经过`2MSL`这个时间,足以让两个方向的数据包都被丢弃,使得原来连接的数据包在网络中都自然消失,再出现的数据包一定都是新建立连接所产生的.
  * 保证被动关闭的连接的一方被正确关闭,即保证最后的`ACK`能被动关闭接收,从而帮助其正常关闭;

##### `TIME_WAIT`过多的危害

* 第一是内存资源占用
* 第二是对端口资源的占用,一个TCP连接至少消耗一个本地端口
  * 一般可以开启的端口为`32768~61000`,也可以通过`net.ipv4.ip_local_port_range`
* 如果发起连接一方的`TIMET_WAIT`状态过多,占满了所有端口资源,则会导致无法创建新连接.
  * 客户端手端口资源限制:
    * 客户端`TIME_WAIT`过多,就会导致端口资源被占用,因为端口就`65536`个,被占满就会导致无法创建新的连接.
  * 服务端受系统资源限制:
    * 由于一个四元组标志TCP连接,理论上服务端可以建立很多连接,服务端确实只监听一个端口,但是会把连接扔给线程,所以理论上监听的端口可以继续监听.但是线程池处理不了那么多一直不断的连接了.所以当服务端出现大量`TIME_WAIT`时,系统资源被占满时,会导致不过来新的连接.

##### 优化`TIME_WAIT`

* 打开`net.ipv4.tcp_tw_reuse`和`net.ipv4_tcp_timestamps`选项;
* `net.ipv4.tcp_max_tw_buckets`
  * 这个值默认为18000,当系统中处于`TIME_WAIT`的连接一旦超过这个值时,系统就会将后面的`TIME_WAIT`连接重置.
* 程序中使用`SO_LINGER`,应用强制使用`RST`关闭

###### `net.ipv4.tcp_tw_reuse`和`net.ipv4.tcp_timestamps`

* 开启`net.ipv4.tcp_tw_reuse = 1 `,可以复用处于`TIME_WAIT`的socket为新的连接所用
  * 注: `tcp_tw_reuse`功能只**能用于客户端(连接发起方)**,因为开启了该功能,在调用connect()函数时,内核会随机找一个`time_wait`状态超过1秒的连接给新的连接复用;**解决的是accpet后的问题.**
  * 使用这个选项,还有个前提,需要打开TCP时间戳的支持,即`net.ipv4.tcp_timestamps = 1`(默认即为1)
  * 这个时间戳的字段是在TCP头部的选项里,用于记录TCP发送方的当前时间戳和从对端接收到的最新时间戳
  * 由于引入了时间戳,之前的`2MSL`问题就不存在了,因为重复的数据包会因为时间戳过期被自然丢弃.
* `SO_REUSEADDR`是用户态的选项,**用于连接的服务方**,用来告诉操作系统内核,如果端口已被占用,但是TCP连接状态位于`TIME_WAIT`,可以重用端口.如果端口忙,而TCP处于其他状态,重用会有"Address already in use" 的错误信息.
  * `SO_REUSEADDR`是为了解决`TIME_WAIT`状态带来的端口占用问题,以及支持同一个port对应多个IP,**解决的是bind时的问题.**

#### 如果已建立了连接,但是客户端突然出现故障了怎么办?

* TCP有个保活机制.这个机制的原理为:

  > 定义一个时间段,在这个时间段内,如果没有任何连接相关的活动,TCP报活机制会开始作用,每隔一个时间间隔,发送一个探测报文,该探测报文包含的数据非常少,如果连续几个探测报文都没有得到响应,则认为当前的TCP连接已经死亡,系统内核将错误信息通知给上层应用程序.

* 在Linux内核可以有对应的参数可以设置保活时间,保活探测的次数,保活探测的时间间隔,一下都为默认值:

  ```properties
  # 表示保活时间是7200秒，单位为秒，缺省是7200秒（即2小时）,也就是2个小时内如果没有任何连接相关的活动,则会启动保活机制
  net.ipv4.tcp_keepalive_time = 7200
  # 表示每次检测间隔为75秒
  net.ipv4.tcp_keepalive_intvl=75
  # 表示检测9次无响应,认为对方是不可达的,从而终端本次的连接.
  net.ipv4.tcp_keepalive_probes=9
  ```

  * 也就是说在Linux中,最少需要经过2小时11分15秒才能发现一个死亡连接`tcp_keepalive_time + (tcp_keepalive_intvl * tcp_keepalive_probes)`

#### `Socket`编程

<img src="https://www.holelin.cn/img/cs-base/http/socket%E7%BC%96%E7%A8%8B.png" alt="img" style="zoom:67%;" />

* 服务端和客户端初始化socket,得到文件描述符;
* 服务端调用bind绑定IP地址和端口;
* 服务端调用listen,进行监听;
* 服务端调用accept,等待客户端连接;
* 客户端调用connect,向服务器端的地址和端口发起连接请求;
* 客户端调用write写入数据;服务端调用read读取数据;
* 客户端断开连接时会调用close,服务端read读取数据的时候,就会读取到了EOF,待处理完数数据后,服务端调用close,表示连接关闭.
* 注: 当服务端调用accept时,连接成功了会返回一个已完成连接的socket,后续用来传输数据.所以,监听的socket和真正用来传送数据的socket,是两个socket,一个叫做**监听socket**,一个叫做**已完成连接socket**,成功建立连接之后,双方开始通过read和write函数来读写数据,就像往一个文件流里面写东西.

##### `listen`时候参数`backlog`的意义

* Linux内核中会维护两个队列

  * 未完成连接队列(`SYN`队列):接收到一个SYN建立连接请求,处于`SYN_RCVD`状态;

  * 已完成连接队列(`Accpet`队列): 已完成TCP三次握手过程,处于`ESTABLISHED`状态;

    <img src="https://www.holelin.cn/img/cs-base/http/SYN%E9%98%9F%E5%88%97%E5%92%8CAccpet%E9%98%9F%E5%88%97.png" alt="img" style="zoom:67%;" />

    *  在早期Linux内核backlog是SYN队列大小,也就是未完成的队列大小.
    * 在Linux内核2.2之后,backlog编程accpet队列,也就是已完成连接建立的队列长度,所以通常认为backlog是accpet队列,但是**上限值是内核参数somaxconn的大小**,也就是说**accpet队列长度=min(backlog,somaxconn)**

##### accpet发生在三次握手哪一步?

<img src="https://www.holelin.cn/img/cs-base/http/TCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8BSocket%E5%BD%A2%E5%BC%8F.png" alt="img" style="zoom:67%;" />

* 客户端的协议栈向服务器端发送了`SYN`包,并告诉服务器端当前发送序列号`client_isn`,客户端进入`SYN_SENT`状态;
* 服务端的协议栈收到这个包,和客户端进行`ACK`应答,应答值为`client_isn+1`,表示`SYN`包`client_isn`的确认,同时服务器也发送`SYN`包,告诉客户端当前我的发送序列号为`server_isn`,服务端进入`SYN_RCVD`状态;
* 客户端协议栈收到`ACK`之后,使得应用程序从`connect`调用返回,表示客户端到服务端的单向连接建立成功,客户端的状态为`ESTABLISHED`,同时客户端协议栈也会对服务器端的`SYN`包进行应答,应答数据为`server_isn+1`;
* 应答包达到服务器端后,服务器端协议栈使得`accpet`阻塞调用返回,这个时候服务端到客户端的单向连接也建立成功,服务器端也进入`ESTABLISHED`状态.
* **客户端connect成功返回是第二次握手,服务端accpet成功返回在三次握手成功后.**

##### 客户端调用了close了,连接断开的流程是什么?

<img src="https://www.holelin.cn/img/cs-base/http/TCP%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8BSocket%E5%BD%A2%E5%BC%8F.png" alt="img" style="zoom:67%;" />

* 客户端调用`close`,表明客户端没有数据需要发送了,则此时会向服务端发送`FIN`报文,进入`FIN_WAIT_1`状态;
* 服务端接收到了`FIN`报文,TCP协议栈会为`FIN`包插入一个文件结束符`EOF`到接收缓冲区中,应用程序可以通过read调用感知这个`FIN`包.这个`EOF`会被放在已排队等候的其他已接收的数据之后,这就意味着服务端需要处理这种异常情况,因为`EOF`表示在该连接上再无额外数据到达.此时服务端进入`CLOSE_WAIT`状态.
* 接着,当处理完成数据后,自然就会读到`EOF`,于是也调用`close`关闭它的套接字,这回使得客户端发出一个`FIN`包,之后处于`LAST_ACK`状态;
* 客户端接收到服务端的FIN包,并发送`ACK`确认包给服务端,此时客户端将进入`TIME_WAIT`状态;
* 服务端收到`ACK`确认包,就进入最后的`CLOSE`状态
* 客户端进过`2MSL`时间之后,也进入`CLOSE`状态

