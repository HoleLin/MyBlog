---
title: Kafka-副本和ISR
mermaid: true
date: 2021-06-23 12:10:28
cover: /img/cover/Kafka.jpg
tags:
- ISR
categories:
- Kafka
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

* Apache Kafka实战--胡夕

### ISR

> 就是Kafka集群动态维护的一组同步副本集合(in-sysn replicas);
* 每个topic分区都有自己的`ISR`列表,`ISR`中的所有副本都与leader保持同步状态.
* 值得注意的是,leader副本总是包含在`ISR`中的,只有`ISR`中的副本才有资格被选举为leader.而 producer写入的一条Kafka消息只有被`ISR`中的所有副本都接收到,被视为"已提交"状态.由此可见,若`ISR`中有N个副本,那么该分区最多可以忍受N-1个副本崩溃而不丢失已提交的消息;

#### foller副本同步

* foller副本只做一件事情:**向leader副本请求数据.**

  <img src="http://www.chenjunlin.vip/img/kafka/Kafka-%E5%89%AF%E6%9C%AC%E5%92%8CISR.png" alt="img" style="zoom:80%;" />

  * **起始位移(base offset)**: 表示该副本当前所含第一条消息的offset;
  * **高水印值(high watermark,HW)**: 副本高水印值.它保存了该副本最新一条已提交消息的位移.
    * **leader分区的HW值决定了副本中已提交消息的范围,也确定了consumer能够获取的消息的上限**,超过HW值的所有消息都被视为"未提交成功的",因而consumer是看不到的;
    * 另外值得注意的是,不是只有leader副本才有HW值.实际上每个follower副本都有HW值,只不过只有leader副本的HW值才能决定clients能看到的消息数量罢了;
  * **日志末端位移(log end offset,LEO)**: 副本日志中下一条待写入消息的offset.
    * 所有的副本都需要维护自己的LEO信息.
    * 每当leader副本接收到producer端推送的消息,它会更新自己的LEO(通常是加1);
    * 同样,follower副本想leader副本请求到数据后也会增加自己的LEO;
    * 事实上只有`ISR`中的所有副本都更新了对应的LEO之后,leader副本才会向右移动HW值,表明消息写入成功

  <img src="http://www.chenjunlin.vip/img/kafka/Kafka-follower-leader%E5%89%AF%E6%9C%AC%E5%90%8C%E6%AD%A5%E6%B5%81%E7%A8%8B.png" alt="img" style="zoom:80%;" />

  > 假设上图中的Kafka集群当前只有一个topic,该topic只有一个分区,分区共有3个副本,一次`ISR`中也是这3个副本.该topic当前没有数据,因此3个副本的LEO都是0,HW值也是0;

  * 现在有一个producer想broker1所在的leader副本发送一条消息,接下来会发生什么呢?
    * broker1上的leader副本接收到消息,把自己的LEO更新为1;
    * broker2和broker3上的follower副本各自发送请求给broker1.
    * broker1分别把该消息推送给follower副本;
    * follower副本接收到消息后各自更新自己的LEO为1;
    * leader副本接收到其他follower副本的数据请求响应(response)之后,更新HW值为1.此时位移0的这条消息才可以被consumer消费;
  * 对于设置了`acks`=1的producer而言,只有完整的做完上面所有5步操作,producer才能正常返回,这也标志着这条消息发送成功.

### ISR设计

* **0.9.0.0版本之前**

  > 0.9.0.0版本之前,kafka提供了一个参数`replica.lag.max.message`,用于控制follower副本落后leader副本的消息数.一旦超过个这个消息数,则视为该follower为"不同步(out-of-sysnc)"状态,从而被Kafka"踢出"ISR.

* **导致follower与leader不同步的原因**

  * **请求速度追不上**: follower副本在一段时间内都无法追上leader副本端的消息接收速度.
    * 比如follower副本所在broker的网络I/O开销过大导致备份消息速度持续慢与从leader处获取消息的速度;
  * **进程卡住**: follower在一段时间内无法向leader请求数据;
    * 比如说follower频繁GC或程序bug等
  * **新创建的副本**: 如果用户增加副本数,那么新创建的follower副本在启动后全力追赶leader进度.在追赶进度这段时间内通常都是与leader不同步的.
  
  
