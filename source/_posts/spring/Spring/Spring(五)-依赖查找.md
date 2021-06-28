---
title: Spring-依赖查找
mermaid: true
date: 2021-06-23 16:26:02
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 依赖查找
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

### 依赖查找方式

#### 单一类型依赖查找接口 BeanFactory

##### 根据Bean名称查找 

* `getBean(String beanName)`
* Spring 2.5覆盖默认参数: `getBean(String,Object...)`

#####  根据Bean类型查找

* 实时查找
  * Spring 3.0 `getBean(Class)`
  * Spring4.1覆盖默认参数: `getBean(Class,Object...)`
* 延迟查找
  * `getBeanProvider(Class)`
  * `getBeanProvider(ResolvableType)`

##### 根据Bean名称+类型查找 getBean(String,Class)

#### 集合类型依赖查找接口 ListBeanFactory

##### 根据Bean类型查找 

* 获取同类型Bean名称列表
  * `getBeanNamesForType(Class)`
  * Spring 4.2 `getBeanNamesForType(ResolvableType)`
* 获取同类型Bean实例列表
  * `getBeansOfType(Class)`以及重载方法

##### 根据注解类型查找

* Spring 3.0 获取标注类型Bean名称列表
  * `getBeanNamesForAnnotation(Class<? extends Annotation>)`
* Spring 3.0 获取标注类型Bean实例列表
  * `getBeansWithAnnotation(Class<? extends Annotation>)`
* Spring 3.0 获取指定名称+标注类型Bean实例
  * `findAnnotationOnBean(String,Class<? extends Annotation>)`

#### 层次性依赖查找

##### 层次依赖查找接口 HierarchicalBeanFactory 

* 双亲BeanFactory: `getParentBeanFactory()`
* 层次查找:
  * 根据Bean名称查找
    * 基于`containsLocalBean(String name)`方法实现
  * 根据Bean类型查找实例列表
    * 单一类型:  `BeanFactoryUtils#beanOfType`方法
    * 集合类型: `BeanFactoryUtils#beanNamesForTypeIncludingAncestors`方法
  * 根据Java注解查找名称列表
    * `BeanFactoryUtils#beanNamesForAnnotationIncludingAncestors`

#### 延迟依赖查找

* Bean延迟依赖查找接口
  * `org.springframework.beans.factory.ObjectFactory`
  * `org.springframework.beans.factory.ObjectProvider`
    * Spring 5 对Java8特性扩展
      * 函数式接口
        *  `getIfAvailable()`
        *  `ifAvailable`
      * Stream扩展-stream()

```java
public class DependencyLookupDemo {

    public static void main(String[] args) {
        // 配置 XML 配置文件
        // 启动 Spring 应用上下文
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/dependency-lookup-context.xml");
        // 按照类型查找
        lookupByType(beanFactory);
        // 按照类型查找结合对象
        lookupCollectionByType(beanFactory);
        // 通过注解查找对象
        lookupByAnnotationType(beanFactory);

        lookupInRealTime(beanFactory);
        lookupInLazy(beanFactory);
    }

    private static void lookupByAnnotationType(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = (Map) listableBeanFactory.getBeansWithAnnotation(Super.class);
            System.out.println("查找标注 @Super 所有的 User 集合对象：" + users);
        }
    }

    private static void lookupCollectionByType(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
            System.out.println("查找到的所有的 User 集合对象：" + users);
        }
    }

    private static void lookupByType(BeanFactory beanFactory) {
        User user = beanFactory.getBean(User.class);
        System.out.println("实时查找：" + user);
    }

    private static void lookupInLazy(BeanFactory beanFactory) {
        ObjectFactory<User> objectFactory = (ObjectFactory<User>) beanFactory.getBean("objectFactory");
        User user = objectFactory.getObject();
        System.out.println("延迟查找：" + user);
    }

    private static void lookupInRealTime(BeanFactory beanFactory) {
        User user = (User) beanFactory.getBean("user");
        System.out.println("实时查找：" + user);
    }
}
```

#### 安全依赖查找

* 依赖查找安全性对比

  | 依赖查找类型 | 代表实现                             | 是否安全 |
  | ------------ | ------------------------------------ | -------- |
  | 单一类型查找 | `BeanFactory#getBean`                | 否       |
  |              | `ObjectFactory#getObject`            | 否       |
  |              | `ObjectProvider#getIfAvailable`      | 是       |
  |              |                                      |          |
  | 集合类型查找 | `ListableBeanFactory#getBeansOfType` | 是       |
  |              | `ObjectProvider#stream`              | 是       |

#### 内建可查找的依赖

* `AbstractApplicationContext`内建可查找的依赖

  | Bean名称                      | Bean实例                                      | 使用场景               |
  | ----------------------------- | --------------------------------------------- | ---------------------- |
  | `environment`                 | `Environment`对象                             | 外部化配置以及Profiles |
  | `systemProperties`            | `Map<String, Object> getSystemProperties();`  | Java系统属性           |
  | `systemEnvironment`           | `Map<String, Object> getSystemEnvironment();` | 操作系统环境变量       |
  | `messageSource`               | `MessageSource`对象                           | 国际化文案             |
  | `applicationEventMulticaster` | `ApplicationEventMulticaster`对象             | Spring事件广播器       |
  | `lifecycleProcessor`          | `LifecycleProcessor`对象                      | `Lifecycle Bean`处理器 |

#### 依赖查找中的经典异常

* BeanException子类型

  | 异常类型                          | 触发条件(举例)                              | 场景举例                                      |
  | --------------------------------- | ------------------------------------------- | --------------------------------------------- |
  | `NoSuchBeanDefinitionException`   | 当查找 Bean 不存在于 IoC 容器时             | `BeanFactory#getBean ObjectFactory#getObject` |
  | `NoUniqueBeanDefinitionException` | 类型依赖查找时，IoC 容器存在多 个 Bean 实例 | `BeanFactory#getBean(Class)`                  |
  | `BeanInstantiationException `     | 当 Bean 所对应的类型非具体类时              | `BeanFactory#getBean`                         |
  | `BeanCreationException`           | 当 Bean 初始化过程中                        | Bean 初始化方法执行异常 时                    |
  | `BeanDefinitionStoreException `   | 当 BeanDefinition 配置元信息非法 时         | XML 配置资源无法打开时                        |

  
