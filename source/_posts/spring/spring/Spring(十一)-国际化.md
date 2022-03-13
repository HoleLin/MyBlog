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
- Spring Framework
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
  * 抽象实现: `java.util.ResourceBundle`
  * Properties资源实现: `java.util.PropertyResourceBundle`
  * 具体实现: `java.util.ListResourceBundle`
  
* ResourceBundle核心特性
  
  * Key-Value设计
  * 层次性设计
  * 缓存设计
  * 字符编码控制: `java.util.ResourceBundle.Control`(`@Since 1.6`)
  * `Control SPI`扩展: `java.util.spi.ResourceBundleControlProvider`(`@Since 1.8`)

#### Java文本格式化

* 核心接口: `java.text.MessageFormat`
* 基本用法:
  * 设置消息格式模式: `new MessageFormat(...)`;
  * 格式化: `format(new Object[]{...})`;
* 消息格式模式:
  * 格式元素: `{ArgumentIndex(,FormatType,(FormatStyle))}`
  * `FormatType`: 消息格式类型,可选项,每种类型在`number,date,time`和`choice`类型选其一
  * `FormatStyle`: 消息格式风格, 可选项,可包括: `short`,`medium`,`long`,`full`,`integer`,`currency`,`percent`
* 高级特性
  * 重置消息格式模式
  * 重置`java.util.Locale`
  * 重置`java.text.Format`

#### MessageSource开箱即用实现

* 基于 ResourceBundle + MessageFormat 组合 MessageSource 实现 
  * `org.springframework.context.support.ResourceBundleMessageSource `
* 可重载 Properties + MessageFormat 组合 MessageSource 实现  
  * `org.springframework.context.support.ReloadableResourceBundleMessageSource `

#### MessageSource内建依赖

* MessageSource 內建 Bean 可能来源 
  * 预注册 Bean 名称为：`messageSource`，类型为：`MessageSource Bean `
  * 默认內建实现 - `DelegatingMessageSource`  
    * 层次性查找 MessageSource 对象  

  

  

  
