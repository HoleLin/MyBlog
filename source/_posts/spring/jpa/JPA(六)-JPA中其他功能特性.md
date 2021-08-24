---
title: JPA(六)-JPA中其他功能特性
date: 2021-08-24 16:04:54
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 乐观锁
- 重试机制
- WebMVC
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
#### 参考文献

* 拉钩教育--Spring Data JPA原理与实战

#### JPA中的乐观锁

* 乐观锁在实际开发过程中很常用，它没有加锁、没有阻塞，在多线程环境以及高并发的情况下 CPU 的利用率是最高的，吞吐量也是最大的。

* 而 Java Persistence API 协议也对乐观锁的操作做了规定：**通过指定 @Version 字段对数据增加版本号控制，进而在更新的时候判断版本号是否有变化。如果没有变化就直接更新；如果有变化，就会更新失败并抛出“OptimisticLockException”异常**。我们用 SQL 表示一下乐观锁的做法，代码如下：

  ```mysql
  select uid,name,version from user where id=1;
  update user set name='jack', version=version+1 where id=1 and version=1
  ```

#### 乐观锁的实现方法

* JPA 协议规定，想要实现乐观锁可以通过 @Version 注解标注在某个字段上面，并且可以持久化到 DB 即可。其支持的类型有如下四种：
  * ` int`or`Integer`
  * ` short`or`Short`
  * ` long`or`Long`
  * ` java.sql.Timestamp`
  * 注意：Spring Data JPA 里面有两个 @Version 注解，请使用 **`@javax.persistence.Version`**，而不是 `@org.springframework.data.annotation.Version`。
* 注意：乐观锁异常不仅仅是同一个方法多线程才会出现的问题，我们只是为了方便测试而采用同一个方法；不同的方法、不同的项目，都有可能导致乐观锁异常。乐观锁的本质是 SQL 层面发生的，和使用的框架、技术没有关系。

#### [Spring 支持的重试机制](https://github.com/spring-projects/spring-retry)

* Spring 全家桶里面提供了@Retryable 的注解，会帮我们进行重试。下面看一个 @Retryable 的例子。

  ```
  org.springframework.retry:spring-retry
  ```

* 第二步：在 UserInfoserviceImpl 的方法中添加 @Retryable 注解，就可以实现重试的机制了

* 第三步：新增一个RetryConfiguration并添加@EnableRetry 注解，是为了开启重试机制，使 @Retryable 生效。

  ```
  @EnableRetry
  @Configuration
  public class RetryConfiguration {
  }
  ```

  * maxAttempts：最大重试次数，默认为 3，如果要设置的重试次数为 3，可以不写；
  * value：抛出指定异常才会重试；
  * include：和 value 一样，默认为空，当 exclude 也为空时，默认异常；
  * exclude：指定不处理的异常；
  * backoff：重试等待策略，默认使用 @Backoff@Backoff 的 value，默认为 1s

    * value=delay：隔多少毫秒后重试，默认为 1000L，单位是毫秒；
    * multiplier（指定延迟倍数）默认为 0，表示固定暂停 1 秒后进行重试，如果把 multiplier 设置为 1.5，则第一次重试为 2 秒，第二次为 3 秒，第三次为 4.5 秒。

  ```java
  @Service 
  public interface MyService { 
    @Retryable( value = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
              backoff = @Backoff(delayExpression = "${retry.maxDelay}")) 
    void retryServiceWithExternalizedConfiguration(String sql) throws SQLException; 
  }
  // @Retryable(value = ObjectOptimisticLockingFailureException.class,backoff = @Backoff(multiplier = 1.5,random = true))
  ```

  ```java
  public enum LockModeType
  {
      //等同于OPTIMISTIC，默认，用来兼容2.0之前的协议
      READ,
      //等同于OPTIMISTIC_FORCE_INCREMENT，用来兼容2.0之前的协议
      WRITE,
      //乐观锁，默认，2.0协议新增
      OPTIMISTIC,
      //乐观写锁，强制version加1，2.0协议新增
      OPTIMISTIC_FORCE_INCREMENT,
      //悲观读锁 2.0协议新增
      PESSIMISTIC_READ,
      //悲观写锁，version不变，2.0协议新增
      PESSIMISTIC_WRITE,
      //悲观写锁，version会新增，2.0协议新增
      PESSIMISTIC_FORCE_INCREMENT,
      //2.0协议新增无锁状态
      NONE
  }
  
  public interface UserInfoRepository extends JpaRepository<UserInfo, Long> {
      @Lock(LockModeType.PESSIMISTIC_WRITE)
      Optional<UserInfo> findById(Long userId);
  }
  ```

#### JPA 对 Web MVC 

* 支持在 Controller 层直接返回实体，而不使用其显式的调用方法；
* 对 MVC 层支持标准的分页和排序功能；
* 扩展的插件支持 Querydsl，可以实现一些通用的查询逻辑。

##### 开启支持

```java
@Configuration
@EnableWebMvc
//开启支持Spring Data Web的支持
@EnableSpringDataWebSupport
public class WebConfiguration { }
```

##### 组件

* DomainClassConverter 组件

  * 这个组件的主要作用是帮我们把 Path 中 ID 的变量，或 Request 参数中的变量 ID 的参数值，直接转化成实体对象注册到 Controller 方法的参数里面

* @DynamicUpdate & @DynamicInsert 详解

  * @DynamicInsert：这个注解表示 insert 的时候，会动态生产 insert SQL 语句，其生成 SQL 的规则是：只有非空的字段才能生成 SQL。

    * 这个注解主要是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 insert SQL 的 Columns 只包含非空的字段；如果实体中不加这个注解，默认的情况是空的，字段也会作为 insert 语句里面的 Columns。

    ```java
    @Target( TYPE )
    @Retention( RUNTIME )
    public @interface DynamicInsert {
       //默认是true，如果设置成false，就表示空的字段也会生成sql语句；
       boolean value() default true;
    }
    ```

  * @DynamicUpdate：和 insert 是一个意思，只不过这个注解指的是在 update 的时候，会动态产生 update SQL 语句，生成 SQL 的规则是：只有非空的字段才会生成到 update SQL 的 Columns 里面.

    * 和上一个注解的原理类似，这个注解也是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 update SQL 的 Columns 只包含非空的字段；如果不加这个注解，默认的情况是空的字段也会作为 update 语句里面的 Columns。

    ```java
    @Target( TYPE )
    @Retention( RUNTIME )
    public @interface DynamicUpdate {
       //和insert里面一个意思，默认true;
       boolean value() default true;
    }
    ```

* HandlerMethodArgumentResolvers 详解

  * HandlerMethodArgumentResolvers 在 Spring MVC 中的主要作用是对 Controller 里面的方法参数做解析，即可以把 Request 里面的值映射到方法的参数中。

    ```java
    public interface HandlerMethodArgumentResolver {
       //检查方法的参数是否支持处理和转化
       boolean supportsParameter(MethodParameter parameter);
       //根据reqest上下文，解析方法的参数
       Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
             NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
    }
    ```

  * 使用步骤

    ```java
    第一步：新建 MyPageableHandlerMethodArgumentResolver。
    这个类的作用有两个：
        用来兼容 ?page[size]=2&page[number]=0 的参数情况；
        支持 JPA 新的参数形式 ?size=2&page=0。
        
    /**
     * 通过@Component把此类加载到Spring的容器里面去 
     */
    @Component
    public class MyPageableHandlerMethodArgumentResolver extends PageableHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
       //我们假设sort的参数没有发生变化，采用PageableHandlerMethodArgumentResolver里面的写法
       private static final SortHandlerMethodArgumentResolver DEFAULT_SORT_RESOLVER = new SortHandlerMethodArgumentResolver();
       //给定两个默认值
       private static final Integer DEFAULT_PAGE = 0;
       private static final Integer DEFAULT_SIZE = 10;
       //兼容新版，引入JPA的分页参数
       private static final String JPA_PAGE_PARAMETER = "page";
       private static final String JPA_SIZE_PARAMETER = "size";
       //兼容原来老的分页参数
       private static final String DEFAULT_PAGE_PARAMETER = "page[number]";
       private static final String DEFAULT_SIZE_PARAMETER = "page[size]";
       private SortArgumentResolver sortResolver;
       //模仿PageableHandlerMethodArgumentResolver里面的构造方法
       public MyPageableHandlerMethodArgumentResolver(@Nullable SortArgumentResolver sortResolver) {
          this.sortResolver = sortResolver == null ? DEFAULT_SORT_RESOLVER : sortResolver;
       }
       
       @Override
       public boolean supportsParameter(MethodParameter parameter) {
    //    假设用我们自己的类MyPageRequest接收参数
          return MyPageRequest.class.equals(parameter.getParameterType());
          //同时我们也可以支持通过Spring Data JPA里面的Pageable参数进行接收，两种效果是一样的
    //    return Pageable.class.equals(parameter.getParameterType());
       }
       /**
        * 参数封装逻辑page和sort，JPA参数的优先级高于page[number]和page[size]参数
        */
        //public Pageable resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) { //这种是Pageable的方式
       @Override
       public MyPageRequest resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {
          String jpaPageString = webRequest.getParameter(JPA_PAGE_PARAMETER);
          String jpaSizeString = webRequest.getParameter(JPA_SIZE_PARAMETER);
          //我们分别取参数里面page、sort和 page[number]、page[size]的值
          String pageString = webRequest.getParameter(DEFAULT_PAGE_PARAMETER);
          String sizeString = webRequest.getParameter(DEFAULT_SIZE_PARAMETER);
          //当两个都有值时候的优先级，及其默认值的逻辑
          Integer page = jpaPageString != null ? Integer.valueOf(jpaPageString) : pageString != null ? Integer.valueOf(pageString) : DEFAULT_PAGE;
          //在这里同时可以计算 page+1的逻辑;如：page=page+1;
          Integer size = jpaSizeString != null ? Integer.valueOf(jpaSizeString) : sizeString != null ? Integer.valueOf(sizeString) : DEFAULT_SIZE;
           //我们假设，sort排序的取值方法先不发生改变
          Sort sort = sortResolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
    //    如果使用Pageable参数接收值，我们也可以不用自定义MyPageRequest对象，直接返回PageRequest;
    //    return PageRequest.of(page,size,sort);
          //将page和size计算出来的记过封装到我们自定义的MyPageRequest类里面去
          MyPageRequest myPageRequest = new MyPageRequest(page, size,sort);
          //返回controller里面的参数需要的对象；
          return myPageRequest;
       }
    }
    
    /**
     * 继承父类，可以省掉很多计算page和index的逻辑
     */
    public class MyPageRequest extends PageRequest {
       protected MyPageRequest(int page, int size, Sort sort) {
          super(page, size, sort);
       }
    }
    第三步：implements WebMvcConfigurer 加载 myPageableHandlerMethodArgumentResolver。
    
    /**
     * 实现WebMvcConfigurer
     */
    @Configuration
    public class MyWebMvcConfigurer implements WebMvcConfigurer {
       @Autowired
       private MyPageableHandlerMethodArgumentResolver myPageableHandlerMethodArgumentResolver;
       /**
        * 覆盖这个方法，把我们自定义的myPageableHandlerMethodArgumentResolver加载到原始的mvc的resolvers里面去
        * @param resolvers
        */
       @Override
       public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
          resolvers.add(myPageableHandlerMethodArgumentResolver);
       }
    }
    第四步：我们看下 Controller 里面的写法。
    //用Pageable这种方式也是可以的
    @GetMapping("/users")
    public Page<UserInfo> queryByPage(Pageable pageable, UserInfo userInfo) {
       return userInfoRepository.findAll(Example.of(userInfo),pageable);
    }
    //用MyPageRequest进行接收
    @GetMapping("/users/mypage")
    public Page<UserInfo> queryByMyPage(MyPageRequest pageable, UserInfo userInfo) {
       return userInfoRepository.findAll(Example.of(userInfo),pageable);
    }
    ```

  * 实操

    ```java
    在实际的工作中，还经常会遇到“取当前用户”的应用场景。此时，普通做法是，当使用到当前用户的 UserInfo 时，每次都需要根据请求 header 的 token 取到用户信息，伪代码如下所示：
    @PostMapping("user/info")
    public UserInfo getUserInfo(@RequestHeader String token) {
        // 伪代码
        Long userId = redisTemplate.get(token);
        UserInfo useInfo = userInfoRepository.getById(userId);
        return userInfo;
    }
    
    如果我们使用HandlerMethodArgumentResolver接口来实现，代码就会变得优雅许多。伪代码如下：
    // 1. 实现HandlerMethodArgumentResolver接口
    @Component
    public class UserInfoArgumentResolver implements HandlerMethodArgumentResolver {
       private final RedisTemplate redisTemplate;//伪代码，假设我们token是放在redis里面的
       private final UserInfoRepository userInfoRepository;
       public UserInfoArgumentResolver(RedisTemplate redisTemplate, UserInfoRepository userInfoRepository) {
          this.redisTemplate = redisTemplate;//伪代码，假设我们token是放在redis里面的
          this.userInfoRepository = userInfoRepository;
       }
       @Override
       public boolean supportsParameter(MethodParameter parameter) {
          return UserInfo.class.isAssignableFrom(parameter.getParameterType());
       }
       @Override
       public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
          HttpServletRequest nativeRequest = (HttpServletRequest) webRequest.getNativeRequest();
          String token = nativeRequest.getHeader("token");
          Long userId = (Long) redisTemplate.opsForValue().get(token);//伪代码，假设我们token是放在redis里面的
          UserInfo useInfo = userInfoRepository.getOne(userId);
          return useInfo;
       }
    }
    //2. 我们只需要在MyWebMvcConfigurer里面把userInfoArgumentResolver添加进去即可，关键代码如下：
    @Configuration
    public class MyWebMvcConfigurer implements WebMvcConfigurer {
       @Autowired
       private MyPageableHandlerMethodArgumentResolver myPageableHandlerMethodArgumentResolver;
    @Autowired
    private UserInfoArgumentResolver userInfoArgumentResolver;
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
       resolvers.add(myPageableHandlerMethodArgumentResolver);
       //我们只需要把userInfoArgumentResolver加入resolvers中即可
       resolvers.add(userInfoArgumentResolver);
    }
    }
    // 3. 在Controller中使用
    @RestController
    public class UserInfoController {
      //获得当前用户的信息
      @GetMapping("user/info")
      public UserInfo getUserInfo(UserInfo userInfo) {
         return userInfo;
      }
      //给当前用户 say hello
      @PostMapping("sayHello")
      public String sayHello(UserInfo userInfo) {
        return "hello " + userInfo.getTelephone();
      }
    }
    ```

* WebMvcConfigurer 介绍

  ````java
  /* 拦截器配置 */
  void addInterceptors(InterceptorRegistry var1);
  /* 视图跳转控制器 */
  void addViewControllers(ViewControllerRegistry registry);
  /**
    *静态资源处理
  **/
  void addResourceHandlers(ResourceHandlerRegistry registry);
  /* 默认静态资源处理器 */
  void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer);
  /**
    *这里配置视图解析器
   **/
  void configureViewResolvers(ViewResolverRegistry registry);
  /* 配置内容裁决的一些选项*/
  void configureContentNegotiation(ContentNegotiationConfigurer configurer);
  /** 解决跨域问题 **/
  void addCorsMappings(CorsRegistry registry) ;
  /** 添加都会contoller的Return的结果的处理 **/
  void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers)；
  ````

  * 用 Result 对 JSON 的返回结果进行统一封装

    ```java
    第一步：我们自定义一个注解 @WarpWithData，表示此注解包装的返回结果用 Data 进行包装，代码如下：
    @Target({ElementType.TYPE, ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    /**
     * 自定义一个注解对返回结果进行包装
     */
    public @interface WarpWithData {
    }
    
    第二步：自定义 MyWarpWithDataHandlerMethodReturnValueHandler，并继承 RequestResponseBodyMethodProcessor 来实现 HandlerMethodReturnValueHandler 接口，用来处理 Data 包装的结果，代码如下：
    //自定义自己的return的处理类，我们直接继承RequestResponseBodyMethodProcessor，这样父类里面的方法我们直接使用就可以了
    @Component
    public class MyWarpWithDataHandlerMethodReturnValueHandler extends RequestResponseBodyMethodProcessor implements HandlerMethodReturnValueHandler {
       //参考父类RequestResponseBodyMethodProcessor的做法
       @Autowired
       public MyWarpWithDataHandlerMethodReturnValueHandler(List<HttpMessageConverter<?>> converters) {
          super(converters);
       }
       //只处理需要包装的注解的方法
       @Override
       public boolean supportsReturnType(MethodParameter returnType) {
          return returnType.hasMethodAnnotation(WarpWithData.class);
       }
       //将返回结果包装一层Data
       @Override
       public void handleReturnValue(Object returnValue, MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest) throws IOException, HttpMediaTypeNotAcceptableException {
          Map<String,Object> res = new HashMap<>();
          res.put("data",returnValue);
          super.handleReturnValue(res,methodParameter,modelAndViewContainer,nativeWebRequest);
       }
    }
    
    第三步：在 MyWebMvcConfigurer 里面直接把 myWarpWithDataHandlerMethodReturnValueHandler 加入 handlers 里面即可，也是通过覆盖父类 WebMvcConfigurer 里面的 addReturnValueHandlers 方法完成的，关键代码如下：
    @Configuration
    public class MyWebMvcConfigurer implements WebMvcConfigurer {
       @Autowired
       private MyWarpWithDataHandlerMethodReturnValueHandler myWarpWithDataHandlerMethodReturnValueHandler;
       //把我们自定义的myWarpWithDataHandlerMethodReturnValueHandler加入handlers里面即可
       @Override
       public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
          handlers.add(myWarpWithDataHandlerMethodReturnValueHandler);
       }
       
      @Autowired
      private RequestMappingHandlerAdapter requestMappingHandlerAdapter;
      //由于HandlerMethodReturnValueHandler处理的优先级问题，我们通过如下方法，把我们自定义的myWarpWithDataHandlerMethodReturnValueHandler放到第一个；
      @PostConstruct
      public void init() {
         List<HandlerMethodReturnValueHandler> returnValueHandlers = Lists.newArrayList(myWarpWithDataHandlerMethodReturnValueHandler);
    //取出原始列表，重新覆盖进去；
            returnValueHandlers.addAll(requestMappingHandlerAdapter.getReturnValueHandlers());
         requestMappingHandlerAdapter.setReturnValueHandlers(returnValueHandlers);
      }
    }
    ```

#### 微服务下的实战建议

* 微服务的大环境下，服务越小，内聚越高，低耦合服务越健壮，所以一般跨库之间一定是是通过 REST 的 API 协议，进行内部服务之间的调用，这是最稳妥的方式，原因有如下几点：
* REST 的 API 协议更容易监控，更容易实现事务的原子性；
* db 之间解耦，使业务领域代码职责更清晰，更容易各自处理各种问题；
* 只读和读写的 API 更容易分离和管理。

#### Spring里面事务的配置方法

* Spring Boot会通过 TransactionAutoConfiguration.java 加载 @EnableTransactionManagement 注解帮我们默认开启事务;

* 默认 @Transactional 注解式事务

  ```java
  @Target({ElementType.METHOD, ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Inherited
  @Documented
  public @interface Transactional {
     @AliasFor("transactionManager")
     String value() default "";
     @AliasFor("value")
     String transactionManager() default "";
     Propagation propagation() default Propagation.REQUIRED;
     Isolation isolation() default Isolation.DEFAULT;
     int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
     boolean readOnly() default false;
     Class<? extends Throwable>[] rollbackFor() default {};
     String[] rollbackForClassName() default {};
     Class<? extends Throwable>[] noRollbackFor() default {};
     String[] noRollbackForClassName() default {};
  }
  ```

  ![img](https://www.chenjunlin.vip/img/spring/jpa/@Transactional.png)

##### 隔离级别

* propagation：代表的是事务的传播机制，这个是 Spring 事务的核心业务逻辑，是 Spring 框架独有的，它和 MySQL 数据库没有一点关系。所谓事务的传播行为是指在同一线程中，在开始当前事务之前，需要判断一下当前线程中是否有另外一个事务存在，如果存在，提供了七个选项来指定当前事务的发生行为。我们可以看 org.springframework.transaction.annotation.Propagation 这类的枚举值来确定有哪些传播行为。7 个表示传播行为的枚举值如下所示。

  ```java
  public enum Propagation {
  	// REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这个值是默认的。
  	REQUIRED(0),
  	// SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
  	SUPPORTS(1),
  	// MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
  	MANDATORY(2),
  	// REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
  	REQUIRES_NEW(3),
  	// NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
  	NOT_SUPPORTED(4),
  	// NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
  	NEVER(5),
  	// NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 REQUIRED。
  	NESTED(6);
  }
  ```

##### @Transactional 的局限性

* 一个当前对象调用对象自己里面的方法不起作用的场景

  ```java
  @Component
  public class UserInfoServiceImpl implements UserInfoService {
     @Autowired
     private UserInfoRepository userInfoRepository;
     /**
      * 根据UserId产生的一些业务计算逻辑
      */
     @Override
     @Transactional(transactionManager = "db2TransactionManager")
     public UserInfo calculate(Long userId) {
        UserInfo userInfo = userInfoRepository.findById(userId).get();
        userInfo.setAges(userInfo.getAges()+1);
        //.....等等一些复杂事务内的操作
        userInfo.setTelephone(Instant.now().toString());
        return userInfoRepository.saveAndFlush(userInfo);
     }
     /**
      * 此方法调用自身对象的方法，就会发现calculate方法上面的事务是失效的
      */
     public UserInfo save(Long userId) {
        return this.calculate(userId);
     }
  }
  ```

* 解决方法: 

  * 方法一: 可以引入一个类` TransactionTemplate`

    ```java
    public UserInfo save(Long userId) {
       return transactionTemplate.execute(status -> this.calculate(userId));
    }
    ```

  * 方法二; 自定义 `TransactionHelper`

    * 第一步：新建一个 `TransactionHelper` 类，进行事务管理，代码如下

      ```java
      /**
       * 利用spring进行管理
       */
      @Component
      public class TransactionHelper {
          /**
           * 利用spring 的机制和jdk8的function机制实现事务
           */
          @Transactional(rollbackFor = Exception.class) //可以根据实际业务情况，指定明确的回滚异常
          public <T, R> R transactional(Function<T, R> function, T t) {
              return function.apply(t);
          }
      }
      ```

    * 第二步：直接在 service 中就可以使用了，代码如下

      ```java
      @Autowired
      private TransactionHelper transactionHelper;
      /**
      * 调用外部的transactionHelper类，利用transactionHelper方法上面的@Transaction注解使事务生效
      */
      public UserInfo save(Long userId) {
         return transactionHelper.transactional((uid)->this.calculate(uid),userId);
      }
      ```

##### 隐式事务 / AspectJ 事务配置

```java
@Configuration
@EnableTransactionManagement
public class AspectjTransactionConfig {
   public static final String transactionExecution = "execution (* com.example..service.*.*(..))";//指定拦截器作用的包路径
   @Autowired
   private PlatformTransactionManager transactionManager;
   @Bean
   public DefaultPointcutAdvisor defaultPointcutAdvisor() {
      //指定一般要拦截哪些类
      AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
      pointcut.setExpression(transactionExecution);
      //配置advisor
      DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor();
      advisor.setPointcut(pointcut);
      //根据正则表达式，指定上面的包路径里面的方法的事务策略
      Properties attributes = new Properties();
      attributes.setProperty("get*", "PROPAGATION_REQUIRED,-Exception");
      attributes.setProperty("add*", "PROPAGATION_REQUIRED,-Exception");
      attributes.setProperty("save*", "PROPAGATION_REQUIRED,-Exception");
      attributes.setProperty("update*", "PROPAGATION_REQUIRED,-Exception");
      attributes.setProperty("delete*", "PROPAGATION_REQUIRED,-Exception");
      //创建Interceptor
      TransactionInterceptor txAdvice = new TransactionInterceptor(transactionManager, attributes);
      advisor.setAdvice(txAdvice);
      return advisor;
   }
}
```

##### 通过日志分析配置方法的过程

* 第一步，在数据连接中加上 logger=Slf4JLogger&profileSQL=true，用来显示 MySQL 执行的 SQL 日志

* 第二步，打开 Spring 的事务处理日志，用来观察事务的执行过程.

  ```properties
  # Log Transactions Details
  logging.level.org.springframework.orm.jpa=DEBUG
  logging.level.org.springframework.transaction=TRACE
  logging.level.org.hibernate.engine.transaction.internal.TransactionImpl=DEBUG
  # 监控连接的情况
  logging.level.org.hibernate.resource.jdbc=trace
  logging.level.com.zaxxer.hikari=DEBUG
  ```

* 第三步，执行一个 saveOrUpdate 的操作，详细的执行日志

#### Spring Cache 结合 Redis 使用的最佳实践

* 不同 cache 的 name 在 redis 里面配置不同的过期时间

* 默认情况下所有 redis 的 cache 过期时间是一样的，实际工作中一般需要自定义不同 cache 的 name 的过期时间，我们这里 cache 的 name 就是指 @Cacheable 里面 value 属性对应的值。主要步骤如下

  ```java
  第一步：自定义一个配置文件，用来指定不同的 cacheName 对应的过期时间不一样。代码如下所示。
  @Getter
  @Setter
  @ConfigurationProperties(prefix = "spring.cache.redis")
  /**
   * 改善一下cacheName的最佳实践方法，目前主要用不同的cache name不同的过期时间，可以扩展
   */
  public class MyCacheProperties {
      private HashMap<String, Duration> cacheNameConfig;
  }
  
  第二步：通过自定义类 MyRedisCacheManagerBuilderCustomizer 实现 RedisCacheManagerBuilderCustomizer 里面的 customize 方法，用来指定不同的 name 采用不同的 RedisCacheConfiguration，从而达到设置不同的过期时间的效果。代码如下所示。
  /**
   * 这个依赖spring boot 2.2 以上版本才有效
   */
  public class MyRedisCacheManagerBuilderCustomizer implements RedisCacheManagerBuilderCustomizer {
      private MyCacheProperties myCacheProperties;
      private RedisCacheConfiguration redisCacheConfiguration;
      public MyRedisCacheManagerBuilderCustomizer(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration) {
          this.myCacheProperties = myCacheProperties;
          this.redisCacheConfiguration = redisCacheConfiguration;
      }
      /**
       * 利用默认配置的只需要在这里加就可以了
       * spring.cache.cache-names=abc,def,userlist2,user3
       * 下面是不同的cache-name可以配置不同的过期时间，yaml也支持，如果以后还有其他属性扩展可以改这里
       * spring.cache.redis.cache-name-config.user2=2h
       * spring.cache.redis.cache-name-config.def=2m
       * @param builder
       */
      @Override
      public void customize(RedisCacheManager.RedisCacheManagerBuilder builder) {
          if (ObjectUtils.isEmpty(myCacheProperties.getCacheNameConfig())) {
              return;
          }
          Map<String, RedisCacheConfiguration> cacheConfigurations = myCacheProperties.getCacheNameConfig().entrySet().stream()
                  .collect(Collectors
                          .toMap(e->e.getKey(),v->builder
                                  .getCacheConfigurationFor(v.getKey())
                                  .orElse(RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(redisCacheConfiguration.getValueSerializationPair()))
                                  .entryTtl(v.getValue())));
          builder.withInitialCacheConfigurations(cacheConfigurations);
      }
  }
  
  第三步：在 CacheConfiguation 里面把我们自定义的 CacheManagerCustomize 加载进去即可，代码如下。
  
  @EnableCaching
  @Configuration
  @EnableConfigurationProperties(value = {MyCacheProperties.class,CacheProperties.class})
  @AutoConfigureAfter({CacheAutoConfiguration.class})
  public class CacheConfiguration {
      /**
       * 支持不同的cache name有不同的缓存时间的配置
       *
       * @param myCacheProperties
       * @param redisCacheConfiguration
       * @return
       */
      @Bean
      @ConditionalOnMissingBean(name = "myRedisCacheManagerBuilderCustomizer")
      @ConditionalOnClass(RedisCacheManagerBuilderCustomizer.class)
      public MyRedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration) {
          return new MyRedisCacheManagerBuilderCustomizer(myCacheProperties,redisCacheConfiguration);
      }
  }
  
  第四步：使用的时候非常简单，只需要在 application.properties 里面做如下配置即可。
  # 设置默认的过期时间是20分钟
  spring.cache.redis.time-to-live=20m
  # 设置 @Cacheable(value="userInfo")5分钟过期
  spring.cache.redis.cache-name-config.userInfo=5m
  # 设置 room的cache1小时过期
  spring.cache.redis.cache-name-config.room=1h
  ```

* 自定义 KeyGenerator 实现，redis 的 key 自定义拼接规则

  ```java
  @Component
  @Log4j2
  public class MyRedisCachingConfigurerSupport extends CachingConfigurerSupport {
      @Override
      public KeyGenerator keyGenerator() {
          return getKeyGenerator();
      }
      /**
       * 覆盖默认的redis key的生成规则，变成"方法名:参数:参数"
       * @return
       */
      public static KeyGenerator getKeyGenerator() {
          return (target, method, params) -> {
              StringBuilder key = new StringBuilder();
              key.append(ClassUtils.getQualifiedMethodName(method));
              for (Object obc : params) {
                  key.append(":").append(obc);
              }
              return key.toString();
          };
      }
      /**
       * 覆盖默认异常处理方法，不抛异常，改打印error日志
       *
       * @return
       */
      @Override
      public CacheErrorHandler errorHandler() {
          return new CacheErrorHandler() {
              @Override
              public void handleCacheGetError(RuntimeException exception, Cache cache, Object key) {
                  log.error(String.format("Spring cache GET error:cache=%s,key=%s", cache, key), exception);
              }
              @Override
              public void handleCachePutError(RuntimeException exception, Cache cache, Object key, Object value) {
                  log.error(String.format("Spring cache PUT error:cache=%s,key=%s", cache, key), exception);
              }
              @Override
              public void handleCacheEvictError(RuntimeException exception, Cache cache, Object key) {
                  log.error(String.format("Spring cache EVICT error:cache=%s,key=%s", cache, key), exception);
              }
              @Override
              public void handleCacheClearError(RuntimeException exception, Cache cache) {
                  log.error(String.format("Spring cache CLEAR error:cache=%s", cache), exception);
              }
          };
      }
  }    
  ```

* 改变默认的 cache 里面 redis 的 value 序列化方式

  ```java
  @EnableCaching
  @Configuration
  @EnableConfigurationProperties(value = {MyCacheProperties.class,CacheProperties.class})
  @AutoConfigureAfter({CacheAutoConfiguration.class})
  public class CacheConfiguration {
      /**
       * 支持不同的cache name有不同的缓存时间的配置
       *
       * @param myCacheProperties
       * @param redisCacheConfiguration
       * @return
       */
      @Bean
      @ConditionalOnMissingBean(name = "myRedisCacheManagerBuilderCustomizer")
      @ConditionalOnClass(RedisCacheManagerBuilderCustomizer.class)
      public MyRedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration) {
          return new MyRedisCacheManagerBuilderCustomizer(myCacheProperties,redisCacheConfiguration);
      }
      /**
       * cache异常不抛异常，只打印error日志
       *
       * @return
       */
      @Bean
      @ConditionalOnMissingBean(name = "myRedisCachingConfigurerSupport")
      public MyRedisCachingConfigurerSupport myRedisCachingConfigurerSupport() {
          return new MyRedisCachingConfigurerSupport();
      }
      /**
       * 依赖默认的ObjectMapper，实现普通的json序列化
       * @param defaultObjectMapper
       * @return
       */
      @Bean(name = "genericJackson2JsonRedisSerializer")
      @ConditionalOnMissingBean(name = "genericJackson2JsonRedisSerializer")
      public GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer(ObjectMapper defaultObjectMapper) {
          ObjectMapper objectMapper = defaultObjectMapper.copy();
          objectMapper.registerModule(new Hibernate5Module().enable(REPLACE_PERSISTENT_COLLECTIONS)); //支持JPA的实体的json的序列化
          objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY, true);//培训
          objectMapper.deactivateDefaultTyping(); //关闭 defaultType，不需要关心reids里面是否为对象的类型
          return new GenericJackson2JsonRedisSerializer(objectMapper);
      }
      /**
       * 覆盖 RedisCacheConfiguration，只是修改serializeValues with jackson
       *
       * @param cacheProperties
       * @return
       */
      @Bean
      @ConditionalOnMissingBean(name = "jacksonRedisCacheConfiguration")
      public RedisCacheConfiguration jacksonRedisCacheConfiguration(CacheProperties cacheProperties,
                                                                    GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer) {
          CacheProperties.Redis redisProperties = cacheProperties.getRedis();
          RedisCacheConfiguration config = RedisCacheConfiguration
                  .defaultCacheConfig();
          config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(genericJackson2JsonRedisSerializer));//修改的关键所在，指定Jackson2JsonRedisSerializer的方式
          if (redisProperties.getTimeToLive() != null) {
              config = config.entryTtl(redisProperties.getTimeToLive());
          }
          if (redisProperties.getKeyPrefix() != null) {
              config = config.prefixCacheNameWith(redisProperties.getKeyPrefix());
          }
          if (!redisProperties.isCacheNullValues()) {
              config = config.disableCachingNullValues();
          }
          if (!redisProperties.isUseKeyPrefix()) {
              config = config.disableKeyPrefix();
          }
          return config;
      }
  }
  ```

#### Spring Data JPA 单元测试的最佳实践

```java
第一步：引入 test 的依赖 org.springframework.boot:spring-boot-starter-test

第二步：利用项目里面的实体和 Repository，假设我们项目里面有 Address 和 AddressRepository，代码如下所示。
@Entity
@Table
@Data
@SuperBuilder
@AllArgsConstructor
@NoArgsConstructor
public class Address extends BaseEntity {
   private String city;
   private String address;
}
//Repository的DAO层
public interface AddressRepository extends JpaRepository<Address, Long>{
  
}

第三步：新建 RepsitoryTest，@DataJpaTest 即可，代码如下所示。
@DataJpaTest
public class AddressRepositoryTest {
    @Autowired
    private AddressRepository addressRepository;
    //测试一下保存和查询
    @Test
    public  void testSave() {
        Address address = Address.builder().city("shanghai").build();
        addressRepository.save(address);
        List<Address> address1 = addressRepository.findAll();
        address1.stream().forEach(address2 -> System.out.println(address2));
    }
}

Service 层单元测试
@ExtendWith(SpringExtension.class)//通过这个注解利用Spring的容器
@Import(UserInfoServiceImpl.class)//导入要测试的UserInfoServiceImpl
public class UserInfoServiceTest {
    @Autowired //利用spring的容器，导入要测试的UserInfoService
    private UserInfoService userInfoService;
    @MockBean //里面@MockBean模拟我们service中用到的userInfoRepository，这样避免真实请求数据库
    private UserInfoRepository userInfoRepository;
    // 利用单元测试的思想，mock userInfoService里面的UserInfoRepository，这样Service层就不用连接数据库，就可以测试自己的业务逻辑了
    @Test
    public void testGetUserInfoDto() {
//利用Mockito模拟当调用findById(1)的时候，返回模拟数据
                Mockito.when(userInfoRepository.findById(1L)).thenReturn(java.util.Optional.ofNullable(UserInfo.builder().name("jack").id(1L).build()));
        UserInfoDto userInfoDto = userInfoService.findByUserId(1L);
        //经过一些service里面的逻辑计算，我们验证一下返回结果是否正确
        Assertions.assertEquals("jack",userInfoDto.getName());
    }
}

Controller 层单元测试
@WebMvcTest(UserInfoController.class)
public class UserInfoControllerTest {
    @Autowired
    private MockMvc mvc;
    @MockBean
    private UserInfoService userInfoService;
    
    //单元测试mvc的controller的方法
    @Test
    public void testGetUserDto() throws Exception {
        //利用@MockBean，当调用 userInfoService的findByUserId(1)的时候返回一个模拟的UserInfoDto数据
        Mockito.when(userInfoService.findByUserId(1L)).thenReturn(UserInfoDto.builder().name("jack").id(1L).build());
        
        //利用mvc验证一下Controller里面的解决是否OK
        MockHttpServletResponse response = mvc
                .perform(MockMvcRequestBuilders
                        .get("/user/1/")//请求的path
                        .accept(MediaType.APPLICATION_JSON)//请求的mediaType，这里面可以加上各种我们需要的Header
                )
                .andDo(print())//打印一下
                .andExpect(status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("jack"))
                .andReturn().getResponse();
        System.out.println(response);
    }
}

Controller 层的集成测试用例的写法
@SpringBootTest(classes = DemoApplication.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) //加载DemoApplication，指定一个随机端口
public class UserInfoControllerIntegrationTest {
    @LocalServerPort //获得模拟的随机端口
    private int port;
    @Autowired //我们利用RestTemplate，发送一个请求
    private TestRestTemplate restTemplate;
    @Test
    public void testAllUserDtoIntegration() {
        UserInfoDto userInfoDto = this.restTemplate
                .getForObject("http://localhost:" + port + "/user/1", UserInfoDto.class);//真实请求有一个后台的API
        Assertions.assertNotNull(userInfoDto);
    }
}
```

