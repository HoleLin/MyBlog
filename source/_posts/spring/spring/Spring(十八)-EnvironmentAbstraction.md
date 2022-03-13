---
title: Spring(十八)-Environment Abstraction
mermaid: true
date: 2021-06-27 21:05:11
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- EnvironmentAbstraction
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

#### 理解Spring `Environment`抽象

* 统一的Spring配置属性管理
  * Spring Framework 3.1开始引入`Environment`抽象,它统一Spring配置属性的存储,包括占位符处理和类型转换,不仅完整地替换`PropertyPlaceholderConfigurer`,而且还支持更丰富的配置属性源`PropertySource`;
* 条件化Spring Bean装配管理
  * 通过`Environment Porfiles`信息,帮助Spring容器提供条件化地装配Bean;

#### Spring E`nvironment`接口使用场景

* 用于属性占位符处理
* 用于转换Spring配置属性类型
* 用于存储Spring配置属性源(`PropertySource`)
* 用于`Profiles`状态的维护

#### `Environment`占位符处理

* Spring 3.1前占位符处理
  * 组件: ~~`org.springframework.beans.factory.config.PropertyPlaceholderConfigurer`~~ ``
  * 接口: `org.springframework.util.StringValueResolver`
* Spring 3.1+占位符处理
  * 组件: `org.springframework.context.support.PropertySourcesPlaceholderConfigurer`
  * 接口: `org.springframework.beans.factory.config.EmbeddedValueResolver`

#### 理解条件配置Spring `Profils`

* Spring 3.1条件配置

  * API: `org.springframework.core.env.ConfigurableEnvironment`

    * 修改: 

      * `void setActiveProfiles(String... profiles);`
      * `void addActiveProfile(String profile);`
      * `void setDefaultProfiles(String... profiles);`

    * 获取: 

      * `String[] getActiveProfiles();`
      * `String[] getDefaultProfiles();`

    * 匹配: 

      ```java
      @Deprecated
      boolean acceptsProfiles(String... profiles);
      ```

      ```java
      boolean acceptsProfiles(Profiles profiles);
      ```

  * 注解: `@org.springframework.context.annotation.Profile`

#### Spring 4 重构`@Profile`

* 基于Spring 4 `org.springframework.context.annotation.Condition`接口
  * `org.springframework.context.annotation.ProfileCondition`

#### 依赖注入`Environment`

* 直接依赖注入
  * 通过`EnvironmentAware`接口回调
  * 通过`@Autowired`注入`Environment`
* 间接依赖注入
  * 通过`ApplicationContextAware`接口回调
  * 通过`@Autowired`注入`ApplicationContext`
* 直接依赖查找
  * 通过`org.springframework.context.ConfigurableApplicationContext#ENVIRONMENT_BEAN_NAME`
* 间接依赖直接
  * 通过`org.springframework.context.ConfigurableApplicationContext#getEnvironment()`

#### 依赖注入`@Value`

* 通过注入`@Value`
  * 实现`org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor`

#### Spring类型转换在`Environment`中的运用

* `Environment`底层实现
  * 底层实现: `org.springframework.core.env.PropertySourcesPropertyResolver`
    * 核心方法: `org.springframework.core.env.AbstractPropertyResolver#convertValueIfNecessary`
  * 底层服务: `org.springframework.core.convert.ConversionService`
    * 默认实现: `org.springframework.core.convert.support.DefaultConversionService`

#### Spring配置属性源`PropertySource`

* API
  * 单配置属性源:`org.springframework.core.env.PropertySource`
  * 多配置属性源:`org.springframework.core.env.PropertySources`
* 注解
  * 单配置属性源: `@org.springframework.context.annotation.PropertySource`
  * 多配置属性源: `@org.springframework.context.annotation.PropertySources`

* 关联
  * 存储对象: `org.springframework.core.env.MutablePropertySources`
  * 关联方法: `org.springframework.core.env.ConfigurableEnvironment#getPropertySources()`

#### Spring内建的配置属性源

* 内建`PropertySource`

  | PropertySource类型                                           | 说明                       |
  | ------------------------------------------------------------ | -------------------------- |
  | `org.springframework.core.env.CommandLinePropertySource`     | 命令行配置属性源           |
  | `org.springframework.jndi.JndiPropertySource`                | JNDI配置属性源             |
  | `org.springframework.core.env.PropertiesPropertySource`      | `Properties`配置属性源     |
  | `org.springframework.web.context.support.ServletConfigPropertySource` | `Servlet`配置属性源        |
  | `org.springframework.web.context.support.ServletContextPropertySource` | `ServletContext`配置属性源 |
  | `org.springframework.core.env.SystemEnvironmentPropertySource` | 环境变量配置属性源         |

#### 基于注解的扩展Spring配置属性源

* `@org.springframework.context.annotation.PropertySource`实现原理
  * 入口: `org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass`
    * `org.springframework.context.annotation.ConfigurationClassParser#processPropertySource`
  * 4.3 新增语义
    * 配置属性字符编码- `encoding`
    * `org.springframework.core.io.support.PropertySourceFactory`
  * 适配对象: `org.springframework.core.env.CompositePropertySource`

####  基于API扩展Spring配置属性源

* Spring应用上下文启动前装配: `PropertySource`
* Spring应用上下文启动后装配: `PropertySource`



  
