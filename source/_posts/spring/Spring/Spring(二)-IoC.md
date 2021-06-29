---
title: Spring(二) Inversion of Control (IoC)
date: 2021-06-23 15:40:34
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- IoC 
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

#### 什么是IoC?

> 控制反转IoC(Inversion of Control)，是一种设计思想，DI(依赖注入)是实现IoC的一种方法;

### IoC主要实现策略

![img](http://www.chenjunlin.vip/img/spring/ioc/IoC%E4%B8%BB%E8%A6%81%E5%AE%9E%E7%8E%B0%E7%AD%96%E7%95%A5.png)

* **使用服务定位模式(Service locator pattern)**
  * Java EE 中使用的模式，一般通过 JNDI 技术获取Java EE组件，如获取 EJB 组件，或者DataSource相关数据源。
* **依赖注入(Dependency injection)**
  * **构造器注入**
  * **参数注入**
  * **Setter注入**
  * **接口注入**
* **上下文的依赖查询**(ContextUalized lookup)
  *  Spring 参考了Java Beans中的实现方式，Java beans 中有一个通用的上下文 beancontext，既可以传输bean，又可以管理bean的层次 
* **模板方法**(Template Method Design Pattern)
* **策略模式**(Strategy Design patten)

### IoC容器的职责

> * 实现与执行的任务之间要解耦;
> * 实现bean的管理,让应用程序不用过渡关注bean的生命周期以及依赖;

![img](http://www.chenjunlin.vip/img/spring/ioc/IoC%E4%B8%BB%E8%A6%81%E8%81%8C%E8%B4%A3.png)

### 通用职责

* **依赖处理**
  * 依赖查找
  * 依赖注入
* **生命周期管理**
  * 容器的生命周期(启动,停止...);
  * 托管的资源(Java Beans或其他资源的生命周期);

* **配置**
  * 容器的配置
  * 外部化配置
  * 托管的资源(Java Beans或其他资源)

### IoC的实现

* Java SE
  * Java Beans
  * Java ServiceLoader SPI
  * JNDI(Java Naming and Directory Interface)
* Java EE
  * EJB(Enterprise Java Beans)
  * Servlet
* 开源
  * Apache Avalon
  * PicoContainter
  * Google Guice
  * Spring Framework

### 传统IoC容器的实现

* Java Beans作为IoC容器
  * 特性
    * 依赖查找
    * 生命周期管理
    * 配置元信息
    * 事件
    * 自定义
    * 资源管理
    * 持久化

### 依赖查找和依赖注入

* 优劣对比

  | 类型       | 依赖查找     | 依赖注入      |
  | ---------- | ------------ | ------------- |
  | 依赖处理   | 主动获取     | 被动提供      |
  | 实现便利性 | 相对繁琐     | 相对便利      |
  | 代码侵入性 | 侵入业务代码 | 低侵入        |
  | API依赖性  | 依赖容器API  | 不依赖容器API |
  | 可读性     | 良好         | 一般          |

#### Spring IoC依赖来源

* **自定义Bean**(自己用XML配置或注解配置的Bean)
* **容器内建Bean对象**(非自己定义的Bean,Spring容器初始化的Bean)
* **容器内建依赖**(非Bean,不能通过依赖查找获取,getBean(XX));
  * 通过`AutowireCapableBeanFactory#resolveDependency()`方法来注册,并非是一个Spring Bean,无法通过依赖查找获取;

#### Spring IoC配置元信息

* Bean定义配置
  * 基于XML文件
  * 基于Properties文件
  * 基于Java注解
  * 基于JavaAPI
* IoC容器配置
  * 基于XML文件
  * 基于Java注解
  * 基于Java API
* 外部化属性配置
  * 基于Java注解 `@Value`

#### Spring 应用上下文

* ApplicationContext除了IoC容器角色,还提供
  * 面向切面(AOP)
  * 配置元信息(Configuration MetaData)
  * 资源管理(Resources)
  * 事件(Events)
  * 国际化(il8n)
  * 注解(Annotations)
  * Environment对象(Environment Abstracttion)

#### 使用Spring IoC容器

* BeanFactory是Spring底层IoC容器
* ApplicationContext是具备应用特征的BeanFactory超集
