---
title: Spring(十一) 国际化
mermaid: true
date: 2021-06-27 21:01:48
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- i18n 
categories:
- Spring
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

#### Spring国际化使用场景

* 普通国际化文案
* Bean Validation校验国际化文案
* Web站点页面渲染
* Web MVC错误消息提示

#### Spring国际化接口

* 核心接口: `org.springframework.context.MessageSource`

* 主要概念

  * 文案模板编码(code)
  * 文案模板参数(args)
  * 区域(Locale)

#### 层次性MessageSource

* Spring 层次性接口回顾
  * ` org.springframework.beans.factory.HierarchicalBeanFactory`
  * `org.springframework.context.ApplicationContext`
  * `org.springframework.beans.factory.config.BeanDefinition`
* Spring 层次性国际化接口
  * `org.springframework.context.HierarchicalMessageSource`

#### Java国际化标准实现

* 核心接口
  * 抽象实现: `java,t`

  
