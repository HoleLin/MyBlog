---
title: RabbitMQ-开启消息追踪
date: 2021-11-17 13:46:47
cover: /img/cover/RabbitMQ.jpeg
tags:
- 消息追踪
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

#### 开启消费记录功能

```sh
rabbitmq-plugins enable rabbitmq_tracing

rabbitmqctl trace_on
```

![img](https://www.holelin.cn/img/rabbitmq/open_message_trace.png)
