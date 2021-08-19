---
title: Spring-事务
date: 2021-08-19 13:10:35
cover: /img/cover/Spring.jpg
tags:
- 事务
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

* [Spring 事务失效的 8 大场景，看看你都遇到过几个？](https://mp.weixin.qq.com/s/W5uyWZ33CqL2SBgIMr5vdg)

#### Spring的@Transactional注解控制事务哪些场景下不生效？

##### 数据库引擎是否支持事务（Mysql 的 MyIsam引擎不支持事务）

##### 注解所在的类是否被加载为 Bean（是否被spring 管理）

##### 注解所在的方法是否为 public 修饰的

> Method visibility and `@Transactional`When you use transactional proxies with Spring’s standard configuration, **you should apply the `@Transactional` annotation only to methods with `public` visibility.** If you do annotate `protected`, `private`, or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. If you need to annotate non-public methods, consider the tip in the following paragraph for class-based proxies or consider using AspectJ compile-time or load-time weaving (described later).When using `@EnableTransactionManagement` in a `@Configuration` class, `protected` or package-visible methods can also be made transactional for class-based proxies by registering a custom `transactionAttributeSource` bean like in the following example. Note, however, that transactional methods in interface-based proxies must always be `public` and defined in the proxied interface.
>
> ```
> /**
>  * Register a custom AnnotationTransactionAttributeSource with the
>  * publicMethodsOnly flag set to false to enable support for
>  * protected and package-private @Transactional methods in
>  * class-based proxies.
>  *
>  * @see ProxyTransactionManagementConfiguration#transactionAttributeSource()
>  */
> @Bean
> TransactionAttributeSource transactionAttributeSource() {
>     return new AnnotationTransactionAttributeSource(false);
> }
> ```
>
> The *Spring TestContext Framework* supports non-private `@Transactional` test methods by default. See [Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx) in the testing chapter for examples.

##### 是否存在自身调用的问题

```
//示例1
 
@Service
public class OrderServiceImpl implements OrderService {
 
    public void update(Order order) {
        updateOrder(order);
    }
 
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
 
}
```

```
//示例2
 
@Service
public class OrderServiceImpl implements OrderService {
 
    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
 
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // update order
    }
 
}
```

* 事务都会失效，因为它们发生了自身调用，就调该类自己的方法，而没有经过 Spring 的代理类，默认只有在外部调用事务才会生效，这也是老生常谈的经典问题了。

##### 所用数据源是否加载了事务管理器

##### @Transactional的扩展配置propagation是否正确

```java
@Service
public class OrderServiceImpl implements OrderService {
 
    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
 
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // update order
    }
 
}
```

##### 异常被吃了

* 这个也是出现比较多的场景：把异常吃了，然后又不抛出来，事务也不会回滚！

  ```
  @Service
  public class OrderServiceImpl implements OrderService {
   
      @Transactional
      public void updateOrder(Order order) {
          try {
              // update order
          } catch {
   
          }
      }
   
  }
  ```

##### 异常类型错误

```
@Service
public class OrderServiceImpl implements OrderService {
 
    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }
 
}
```

* 这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：`@Transactional(rollbackFor = Exception.class)`这个配置仅限于 Throwable 异常类及其子类。
