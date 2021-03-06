---
title: Spring-事务
date: 2021-08-19 13:10:35
cover: /img/cover/Spring.jpg
tags:
- 事务
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

* [Spring 事务失效的 8 大场景，看看你都遇到过几个？](https://mp.weixin.qq.com/s/W5uyWZ33CqL2SBgIMr5vdg)
* [Spring官方推荐的@Transactional还能导致生产事故？原来姿势完全错了](https://mp.weixin.qq.com/s/7F3ohsHr9u-IpNBrV3KQlA)

### Spring事务隔离级别

* Spring提供事5种务隔离级别
  * `@Transaction(isolation=Isolation.DEFAULT)`: 默认的事务隔离级别,即使用数据库的事务隔离级别.
  * `@Transaction(isolation=Isolation.READ_UNCOMMITTED)`: 读未提交,这是最低的事务隔离级别,允许其他事务读取未提交的数据,这种级别的事务隔离会产生脏读,不可重复读和幻读.
  * `@Transaction(isolation=Isolation.READ_COMMITTED)`: 读已提交,这种级别的事务隔离能读取其他事务已修改的数据,不能读取未提交的数据,会产生不可重复读和幻读.
  * `@Transaction(isolation=Isolation.REPEATABLE_READ)`: 可重复读,这种级别的事务隔离可以防止不可重复读和脏读,但是会产生幻读.
  * `@Transaction(isolation=Isolation.SERIALIZABLE)`: 串行化,这是最高级别的事务隔离,会避免脏读,不可重复读和幻读.在这种隔离级别下,事务会按顺序进行.

### Spring事务传播行为

* 事务传播行为是指如果多个事务同事存在,Spring就会处理这些事务的行为.
* 事务传播行为分为7种:
  * `REQUIRED`: 如果当前存在事务,就加入该事务;如果当前没有事务,就创建一个新的事务,这是Spring默认的事务传播行为.
  * `SUPPORTS`: 如果当前存在事务,就加入该事务;如果当前没有事务,就以非事务的方式继续运行.
  * `MANDATORY`: 如果当前事务存在事务,就加入该事务;如果当前事务,就抛出异常.
  * `REQUIRES_NEW`: 创建一个新的事务,如果当前存在事务,就把当前事务挂起.新建事务和被挂起的事务没有任何关系,是两个独立的事务.外层事务回滚失败时,不能回滚内层事务执行结果,外层事务不能相互干扰.
  * `NOT_SUPPORTED`: 以非事务方式运行,如果当前存在事务,就把当前事务挂起.
  * `NEVER`: 以非事务方式运行,如果当前存在事务,就抛出异常.
  * `NESTED`: 如果当前存在事务,就创建一个事务作为当前事务的嵌套事务来运行;如果当前没有事务,该取值就等价于`REQUIRED`

### 声明式事务属性

* `value`: 存放String类型的值,主要用来指定不同的事务管理器,满足在同一个系统中存在不同的事务管理器.
* `transactionManager`: 与`value`类似,也是用来选择事务管理器.
* `propagation`: 事务传播行为,默认为`Propagation.REQUIRED`
* `isolation`: 事务的隔离级别,默认为`Isolation.DEFAULT`
* `timeout`: 事务的超时时间,默认为-1,如果超过了设置的时间还没有执行完成,就会自动回滚当前事务.
* `readOnly`: 当前事务是不是自读事务,默认值是false.通常可以设置读取数据的事务的属性值为true.
* `rollbackFor`: 可以设置触发事务的指定异常,允许指定多个类型的异常.
* `noRollbackFor`: 与`rollbackFor`相反,可以设置不触发事务的指定异常,允许指定多个类型的异常.

### 事务回滚规则

* Spring的事务回滚通常是根据当前事务抛出异常的时候,Spring事务管理器捕捉到未经处理的异常,然后根据规则来决定当前事务是否回滚.
* 若捕获的异常正好是`noRollbackFor`属性的异常,那么将不会被捕获.在默认配置下,Spring只有捕获运行时异常(RuntimeException)的子类才会进行回滚.

### Spring的@Transactional注解控制事务哪些场景下不生效？

#### 数据库引擎是否支持事务（Mysql 的 MyIsam引擎不支持事务）

#### 注解所在的类是否被加载为 Bean（是否被spring 管理）

#### 注解所在的方法是否为 public 修饰的

> Method visibility and `@Transactional`When you use transactional proxies with Spring’s standard configuration, **you should apply the `@Transactional` annotation only to methods with `public` visibility.** If you do annotate `protected`, `private`, or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. If you need to annotate non-public methods, consider the tip in the following paragraph for class-based proxies or consider using AspectJ compile-time or load-time weaving (described later).When using `@EnableTransactionManagement` in a `@Configuration` class, `protected` or package-visible methods can also be made transactional for class-based proxies by registering a custom `transactionAttributeSource` bean like in the following example. Note, however, that transactional methods in interface-based proxies must always be `public` and defined in the proxied interface.
>
> ```java
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

#### 是否存在自身调用的问题

```java
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

```java
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

#### 所用数据源是否加载了事务管理器

#### `@Transactional`的扩展配置`propagation`是否正确

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

#### 异常被catch捕获导致`@Transactional`失效

* 这个也是出现比较多的场景：把异常吃了，然后又不抛出来，事务也不会回滚！

  ```java
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

#### 异常类型错误

```java
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

* 这样事务也是不生效的，因为默认回滚的是：`RuntimeException`，如果你想触发其他异常的回滚，需要在注解上配置一下，如：`@Transactional(rollbackFor = Exception.class)`这个配置仅限于 Throwable 异常类及其子类。

### 使用`@Transactional`注解注意点

* `@Transactional` 注解，是使用 AOP 实现的，本质就是在目标方法执行前后进行拦截。在目标方法执行前加入或创建一个事务，在执行方法执行后，根据实际情况选择提交或是回滚事务。
* 当 Spring 遇到该注解时，会自动从数据库连接池中获取 connection，并开启事务然后绑定到 ThreadLocal 上，对于`@Transactional`注解包裹的整个方法都是**使用同一个connection连接**。如果我们出现了耗时的操作，比如第三方接口调用，业务逻辑复杂，大批量数据处理等就会导致我们我们占用这个connection的时间会很长，数据库连接一直被占用不释放。一旦类似操作过多，就会导致数据库连接池耗尽。

#### 避免长事务

* 长事务: 运行时间比较长，长时间未提交的事务

#### 长事务会引发哪些问题?

* 数据库连接池被占满,应用无法获取连接资源;
* 容易引发数据库死锁;
* 数据库回滚时间长;
* 在主从架构中会导致主从延时变大.

### 如何避免长事务

* 对事务方法进行拆分,尽量爱让事务变小,变快,减小事务的颗粒度.

* Spring进行事务管理的方式:

  * 声明式事务: 在方法上使用`@Transactional`注解进行事务管理的操作叫声明式事务.

    * 优点: 使用简单
    * 缺点: 事务的颗粒度是整个方法,无法进行精细化控制.

  * 编程式事务:基于底层的API，开发者在代码中手动的管理事务的开启、提交、回滚等操作

    * 在Spring项目中可以使用`TransactionTemplate`类的对象，手动控制事务。

      ```java
      @Autowired 
      private TransactionTemplate transactionTemplate; 
      
      public void xxx(TempBean tempBean) { 
          transactionTemplate.execute(transactionStatus -> {
              tempDao.save(tempBean);
              //保存明细表
              tempDetailDao.save(tempBean.getDetail());
              return Boolean.TRUE; 
          });
      } 
      ```

    * 优点: 可以精细化控制事务的范围.

* 避免长事务的最简单的方法就是**不要使用声明式事务`@Transactional`,而是使用编程式事务手动控制事务范围.**
