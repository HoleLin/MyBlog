---
title: Spring(十七)-Annotations注解
mermaid: true
date: 2021-06-27 21:04:27
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- Annotations
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

### 参数文献

#### Spring注解驱动编程发展历程

* 注解驱动启蒙时代: `Spring Framerwork 1.x`
* 注解驱动过渡时代: `Spring Framerwork 2.x`
* 注解驱动黄金时代: `Spring Framerwork 3.x`
* 注解驱动完善时代: `Spring Framerwork 4.x`
* 注解驱动当下时代: `Spring Framerwork 5.x`

#### Spring核心注解场景分类

* Spring模式注解

  | Spring注解       | 场景说明          | 起始版本 |
  | ---------------- | ----------------- | -------- |
  | `@Repository`    | 数据仓储模式注解  | 2.0      |
  | `@Component`     | 通用组件模式注解  | 2.5      |
  | `@Service`       | 服务模式注解      | 2.5      |
  | `@Controller`    | Web控制器模式注解 | 2.5      |
  | `@Configuration` | 配置类模式注解    | 3.0      |

* 装配注解

  | Spring注解        | 场景说明                                | 起始版本 |
  | ----------------- | --------------------------------------- | -------- |
  | `@ImportResource` | 替换XML元素\<import>                    | 2.5      |
  | `@Import`         | 导入`Configuration`类                   | 2.5      |
  | `@ComponentScan`  | 扫描指定package下标注Spring模式注解的类 | 3.1      |

* 依赖注入注解

  | Spring注解   | 场景说明                          | 起始版本 |
  | ------------ | --------------------------------- | -------- |
  | `@Autowired` | Bean依赖注入,支持多种依赖查找方式 | 2.5      |
  | `@Qualifier` | 细粒度的`@Autowired`依赖查找      | 2.5      |

#### Spring注解编程模型

* 编程模型

  * 元注解(`Meta-Annotations`)
    * `java.lang.annotation.Documented`
    * `java.lang.annotation.Inherited`
    * `java.lang.annotation.Repeatable`
  * Spring模式注解(`Stereotype Annotations`)\
    * `@Repository`
    * `@Component`
    * `@Service`
    * `@Configuration`
    * `org.springframework.boot.SpringBootConfiguration`
    * `@Component`"派生性"原理
      * 核心组件:
        *  `org.springframework.context.annotation.ClassPathBeanDefinitionScanner`
        * `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider`
      * 资源处理 
        * `org.springframework.core.io.support.ResourcePatternResolver`
      * 资源-类元信息
        * `org.springframework.core.type.classreading.MetadataReaderFactory`
      * 类元信息
        * `org.springframework.core.type.ClassMetadata`
        * ASM实现: ~~`org.springframework.core.type.classreading.ClassMetadataReadingVisitor`~~ `org.springframework.core.type.classreading.SimpleAnnotationMetadataReadingVisitor`
        * 反射实现: `org.springframework.core.type.StandardAnnotationMetadata`
      * 注解元信息
        * `org.springframework.core.type.AnnotationMetadata`
        * ASM实现: ~~`org.springframework.core.type.classreading.AnnotationMetadataReadingVisitor`~~ `org.springframework.core.type.classreading.SimpleAnnotationMetadataReadingVisitor`
        * 反射实现: `org.springframework.core.type.StandardAnnotationMetadata`
  * Spring组合注解(`Composed Annotations`)
  * Spring注解属性别名和覆盖(`Attribute Aliases and Overrides`)

  #### Spring `@Enable`模块驱动

  * `@Enable`模块驱动

    > `@Enable`模块驱动是以`@Enable`为前缀的注解驱动编程模型.所谓"模块"是指具备相同领域的功能组件集合,组合所形成一个独立的单元.比如`Web MVC`模块,`AspectJ`代理模块,`Caching`(缓存)模块,`JMX`(Java管理扩展)模块,`Async`(异步处理)模块等

  * 示例
    * `@EnableWebMvc`
    * `@EnabTransactionManagement`
    * `@EnableCaching`
    * `@EnableMBeanExport`
    * `@EnableAsync`
  * `@Enable`模块驱动编程模式
    * 驱动注解: `@EnableXXX`
    * 导入注解: `@Import`具体实现
    * 具体实现: 
      * 基于`Configuration Class`
      * 基于`ImportSelector`接口实现
      * 基于`ImportBeanDefinitionRegistrar`接口实现

  #### Spring条件注解

  * 基于配置条件注解: `@org.springframework.context.annotation.Profile`

    * 关联对象: `org.springframework.core.env.Environment`中的`Profiles`

    * 实现变化: 从Spring 4.0开始,`@Profile`基于`@Conditional`

      ```java
      @Target({ElementType.TYPE, ElementType.METHOD})
      @Retention(RetentionPolicy.RUNTIME)
      @Documented
      @Conditional(ProfileCondition.class)
      public @interface Profile {
      
      	/**
      	 * The set of profiles for which the annotated component should be registered.
      	 */
      	String[] value();
      
      }
      ```

  * 基于编程条件注解: `@org.springframework.context.annotation.Conditional`

    * 关联对象: `org.springframework.context.annotation.Condition`具体实现

  * `@Conditional`实现原理

    * 上下文对象: `org.springframework.context.annotation.ConditionContext`
    * 条件判断: `org.springframework.context.annotation.ConditionEvaluator`
    * 配置阶段: `org.springframework.context.annotation.ConfigurationCondition.ConfigurationPhase`
    * 判断入口: `org.springframework.context.annotation.ConfigurationClassPostProcessor`

  * Spring Boot注解

    | 注解                       | 场景说明                | 起始版本 |
    | -------------------------- | ----------------------- | -------- |
    | `@SpringBootConfiguration` | Spring Boot配置类       | 1.4.0    |
    | `@SpringBootApplication`   | Spring Boot应用引导注解 | 1.2.0    |
    | `@EnableAutoConfiguration` | Spring Boot激活自动装配 | 1.0.0    |

  * Spring Cloud注解

    | 注解                      | 场景说明                           | 起始版本 |
    | ------------------------- | ---------------------------------- | -------- |
    | `@SpringCloudApplication` | Spring Cloud应用引导注解           | 1.0.0    |
    | `@EnableDiscoveryClient`  | Spring Cloud激活服务发现客户端注解 | 1.0.0    |
    | `@EnableCircuitBreaker`   | Spring Cloud激活熔断注解           | 1.0.0    |

    
