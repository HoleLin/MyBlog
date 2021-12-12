---
title: RabbitMQ-基础知识点
date: 2021-11-26 10:04:50
cover: /img/cover/RabbitMQ.jpeg
tags:
- 基础
categories:
- RabbitMQ
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

* https://blog.csdn.net/unique_perfect/article/details/109380996
* amqp.0-10
* [AMQP](https://www.amqp.org/)

### 引言

* RabbitMQ是**基于AMQP协议**，erlang语言开发，是部署最广泛的开源,消息中间件,是最受欢迎的开源消息中间件之一。

* AMQP 协议AMQP（advanced message queuing protocol）在2003年时被提出，最早用于解决金融领不同平台之间的消息传递交互问题。顾名思义，AMQP是一种协议，更准确的说是一种binary wire-level protocol(链接协议)。这是其和 	JMS的本质差别，AMQP不从API层进行限定，而是直接 	定义网络交换的数据格式。这使得实现了AMQP的provider天然性就是跨平台的。

* AMQP Model:

  ![img](https://www.holelin.cn/img/rabbitmq/amqp_model.png)
  
  * 举例说明
  
    > 如果我们拿电子邮件系统做个类比，我们会发现AMQP的概念并不激进:
    > 1. AMQP消息类似于电子邮件消息。
    > 2. 消息队列类似于邮箱。
    > 3.消费者类似于获取和删除电子邮件的邮件客户机。
    > 4. 交易所类似于MTA(邮件传输代理)，它检查电子邮件，并根据路由键和表决定如何将电子邮件发送到一个或多个邮箱。
    > 5. 路由键对应于电子邮件的to:或Cc:或Bcc:地址，没有服务器信息(路由完全是AMQP服务器内部的)。
    > 6. 每个交换实例就像一个单独的MTA进程，处理一些电子邮件子域或特定类型的电子邮件流量。
    > 7. 绑定就像MTA路由表中的一个条目。
  
  * Exchange(交换机):交换器接受来自生产者应用程序的消息，并根据预先安排的条件将它们路由到消息队列。这些标准称为“绑定”。交换器是匹配和路由引擎。也就是说，它们检查消息并使用它们的绑定表，决定如何将这些消息转发到消息队列。交换器从不存储消息。
  
    > An exchange accepts messages from a producer application and routes them to message queues according to pre- arranged criteria. These criteria are called "bindings". Exchanges are matching and routing engines. That is, they inspect messages and using their binding tables, decide how to forward these messages to message queues. Exchanges never store messages.
  
  * Routing Key(路由键): 
  
    * 在一般情况下，交换检查消息的属性、头字段和正文内容，并使用此数据(可能还有来自其他来源的数据)决定如何路由消息。
    * 在大多数简单的情况下，交换检查单个键字段，我们称之为“路由键”。路由键是一个虚拟地址，交换器可以使用它来决定如何路由消息。
    * 对于点对点路由，路由键是消息队列的名称。
    * 对于主题发布-子路由，路由键是主题层次结构值。在更复杂的情况下，路由可能基于消息头字段和/或消息体。
  
    > In the general case, an exchange examines a message's properties, its header fields, and its body content, and using this and possibly data from other sources, decides how to route the message.
    > In the majority of simple cases, the exchange examines a single key field, which we call the "routing key". The routing key is a virtual address that the exchange may use to decide how to route the message.
    > For point-to-point routing, the routing key is the name of a message queue. For topic pub-sub routing, the routing key is the topic hierarchy value.
    > In more complex cases, routing may be based on message header fields and/or the message body.
  
* Message Flow

  ![img](https://www.holelin.cn/img/rabbitmq/message_flow.png)

  

  
