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

### 引言-AMQP

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

### 使用场景

#### 推送通知

* "发布/订阅"是RabbitMQ的重要功能.可以用"发布/订阅"功能来实现通知功能.消费者(consumer)一直监听RabbitMQ的数据.如果RabbitMQ有数据,则消费者会按照先进先出的规则逐条进行消费.而生产者(producer)只需要将数据存入RabbitMQ.
* "发布/订阅"功能支持三种模式:一对一,一对多,广播.

#### 异步任务

* 后台系统接到任务后,将其分解分多个小任务,只要分别完成这些小任务,整个任务便可以完成.但是,如果某个小任务很费时,且延迟执行并不影响整个任务,则可以将该任务放入消息队列中去处理,以便加快请求响应时间.

#### 多平台应用的通信

* 不同语言的软件解耦,可以最大限度地减少程序之间的相互依赖,提高系统可用性及可扩展性,同时还增加了消息的可靠传输和事务管理功能.
* RabbitMQ提供两种事务模式:
  * AMQP事务模式
  * Confirm事务模式

#### 消息延迟

* 利用RabbitMQ消息队列延迟功能,可以实现订单,支付过期定时取消功能.因为延迟队列存储延时消息,所以当消息被发送之后,消费者不是立即拿到消息,而是等待指定时间才拿到这个消息进行消费.

* 死信,计时器,定时任务也可以实现延迟或定时功能.

  > 死信是一种不能投递给收件人，也不能退还给寄件人的邮件。死信也被称为无法投递的邮件。

* 要实现消息队列延迟功能,一般采用官方提供的插件"rabbitmq_delayed_message_exchange"来实现,但RabbitMQ版本必须为3.5.8版本以上才支持该队列.如果低于这个版本,则可以利用死信来完成

### 特性

* 信息确认,RabbitMQ有两种应答模式:
  * 自动应答:当RabbitMQ把消息发送到接收端,接收端从队列接收消息时,会自动发送应答消息给服务端.
  * 手动应答:需要开发人员手动调用方式告诉服务端已经收到.
* 队列持久化:队列可以被持久化,但是否为持久化,要看持久化设置.
* 信息持久化:设置`properties.DeliveryMode`值即可.默认为1,代表不是持久的,2代表持久化.
* 消息拒收:接收端可以拒收消息,而且在发送"reject"命令时,可以选择是否要把拒收的消息重新放回队列中.
* 消息的QoS: 在接收端设置的.发送端没有任何变化,接收端的代码也比较简单,只需要加上`channel.BasicQos(0,1,false)`.

### RabbitMQ基本概念

#### 生成者,消费者,代理

* 生产者: 消息的创建这,负责创建和推送数据到消息服务器.
* 消费者: 消息的接收方,用于处理消息和确认消息.
* 代理: RabbitMQ本身,扮演"快递"的角色,本身不生产消息.

#### 消息队列

* Queue是RabbitMQ的内部对象,用于存储生产者的消息直到发送给消费者,也是消费者接收消息的地方.RabbitMQ中消息也都是只能存储在Queue中,多个消费者可以订阅同一个Queue.
* Queue的重要属性
  * 持久性: 若启用,则队列将会在消息协商期(broker)重启前都有效.
  * 自动删除: 若启用,则队列将会在所有的消息者停止使用之后自动删除.
  * 惰性: 若没有声明队列,则应用程序调用队列时会导致异常,并不会主动声明.
  * 排他性: 若启用,则声明它的消费者才能使用.

#### 交换机

* Exchange用于接收,分配消息.生产者先要指定一个"routing key",然后将消息发送到交换机.这个"routing key"需要与"Exchange Type"及"binding key"联合使用才能最终生效,然后交换机将消息路由到一个或多个Queue中或丢弃.
* 在虚拟主机中消息协商期(broker)中,每个Exchange都有唯一的名字
* Exchange包含4中类型: `direct`,`topic`,`fanout`,`headers`.不同的类型代表绑定到队列行为不同.
  * `direct`: `direct`类型的行为是"先匹配,再投送".在绑定队列时会设定一个routing key,只有在消息的routing key与队列匹配时,消息才会被交换机投送到绑定的队列中.允许一个队列通过一个固定的routing key(通常为队列的名字),进行绑定.direct交换机将消息根据其routing key属性投递到包含对应key属性的绑定器上.
    * Direct Exchange是RabbitMQ默认的交换机模式,也是最简单的模式.它根据routing key全文匹配去寻找队列.
  * `topic`: 按规则转发消息(最灵活).主题交换机(topic Exchange)转发消息主要根据通配符.队列和交换机的绑定会定以一种路由模式,通配符就要在这种路由模式和路由键之间匹配后,交换机才能转发消息.
    * 在这种交换机模式下,路由键必须是一串字符,用**"."**隔开.
    * topic还支持消息的routing key,用"*"或"#"的模式进行绑定.**"\*"**匹配一个单词,"#"匹配0个或多个单词,
  * `headers`: 它根据应用程序消息的特定属性进行匹配,可以在binding key中标记消息为可选或必选.在队列与交换机绑定时,会设定一组键值对规则.消息中也包括一组键值(headers属性),当这些键值对中有一对,或全部匹配时,消息被投送到对应队列.
  * `fanout`: 消息广播的模式,即将消息广播到所有绑定到它的队列中,而不考虑routing key的值.若配置了routing key,则routing key依然会忽略.

#### 绑定

* RabbitMQ中通过绑定(binding),将Exchange与Queue关联起来.这样RabbitMQ就知道如何正确地将消息路由到指定的Queue了.
* 在绑定Exchange与Queue时,一般会指定一个binding key.消费者将消息发送给Exchange时,一般会指定一个routing key.如果binding key与routing key想匹配,则消息将会被路由到对应的Queue中.
* 绑定是生产者和消费者消息传递的连接.生产者发送消息到Exchange,消费者从Queue接收消息,都是根据绑定来执行的.

### RabbitMQ的6种工作模式

#### 简单模式

* 生产者把消费者放入队列,消费者获得消息.这个模式只有一个消费者,一个生产者,一个队列,只需要配置主机参数,其他参数使用默认值即可通信.

#### 工作队列模式

* 一个生产者,多个消费者,每个消费者获取到的消息唯一.

#### 订阅模式

#### Routing转发模式

#### 主题转发模式

#### RPC模式

