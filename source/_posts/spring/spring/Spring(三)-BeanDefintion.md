---
title: Spring(三) BeanDefinition
mermaid: true
date: 2021-06-27 20:57:01
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- BeanDefinition
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

### Spring `BeanDefinition`

* `BeanDefinition`是`SpringFramework`中定义Bean的配置元信息接口,包含:
  * Bean的类名;
  * Bean行为配置元素,如作用域,自动绑定模式,生命周期回调等;
  * 其他Bean引用,又可称为合作者(collaborators)或者依赖(dependencies);
  * 配置设置,比如Bean属性(Properties);

#### `BeanDefinition`元信息

| 属性(Property)           | 说明                                       |
| ------------------------ | ------------------------------------------ |
| Class                    | Bean全类名,必须是具体类,不能用抽象类或接口 |
| Name                     | Bean的名称或ID                             |
| Scope                    | Bean的作用域(如,Singleton,Prototype等)     |
| Constructor arguments    | Bean构造器参数(用于依赖注入)               |
| Properties               | Bean属性设置(用于依赖注入)                 |
| Autowiring mode          | Bean自动绑定模式(如:通过名称byName)        |
| Lazy initialization mode | Bean延迟初始化模式(延迟和非延迟)           |
| Initialization method    | Bean 初始化回调方法名称                    |
| Destruction method       | Bean销毁回调方法名称                       |

#### `BeanDefinition`构建

* 通过`BeanDefinitionBuilder`
* 通过`AbstractBeanDefinition`以及派生类

#### 命名Spring Bean

* Bean的名称
  * 每个Bean拥有一个或多个标识符(`identifiers`),这些标识符在Bean所在的容器必须是唯一的.通常,一个Bean仅有一个标识符,如果需要额外的,可以考虑使用`Alias`来扩充;
  * 在基于XML的配置元信息中,开发人员可用`id`或者`name`属性来规定Bean的标识符.通常Bean的标识符由字母组成,允许出现特殊字符.如果要引入Bean的别名话,可在name属性使用半角逗号(",")或分号(";")来间隔;
  * Bean的id或name属性并非必须指定,如果留空的话,容器会为Bean自动生成一个唯一的名称.Bean的命名尽管没有限制,不过官方建议采用驼峰的方式,更符合Java的命令约定.
* Bean名称生成器(`BeanNameGenerator`)
  * 由Spring Framework 2.0.3引入,框架内建两种实现
    * `DefaultBeanNameGenerator`: 默认通用`BeanNameGenerator`实现
    * `AnnotationBeanNameGenerator`: 基于注解扫描的`BeanNameGenerator`实现,起始于Spring Framework 2.5

#### Spring Bean的别名

* Bean别名(Alias)的价值

  * 复用现有的`BeanDefinition`

  * 更具有场景化的命名方法,如

    ```xml
    <alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
    <alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
    ```

#### `BeanDefinition`注册

  * XML配置元信息

    ```xml
    <bean name=".." .../>
    ```

  * Java注解配置元信息

    * `@Bean`
    * `@Component`
    * `@Import`

  * Java API 配置元信息

    * 命名方式: `BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)  `
    * 非命名方式: `BeanDefinitionReaderUtils#registerWithGeneratedName(AbstractBeanDefinition,Be
      anDefinitionRegistry)  `
    * 配置类方式: `AnnotatedBeanDefinitionReader#register(Class...)  `

  * 外部单例对象注册:

    * Java API 配置元信息
      * `SingletonBeanRegistry#registerSingleton   `

#### 实例化 Spring Bean

* 常规方式
  * 通过构造器(配置元信息: XML,Java注解和Java API)
  * 通过静态工厂方法(配置元信息: XML和Java API)
  * 通过Bean工厂方法(配置元信息: XML和Java API)
  * 通过`FactoryBean`(配置元信息: XML,Java注解和Java API)
* 特殊方式
  * 通过 `ServiceLoaderFactoryBean`（配置元信息：XML、Java 注解和 Java API ）
  *  通过 `AutowireCapableBeanFactory#createBean(java.lang.Class, int, boolean)`
  *  通过 `BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)  `

#### 初始化 Spring Bean

* Bean初始化(Initialization)
  * `@PostConstructor`标注方法
  * 实现`InitializingBean`接口的`afterPropertiesSet()`方法
  * 自定义初始化方法
    * XML配置: `<bean init-method="init" .../>`
    * Java注解:`@Bean(initMethod="init")`
    * Java API: `AbstractBeanDefinition#setInitMethodName(String)  `

#### 延迟初始化Spring Bean

* Bean延迟初始化(Lazy Initialization)
  * XML配置: `<bean lazy-init="true" .../>`
  * Java 注解: `@Lazy(true)`

#### 销毁Spring Bean

* Bean销毁(Destroy)
  * `@PreDestory`标注方法
  * 实现`DisposableBean`接口的`destroy()`方法
  * 自定义销毁方法
    * XML 配置：`<bean destroy="destroy" ... />`
    *  Java 注解：`@Bean(destroy="destroy")`
    *  Java API：`AbstractBeanDefinition#setDestroyMethodName(String)  `

#### 垃圾回收Spring Bean

* Bean垃圾回收(GC)
  * 关闭Spring容器(应用上下文)
  * 执行GC
  * Spring Bean覆盖`finalize()`方法回调
