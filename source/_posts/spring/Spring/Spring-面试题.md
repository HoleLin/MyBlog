---
title: Spring-面试题
mermaid: true
date: 2021-06-23 15:17:06
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 面试题
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

#### 什么是Spring Framework?

* Spring 是一个开源应用框架，旨在降低应用程序开发的复杂度。
* 它是轻量级、松散耦合的。
* Spring核心主要两部分:
  * IOC - 控制反转
  * AOP - 面向切面编程可以将应用业务逻辑和系统服务分离，以实现高内聚。

#### Spring Framework有哪些核心模块?

* **spring-core**: Spring基础API模块,如资源管理,泛型处理;
* **spring-beans**: Spring Bean相关,如依赖查找,依赖注入;
* **spring-aop**: Spring AOP处理,如动态代理,AOP字节码提升;
* **spring-context**: 事件驱动,注解驱动,模块驱动等;
* **spring-expression**: Spring表达式语言模块;

#### 什么是IoC?

> IoC是反转控制,类似好莱坞原则,主要实现有依赖注入(DI)和依赖查找;

#### 依赖查找和依赖注入的区别?

* 依赖查找是主动或手动的依赖查找方式,通常需要依赖容器或标准API实现;
* 依赖注入则是自动或手动的依赖绑定方式,无需依赖特定的容器和API;

#### Spring作为IoC容器有什么优势?

> 典型的IoC管理,依赖查找和依赖注入
>
> * AOP抽象
> * 强大的第三方整合
> * 事务抽象
> * 事务机制
> * 易测试性
> * SPI扩展
> * 更好的面向对象

#### BeanFactory和ObjectFactory的区别?

* ObjectFactory通常是**针对单类Bean做延迟获取的**,BeanFactory则是全局Bean管理的容器;
* ObjectFacotry与BeanFactory均提供依赖查找的能力,不过ObjectFactory仅关注一个类或一种类型的Bean依赖查找,并且自身不具备依赖查找的能力,能力由BeanFactory输出;
* BeanFactory则提供单一类型集合类以及层次类型的依赖查找;

#### BeanFactory和FactoryBean的区别?

* **BeanFactory，以Factory结尾，表示它是一个工厂类(接口)，用于管理Bean的一个工厂。在Spring中，BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。**
* **FactoryBean以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了FactoryBean<T>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。**实现了这个接口后，Spring在容器初始化时，把实现这个接口的Bean取出来，使用接口的getObject()方法来生成我们要想的Bean

* FactoryBean是一种特殊的Bean,需要注册IoC容器通过容器getBean获取FactoryBean#getObject()方法的内容,而BeanFactory#getBean则是依赖查找,如果Bean没有初始化,那么将从底层查找或构建;

#### 内建依赖和自定以Bean的区别

* 内建依赖指的是`DefaultListableBeanFactory`中的`resolvableDependencies`这个map里面保存的Bean;
* 自定义Bean指的是通过`DefaultSingletonBeanRegistry#registerSingleton`手动注册的Bean,它们都在BeanFactory里面;
* 依赖注入的时候比如`@AutoWired()`(`AutowireAnnotationBeanPostProcessor`处理)会调用`DefaultListableBeanFactory#resolveDependency`方法去`resolvableDependencies`里面去查找
* 依赖查找BeanFactory.getBean(xxx)是不会去`resolvableDependencies`这个map中去查找的;

#### BeanFactory和ApplicationContext谁才是Spring IoC容器?

* `BeanFactory`是Spring底层IoC容器;
* `ApplicationContext`是具备应用特性的BeanFactory超集;
* `ApplicationContext`是`BeanFactory`的子接口,说明`ApplicationContext is BeanFactory`,并且`ApplicationContext`的包装类,也就是内部组合了`BeanFactory`的实现(`DefaultListableBeanFactory`)

