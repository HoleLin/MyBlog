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

#### 注入和查找的依赖来源是否相同?

* 否;
* 依赖查找的来源仅限于`Spring BeanDefinition`以及单例对象;
* 而依赖注入的来源还包括`Resolvable Dependency`以及`@Value`所标注的外部化配置;

#### 单例对象能在IoC容器启动后注册吗?

* 可以的
* 单例对象的注册与`BeanDefinition`不同,`BeanDefinition`会被`ConfigurableListableBeanFactory#freezeConfiguration`方法影响,从而冻结注册,单例对象则没有这个限制;

#### Spring依赖注入的来源有哪些?

* Spring `BeanDefinition`
* 单例对象
* `Resolvable Dependency`
* `@Value`外部化配置

#### 有多少中依赖注入的方式 ?

*  构造器注入
* `Setter`注入
* 字段注入
* 方法注入
* 接口回调注入

 #### Spring内建的Bean的作用域有几种?

* singletion
* prototype
* request
* session
* application
* websocket

#### Singleton Bean是否在一个应用是唯一的?

* 否;
* Singletoin bean仅在当前Spring IoC容器(BeanFactory)中是单例对象;

#### BeanPostProcessor 的使用场景有哪些？  

* `BeanPostProcessor` 提供 Spring Bean 初始化前和初始化后的生命周期回调，分别对应 `postProcessBeforeInitialization` 以及`postProcessAfterInitialization`方法，允许对关心的 Bean 进行扩展，甚至是替换。  
* 其中，`ApplicationContext `相关的 Aware 回调也是基于`BeanPostProcessor `实现，即 `ApplicationContextAwareProcessor`。  

#### BeanFactoryPostProcessor 与BeanPostProcessor 的区别  

* `BeanFactoryPostProcessor `是 Spring `BeanFactory`（实际为`ConfigurableListableBeanFactory`） 的后置处理器，用于扩展BeanFactory，或通过 BeanFactory 进行依赖查找和依赖注入。  
* `BeanFactoryPostProcessor `必须有 Spring `ApplicationContext`执行，BeanFactory 无法与其直接交互;
* 而 `BeanPostProcessor` 则直接与BeanFactory 关联，属于 N 对 1 的关系。  

##### BeanFactory是怎么处理Bean生命周期的

BeanFactory默认实现为`DefaultListableBeanFactory`其中Bean生命周期与方法映射如下:

* BeanDefinition注册阶段:  `registerBeanDefinition`
* BeanDefinition合并阶段:  `getMergedLocalBeanDefinition`
* Bean实例化前阶段:  `resolveBeforeInstantiation`
* Bean实例化: `createBeanInstance`
* Bean实例化后阶段: `populateBean`
* Bean属性赋值前阶段: `populateBean`
* Bean属性赋值阶段: `populateBean`
* Bean Aware接口回调阶段:  `initializeBean`
* Bean 初始化前阶段:  `initializeBean`
* Bean 初始化阶段:  `initializeBean`
* Bean 初始化后阶段:  `initializeBean`
* Bean 初始化完成阶段: `preInstantiateSingletons`
* Bean 销毁前阶段: `destoryBean`
* Bean 销毁阶段: `destoryBean`

#### Spring 內建 XML Schema 常见有哪些？  

| 命名空间 | 所属模块       | Schema 资源 URL                                              |
| -------- | -------------- | ------------------------------------------------------------ |
| beans    | spring-beans   | https://www.springframework.org/schema/beans/spring-beans.xsd |
| context  | spring-context | https://www.springframework.org/schema/context/spring context.xsd |
| aop      | spring-aop     | https://www.springframework.org/schema/aop/spring-aop.xsd    |
| tx       | spring-tx      | https://www.springframework.org/schema/tx/spring-tx.xsd      |
| util     | spring-beans   | https://www.springframework.org/schema/util/spring-util.xsd  |
| tool     | spring-beans   | https://www.springframework.org/schema/tool/spring-tool.xsd  |

#### Spring配置元信息具体有哪些？  

* Bean 配置元信息：通过媒介（如 XML、Proeprties 等），解析 BeanDefinition
* IoC 容器配置元信息：通过媒介（如 XML、Proeprties 等），控制 IoC 容器行为，比如注解驱动、AOP 等
* 外部化配置：通过资源抽象（如 Proeprties、YAML 等），控制 PropertySource
* Spring Profile：通过外部化配置，提供条件分支流程  

#### Extensible XML authoring 的缺点？  

* 高复杂度：开发人员需要熟悉 XML Schema，spring.handlers，spring.schemas以及 Spring API 。
* 嵌套元素支持较弱：通常需要使用方法递归或者其嵌套解析的方式处理嵌套（子）元素。
* XML 处理性能较差：Spring XML 基于 DOM Level 3 API 实现，该 API 便于理解，然而性能较差。
* XML 框架移植性差：很难适配高性能和便利性的 XML 框架，如 JAXB  

#### Spring配置资源中有哪些常见类型?

* XML资源
* Properties资源
* YAML资源

#### 请例举不同类型Spring配置资源?

* XML资源
  * 普通Bean Definition XML配置资源 : `*.xml`
  * Spring Schema 资源 :`*.xsd`
* Properties资源
  * 普通Properties格式资源: `*.properties`
  * Spring Handler实现类映射文件: `META-INF/spring.handlers`
  * Spring Schema资源映射文件: `META-INF/spring.schemas`
* YAML资源
  * 普通YAML配置资源: `*.yaml`或`*.yml`

#### Java标准资源管理扩展的步骤

* 简易实现
  * 实现`URLStreamHandler`并放置在`sun.net.www.protocol.${protocol}.Handler`包下
* 自定义实现
  * 实现`URLStreamHandler`
  * 添加`-Djava.protocol.handler.pkgs`启动参数,指向`URLStreamHandler`实现类的包下
* 高级实现
  * 实现`URLStreamHandlerFactory`并传递到URL之中

#### SpringBoot为什么要新建`MessageSource Bean`

* `AbstractApplicationContext` 的实现决定了 `MessageSource` 內建实现 
* Spring Boot 通过外部化配置简化 `MessageSource Bean` 构建  
* `Spring Boot` 基于 `Bean Validation` 校验非常普遍

#### Spring国际化接口有哪些?

* 核心接口: `MessageSource`
* 层次性接口: `org.springframework.context.HierarchicalMessageSource  `

#### Spring 有哪些 MessageSource 內建实现 ?

* `org.springframework.context.support.ResourceBundleMessageSource`
* `org.springframework.context.support.ReloadableResourceBundleMessageSource`
* `org.springframework.context.support.StaticMessageSource`
* `org.springframework.context.support.DelegatingMessageSource `

#### 如何实现配置自动更新 MessageSource ?

* 主要技术
  * `Java NIO2`：`java.nio.file.WatchService`
  * `Java Concurrency` : `java.util.concurrent.ExecutorService`
  * `Spring`：`org.springframework.context.support.AbstractMessageSource`

#### Spring 校验接口是哪个？

* `org.springframework.validation.Validator`

#### Spring 有哪些校验核心组件?

* 检验器：`org.springframework.validation.Validator`
* 错误收集器：`org.springframework.validation.Errors`
* Java Bean 错误描述：`org.springframework.validation.ObjectError`
* Java Bean 属性错误描述：`org.springframework.validation.FieldError`
* Bean Validation 适配：`org.springframework.validation.beanvalidation.LocalValidatorFactoryBean`  

#### Spring 类型转换器接口有哪些 ?

* 类型转换接口 - `org.springframework.core.convert.converter.Converter`
* 通用类型转换接口 - `org.springframework.core.convert.converter.GenericConverter`
* 类型条件接口 - `org.springframework.core.convert.converter.ConditionalConverter`
* 综合类型转换接口 -`org.springframework.core.convert.converter.ConditionalGenericConverter `

#### Java 泛型擦写发生在编译时还是运行时?

* 运行时

#### 请介绍 Java 5 Type 类型的派生类或接口 ?

* `java.lang.Class`
* `java.lang.reflect.GenericArrayType`
* `java.lang.reflect.ParameterizedType`
* `java.lang.reflect.TypeVariable`
* `java.lang.reflect.WildcardType`

#### Spring事件核心接口/组件

* Spring事件: `org.springframework.context.ApplicationEvent`
* Spring事件监听器: `org.springframework.context.ApplicationListener`
* Spring事件发布器: `org.springframework.context.ApplicationEventPublisher`
* Spring事件广播器: `org.springframework.context.event.ApplicationEventMulticaster`

#### Spring同步和异步事件处理的使用场景

* Spring同步事件: 绝大多数Spring使用场景: 如`ContextRefreshedEvent`
* Spring异步事件: 主要`@EventListener`与`@Async`,实现异步处理,不阻塞主线程,不如长时间的数据计算任务等.不要轻易调整`SimpleApplicationEventMulticaster`中关联的`taskExecutor`对象,除非使用者非常了解Spring事件机制,否则容易出现异常行为;
