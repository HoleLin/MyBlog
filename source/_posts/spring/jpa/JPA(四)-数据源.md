---
title: JPA(四)-数据源
date: 2021-08-24 16:18:28
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 数据源
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

#### 数据源

##### 配置[HikariCP](https://github.com/brettwooldridge/HikariCP)

```java
## 最小空闲链接数量
spring.datasource.hikari.minimum-idle=5
## 空闲链接存活最大时间，默认600000（10分钟）
spring.datasource.hikari.idle-timeout=180000
## 链接池最大链接数，默认是10
spring.datasource.hikari.maximum-pool-size=10
## 此属性控制从池返回的链接的默认自动提交行为,默认值：true
spring.datasource.hikari.auto-commit=true
## 数据源链接池的名称
spring.datasource.hikari.pool-name=MyHikariCP
## 此属性控制池中链接的最长生命周期，值0表示无限生命周期，默认1800000即30分钟
spring.datasource.hikari.max-lifetime=1800000
## 数据库链接超时时间,默认30秒，即30000
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.connection-test-query=SELECT 1mysql

Hikari 数据源下的 MySQL 配置最佳实践
##数据源的配置：logger=Slf4JLogger&profileSQL=true是用来debug显示sql的执行日志的
spring.datasource.url=jdbc:mysql://localhost:3306/test?logger=Slf4JLogger&profileSQL=true
spring.datasource.username=root
spring.datasource.password=E6kroWaR9F
##采用默认的
#spring.datasource.hikari.connectionTimeout=30000
#spring.datasource.hikari.idleTimeout=300000
##指定一个链接池的名字，方便我们分析线程问题
spring.datasource.hikari.pool-name=jpa-hikari-pool
##最长生命周期15分钟够了
spring.datasource.hikari.maxLifetime=900000
spring.datasource.hikari.maximumPoolSize=8
##最大和最小相对应减少创建线程池的消耗；
spring.datasource.hikari.minimumIdle=8
spring.datasource.hikari.connectionTestQuery=select 1 from dual
##当释放连接到连接池之后，采用默认的自动提交事务
spring.datasource.hikari.autoCommit=true
##用来显示链接测trace日志
logging.level.com.zaxxer.hikari.HikariConfig=DEBUG 
logging.level.com.zaxxer.hikari=TRACE
```

* 监控
  * [Granfan 图表或者 Prometheus](https://github.com/prometheus-operator/prometheus-operator) 

#### Naming 命名策略详解及其实践

* 第一步：通过ImplicitNamingStrategy先找到实例里面定义的逻辑的字段名字。

  * 这是通过ImplicitNamingStrategy 的实现类指定逻辑字段查找策略，也就是当实体里面定义了 @Table、@Column 注解的时候，以注解指定名字返回；而当没有这些注解的时候，返回的是实体里面的字段的名字。
  * 其中，org.hibernate.boot.model.naming.ImplicitNamingStrategy 是一个接口，ImplicitNamingStrategyJpaCompliantImpl 这个实现类兼容 JPA 2.0 的字段映射规范。除此之外，还有如下四个实现类：
    * ImplicitNamingStrategyLegacyHbmImpl：兼容 Hibernate 老版本中的命名规范；
    * ImplicitNamingStrategyLegacyJpaImpl：兼容 JPA 1.0 规范中的命名规范；
    * ImplicitNamingStrategyComponentPathImpl：@Embedded 等注解标志的组件处理是通过 attributePath 完成的，因此如果我们在使用 @Embedded 注解的时候，如果要指定命名规范，可以直接继承这个类来实现；
    * SpringImplicitNamingStrategy：默认的 spring data 2.2.3 的策略，只是扩展了ImplicitNamingStrategyJpaCompliantImpl 里面的 JoinTableName 的方法

* 第二步：通过 PhysicalNamingStrategy 将逻辑字段转化成数据库的物理字段名字。

  * 它的实现类负责将逻辑字段转化成带下划线，或者统一给字段加上前缀，又或者加上双引号等格式的数据库字段名字，其主要的接口是：org.hibernate.boot.model.naming.PhysicalNamingStrategy，而它的实现类也只有两个.

  * PhysicalNamingStrategyStandardImpl：这个类什么都没干，即直接将第一个步骤得到的逻辑字段名字当成数据库的字段名字使用。这个主要的应用场景是，如果某些字段的命名格式不是下划线的格式，我们想通过 @Column 的方式显示声明的话，可以把默认第二步的策略改成 PhysicalNamingStrategyStandardImpl。

    ```
    userInfo -> userInfo
    id->id
    ages->ages
    lastName -> lastName
    myAddress -> myAddress
    ```

  * SpringPhysicalNamingStrategy：这个类是将第一步得到的逻辑字段名字的大写字母前面加上下划线，并且全部转化成小写，将会标识出是否需要加上双引号。此种是默认策略。

    ```
    userInfo -> user_info
    id->id
    ages->ages
    lastName -> last_name
    myAddress -> my_address
    ```

* 加载原理与自定义方法

  * 如果我们修改默认策略，只需要在 application.properties 里面修改下面代码所示的两个配置，换成自己的自定义的类即可。

    ```properties
    spring.jpa.hibernate.naming.implicit-strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
    spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
    ```

#### 生产环境多数据源的处理方法

* 第一种方式：多个数据源的 @Configuration 的配置方法

  * 这种方式的主要思路是，不同 Package 下面的实体和 Repository 采用不同的 Datasource。所以我们改造一下我们的 example 目录结构，来看看不同 Repositories 的数据源是怎么处理的。

  * **第一步：规划 Entity 和 Repository 的目录结构，为了方便配置多数据源。**

    * 将 User 和 UserAddress、UserRepository 和 UserAddressRepository 移动到 db1 里面；将 UserInfo 和 UserInfoRepository 移动到 db2 里面。
    * 我们把实体和 Repository 分别放到了 db1 和 db2 两个目录里面，这时我们假设数据源 1 是 MySQL，User 表和 UserAddress 在数据源 1 里面，那么我们需要配置一个 DataSource1 的 Configuration 类，并且在里面配置 DataSource、TransactionManager 和 EntityManager。.

  * **第二步：配置 DataSource1Config 类。**

    ```java
    目录结构调整完之后，接下来我们开始配置数据源，完整代码如下：
    @Configuration
    @EnableTransactionManagement//开启事务
    //利用EnableJpaRepositories配置哪些包下面的Repositories，采用哪个EntityManagerFactory和哪个trannsactionManager
    @EnableJpaRepositories(
          basePackages = {"com.example.jpa.example1.db1"},//数据源1的repository的包路径
          entityManagerFactoryRef = "db1EntityManagerFactory",//改变数据源1的EntityManagerFactory的默认值，改为db1EntityManagerFactory
          transactionManagerRef = "db1TransactionManager"//改变数据源1的transactionManager的默认值，改为db1TransactionManager
          )
    public class DataSource1Config {
       /**
        * 指定数据源1的dataSource配置
        * @return
        */
       @Primary
       @Bean(name = "db1DataSourceProperties")
       @ConfigurationProperties("spring.datasource1") //数据源1的db配置前缀采用spring.datasource1
       public DataSourceProperties dataSourceProperties() {
          return new DataSourceProperties();
       }
       /**
        * 可以选择不同的数据源，这里我用HikariDataSource举例，创建数据源1
        * @param db1DataSourceProperties
        * @return
        */
       @Primary
       @Bean(name = "db1DataSource")
       @ConfigurationProperties(prefix = "spring.datasource.hikari.db1") //配置数据源1所用的hikari配置key的前缀
       public HikariDataSource dataSource(@Qualifier("db1DataSourceProperties") DataSourceProperties db1DataSourceProperties) {
          HikariDataSource dataSource = db1DataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
          if (StringUtils.hasText(db1DataSourceProperties.getName())) {
             dataSource.setPoolName(db1DataSourceProperties.getName());
          }
          return dataSource;
       }
       /**
        * 配置数据源1的entityManagerFactory命名为db1EntityManagerFactory，用来对实体进行一些操作
        * @param builder
        * @param db1DataSource entityManager依赖db1DataSource
        * @return
        */
       @Primary
       @Bean(name = "db1EntityManagerFactory")
       public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder, @Qualifier("db1DataSource") DataSource db1DataSource) {
          return builder.dataSource(db2DataSource)
    .packages("com.example.jpa.example1.db1") //数据1的实体所在的路径
    .persistenceUnit("db1")// persistenceUnit的名字采用db1
    .build();
       }
       /**
        * 配置数据源1的事务管理者，命名为db1TransactionManager依赖db1EntityManagerFactory
        * @param db1EntityManagerFactory 
        * @return
        */
       @Primary
       @Bean(name = "db1TransactionManager")
       public PlatformTransactionManager transactionManager(@Qualifier("db1EntityManagerFactory") EntityManagerFactory db1EntityManagerFactory) {
          return new JpaTransactionManager(db1EntityManagerFactory);
       }
    }
    ```

  * **第三步：配置 DataSource2Config类，加载数据源 2**

    ```java
    @Configuration
    @EnableTransactionManagement//开启事务
    //利用EnableJpaRepositories，配置哪些包下面的Repositories，采用哪个EntityManagerFactory和哪个trannsactionManager
    @EnableJpaRepositories(
            basePackages = {"com.example.jpa.example1.db2"},//数据源2的repository的包路径
            entityManagerFactoryRef = "db2EntityManagerFactory",//改变数据源2的EntityManagerFactory的默认值，改为db2EntityManagerFactory
            transactionManagerRef = "db2TransactionManager"//改变数据源2的transactionManager的默认值，改为db2TransactionManager
    )
    public class DataSource2Config {
        /**
         * 指定数据源2的dataSource配置
         *
         * @return
         */
        @Bean(name = "db2DataSourceProperties")
        @ConfigurationProperties("spring.datasource2") //数据源2的db配置前缀采用spring.datasource2
        public DataSourceProperties dataSourceProperties() {
            return new DataSourceProperties();
        }
        /**
         * 可以选择不同的数据源，这里我用HikariDataSource举例，创建数据源2
         *
         * @param db2DataSourceProperties
         * @return
         */
        @Bean(name = "db2DataSource")
        @ConfigurationProperties(prefix = "spring.datasource.hikari.db2") //配置数据源2的hikari配置key的前缀
        public HikariDataSource dataSource(@Qualifier("db2DataSourceProperties") DataSourceProperties db2DataSourceProperties) {
            HikariDataSource dataSource = db2DataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
            if (StringUtils.hasText(db2DataSourceProperties.getName())) {
                dataSource.setPoolName(db2DataSourceProperties.getName());
            }
            return dataSource;
        }
        /**
         * 配置数据源2的entityManagerFactory命名为db2EntityManagerFactory，用来对实体进行一些操作
         *
         * @param builder
         * @param db2DataSource entityManager依赖db2DataSource
         * @return
         */
        @Bean(name = "db2EntityManagerFactory")
        public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder, @Qualifier("db2DataSource") DataSource db2DataSource) {
            return builder.dataSource(db2DataSource)
                .packages("com.example.jpa.example1.db2") //数据2的实体所在的路径
                .persistenceUnit("db2")// persistenceUnit的名字采用db2
                .build();
        }
        /**
         * 配置数据源2的事务管理者，命名为db2TransactionManager依赖db2EntityManagerFactory
         *
         * @param db2EntityManagerFactory
         * @return
         */
        @Bean(name = "db2TransactionManager")
        public PlatformTransactionManager transactionManager(@Qualifier("db2EntityManagerFactory") EntityManagerFactory db2EntityManagerFactory) {
            return new JpaTransactionManager(db2EntityManagerFactory);
        }
    }
    ```

  * **第四步：通过 application.properties 配置两个数据源的值，代码如下：**

    ```properties
    ###########datasource1 采用Mysql数据库
    spring.datasource1.url=jdbc:mysql://localhost:3306/test2?logger=Slf4JLogger&profileSQL=true
    spring.datasource1.username=root
    spring.datasource1.password=root
    ##数据源1的连接池的名字
    spring.datasource.hikari.db1.pool-name=jpa-hikari-pool-db1
    ##最长生命周期15分钟够了
    spring.datasource.hikari.db1.maxLifetime=900000
    spring.datasource.hikari.db1.maximumPoolSize=8
    ###########datasource2 采用h2内存数据库
    spring.datasource2.url=jdbc:h2:~/test
    spring.datasource2.username=sa
    spring.datasource2.password=sa
    ##数据源2的连接池的名字
    spring.datasource.hikari.db2.pool-name=jpa-hikari-pool-db2
    ##最长生命周期15分钟够了
    spring.datasource.hikari.db2.maxLifetime=500000
    ##最大连接池大小和数据源1区分开，我们配置成6个
    spring.datasource.hikari.db2.maximumPoolSize=6
    ```

* 第二种方式：利用 AbstractRoutingDataSource 配置多数据源

  * **第一步：定一个数据源的枚举类，用来标示数据源有哪些。**

    ```java
    /**
     * 定义一个数据源的枚举类
     */
    public enum RoutingDataSourceEnum {
       DB1, //实际工作中枚举的语义可以更加明确一点；
       DB2;
       public static RoutingDataSourceEnum findbyCode(String dbRouting) {
          for (RoutingDataSourceEnum e : values()) {
             if (e.name().equals(dbRouting)) {
                return e;
             }
          }
          return db1;//没找到的情况下，默认返回数据源1
       }
    }
    ```

  * **第二步：新增 DataSourceRoutingHolder，用来存储当前线程需要采用的数据源。**

    ```java
    /**
     * 利用ThreadLocal来存储，当前的线程使用的数据
     */
    public class DataSourceRoutingHolder {
       private static ThreadLocal<RoutingDataSourceEnum> threadLocal = new ThreadLocal<>();
       public static void setBranchContext(RoutingDataSourceEnum dataSourceEnum) {
          threadLocal.set(dataSourceEnum);
       }
       public static RoutingDataSourceEnum getBranchContext() {
          return threadLocal.get();
       }
       public static void clearBranchContext() {
          threadLocal.remove();
       }
    }
    ```

  * **第三步：配置 RoutingDataSourceConfig，用来指定哪些 Entity 和 Repository 采用动态数据源。**

    ```java
    @Configuration
    @EnableTransactionManagement
    @EnableJpaRepositories(
          //数据源的repository的包路径，这里我们覆盖db1和db2的包路径
          basePackages = {"com.example.jpa.example1"},
          entityManagerFactoryRef = "routingEntityManagerFactory",
          transactionManagerRef = "routingTransactionManager"
    )
    public class RoutingDataSourceConfig {
       @Autowired
       @Qualifier("db1DataSource")
       private DataSource db1DataSource;
       @Autowired
       @Qualifier("db2DataSource")
       private DataSource db2DataSource;
       /**
        * 创建RoutingDataSource，引用我们之前配置的db1DataSource和db2DataSource
        *
        * @return
        */
       @Bean(name = "routingDataSource")
       public DataSource dataSource() {
          Map<Object, Object> dataSourceMap = Maps.newHashMap();
          dataSourceMap.put(RoutingDataSourceEnum.DB1, db1DataSource);
          dataSourceMap.put(RoutingDataSourceEnum.DB2, db2DataSource);
          RoutingDataSource routingDataSource = new RoutingDataSource();
          //设置RoutingDataSource的默认数据源
          routingDataSource.setDefaultTargetDataSource(db1DataSource);
          //设置RoutingDataSource的数据源列表
          routingDataSource.setTargetDataSources(dataSourceMap);
          return routingDataSource;
       }
       /**
        * 类似db1和db2的配置，唯一不同的是，这里采用routingDataSource
        * @param builder
        * @param routingDataSource entityManager依赖routingDataSource
        * @return
        */
       @Bean(name = "routingEntityManagerFactory")
       public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder, @Qualifier("routingDataSource") DataSource routingDataSource) {
          return builder.dataSource(routingDataSource).packages("com.example.jpa.example1") //数据routing的实体所在的路径，这里我们覆盖db1和db2的路径
                .persistenceUnit("db-routing")// persistenceUnit的名字采用db-routing
                .build();
       }
       /**
        * 配置数据的事务管理者，命名为routingTransactionManager依赖routtingEntityManagerFactory
        *
        * @param routingEntityManagerFactory
        * @return
        */
       @Bean(name = "routingTransactionManager")
       public PlatformTransactionManager transactionManager(@Qualifier("routingEntityManagerFactory") EntityManagerFactory routingEntityManagerFactory) {
          return new JpaTransactionManager(routingEntityManagerFactory);
       }
    }
    ```

  * **第四步：写一个 MVC 拦截器，用来指定请求分别采用什么数据源。**

    ```
    /**
     * 动态路由的实现逻辑，我们通过请求里面的db-routing，来指定此请求采用什么数据源
     */
    @Component
    public class DataSourceInterceptor extends HandlerInterceptorAdapter {
       /**
        * 请求处理之前更改线程里面的数据源
        */
       @Override
       public boolean preHandle(HttpServletRequest request,
                          HttpServletResponse response, Object handler) throws Exception {
          String dbRouting = request.getHeader("db-routing");
          DataSourceRoutingHolder.setBranchContext(RoutingDataSourceEnum.findByCode(dbRouting));
          return super.preHandle(request, response, handler);
       }
       /**
        * 请求结束之后清理线程里面的数据源
        */
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
          super.afterCompletion(request, response, handler, ex);
          DataSourceRoutingHolder.clearBranchContext();
       }
    }
    ```

  * 多数据源实战注意事项

    * 此种方式利用了当前线程事务不变的原理，所以要注意异步线程的处理方式；
    * 此种方式利用了 DataSource 的原理，动态地返回不同的 db 连接，一般需要在开启事务之前使用，需要注意事务的生命周期；
    * 比较适合读写操作分开的业务场景；
    * 多数据的情况下，避免一个事务里面采用不同的数据源，这样会有意想不到的情况发生，比如死锁现象；
    * 学会通过日志检查我们开启请求的方法和开启的数据源是否正确，可以通过 Debug 断点来观察数据源是否选择的正确

#### Datasource 与 TransactionManager、EntityManagerFactory 的关系分析

* HikariDataSource 负责实现 DataSource，交给 EntityManager 和 TransactionManager 使用；
* EntityManager 是利用 Datasouce 来操作数据库，而其实现类是 SessionImpl；
* EntityManagerFactory 是用来管理和生成 EntityManager 的，而 EntityManagerFactory 的实现类是 LocalContainerEntityManagerFactoryBean，通过实现 FactoryBean 接口实现，利用了 FactoryBean 的 Spring 中的 bean 管理机制，所以需要我们在 Datasource1Config 里面配置 LocalContainerEntityManagerFactoryBean 的 bean 的注入方式；
* JpaTransactionManager 是用来管理事务的，实现了 TransactionManager 并且通过 EntityFactory 和 Datasource 进行 db 操作，所以我们要在 DataSourceConfig 里面告诉 JpaTransactionManager 用的 TransactionManager 是 db1EntityManagerFactory。
