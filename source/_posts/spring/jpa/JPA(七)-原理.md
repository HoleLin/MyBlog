---
title: JPA(七)-原理
date: 2021-08-24 16:40:25
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 原理
categories:
- Spring
- JPA
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

#### `JpaProperties` 属性

```properties
# 可以配置JPA的实现者的原始属性的配置，如：这里我们用的JPA的实现者是hibernate
# 那么hibernate里面的一些属性设置就可以通过如下方式实现，具体properties里面有哪些，本讲会详细介绍，我们先知道这里可以设置即可
spring.jpa.properties.hibernate.hbm2ddl.auto=none
#hibernate的persistence.xml文件有哪些，目前已经不推荐使用
#spring.jpa.mapping-resources=
# 指定数据源的类型，如果不指定，Spring Boot加载Datasource的时候会根据URL的协议自己判断
# 如：spring.datasource.url=jdbc:mysql://localhost:3306/test 上面可以明确知道是mysql数据源，所以这个可以不需要指定；
# 应用场景，当我们通过代理的方式，可能通过datasource.url没办法判断数据源类型的时候，可以通过如下方式指定，可选的值有：DB2,H2,HSQL,INFORMIX,MYSQL,ORACLE,POSTGRESQL,SQL_SERVER,SYBASE)
spring.jpa.database=mysql
# 是否在启动阶段根据实体初始化数据库的schema，默认false，当我们用内存数据库做测试的时候可以打开，很有用
spring.jpa.generate-ddl=false
# 和spring.jpa.database用法差不多，指定数据库的平台，默认会自己发现；一般不需要指定，database-platform指定的必须是org.hibernate.dialect.Dialect的子类，如mysql默认是用下面的platform
spring.jpa.database-platform=org.hibernate.dialect.MySQLInnoDBDialect
# 是否在view层打开session，默认是true，其实大部分场景不需要打开，我们可以设置成false，
# 22课时我们再详细讲解
spring.jpa.open-in-view=false
# 是否显示sql，当执行JPA的数据库操作的时候，默认是false，在本地开发的时候我们可以把这个打开，有助于分析sql是不是我们预期的
# 在生产环境的时候建议给这个设置成false，改由logging.level.org.hibernate.SQL=DEBUG代替，这样的话日志默认是基于logback输出的
# 而不是直接打印到控制台的，有利于增加traceid和线程ID等信息，便于分析
spring.jpa.show-sql=true
```

#### Debug 时日志的配置

```properties
### 日志级别的灵活运用
## hibernate相关
# 显示sql的执行日志，如果开了这个,show_sql就可以不用了
logging.level.org.hibernate.SQL=debug
# hibernate id的生成日志
logging.level.org.hibernate.id=debug
# hibernate所有的操作都是PreparedStatement，把sql的执行参数显示出来
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
# sql执行完提取的返回值
logging.level.org.hibernate.type.descriptor.sql=trace
# 请求参数
logging.level.org.hibernate.type=debug
# 缓存相关
logging.level.org.hibernate.cache=debug
# 统计hibernate的执行状态
logging.level.org.hibernate.stat=debug
# 查看所有的缓存操作
logging.level.org.hibernate.event.internal=trace
logging.level.org.springframework.cache=trace
# hibernate 的监控指标日志
logging.level.org.hibernate.engine.internal.StatisticalLoggingSessionEventListener=DEBUG
### 连接池的相关日志
## hikari连接池的状态日志，以及连接池是否完好 #连接池的日志效果：HikariCPPool - Pool stats (total=20, active=0, idle=20, waiting=0)
logging.level.com.zaxxer.hikari=TRACE
#开启 debug可以看到 AvailableSettings里面的默认配置的值都有哪些，会输出类似下面的日志格式
# org.hibernate.cfg.Settings               : Statistics: enabled
# org.hibernate.cfg.Settings               : Default batch fetch size: -1
logging.level.org.hibernate.cfg=debug
#hikari数据的配置项日志
logging.level.com.zaxxer.hikari.HikariConfig=TRACE
### 查看事务相关的日志，事务获取，释放日志
logging.level.org.springframework.orm.jpa=DEBUG
logging.level.org.springframework.transaction=TRACE
logging.level.org.hibernate.engine.transaction.internal.TransactionImpl=DEBUG
### 分析connect 以及 orm和 data的处理过程更全的日志
logging.level.org.springframework.data=trace
logging.level.org.springframework.orm=trace

当我们分析一个问题的时候，如果不知道日志具体在哪个类里面，通过设置 logging.level.root=trace 的话，日志又非常多几乎没有办法看，那么我们可以缩小范围，不如说我们分析的是 hikari 包里面相关的问题。
我们可以把整个日志级别 logging.level.root=info 设置成 info，把其他所有的日志都关闭，并把 logging.level.com.zaxxer=trace 设置成最大的，保持日志不受干扰，然后观察日志再逐渐减少查看范围
```

#### @PersistenceUnit  @PersistenceContext

* EntityManagerFactory 和 Persistence Unit 是什么？

  * 按照 JPA 协议里面的定义：persistence unit 是一些持久化配置的集合，里面包含了数据源的配置、EntityManagerFactory 的配置，spring 3.1 之前主要是通过 persistence.xml 的方式来配置一个 persistence unit。

* EntityManager 和 PersistenceContext 是什么？

  * 按照 JPA 协议的规范，我们先理解一下 PersistenceContext，它是用来管理会话里面的 Entity 状态的一个上下文环境，使 Entity 的实例有了不同的状态，也就是我们所说的实体实例的生命周期。
  * 而这些实体在 PersistenceContext 中的不同状态都是通过 EntityManager 提供的一些方法进行管理的，也就是说：
    * PersistenceContext 是持久化上下文，是 JPA 协议定义的，而 Hibernate 的实现是通过 Session 创建和销毁的，也就是说一个 Session 有且仅有一个 PersistenceContext；
    * PersistenceContext 既然是持久化上下文，里面管理的是 Entity 的状态；
    * EntityManager 是通过 PersistenceContext 创建的，用来管理 PersistenceContext 中 Entity 状态的方法，离开PersistenceContext 持久化上下文，EntityManager 没有意义；
    * EntityManger 是操作对象的唯一入口，一个请求里面可能会有多个 EntityManger 对象。

#### 实体对象的生命周期

  * 既然 PersistenceContext 是存储 Entity 的，那么 Entity 在 PersistenceContext 里面肯定有不同的状态。对此，JPA 协议定义了四种状态：new、manager、detached、removed。

    ![img](https://www.chenjunlin.vip/img/spring/jpa/%E5%AE%9E%E4%BD%93%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

###### 第一种：New 状态的对象

* 当我们使用关键字 new 的时候创建的实体对象，称为 new 状态的 Entity 对象。它需要同时满足两个条件：new 状态的实体 Id 和 Version 字段都是 null；new 状态的实体没有在 PersistenceContext 中出现过。

* 那么如果我们要把 new 状态的 Entity 放到 PersistenceContext 里面，有两种方法：执行 entityManager.persist(entity) 方法；通过关联关系的实体关系配置 cascade=PERSIST or cascade=ALL 这种类型，并且关联关系的一方，也执行了 entityManager.persist(entity) 方法。

###### 第二种：Detached（游离）的实体对象

* Detached 状态的对象表示和 PersistenceContext 脱离关系的 Entity 对象。它和 new 状态的对象的不同点在于：

  * Detached 是 new 状态的实体对象没有持久化 ID（即没有 ID 和 version）；

  * 变成持久化对象需要进行 merger 操作，merger 操作会 copy 一个新的实体对象，然后把新的实体对象变成 Manager 状态。  

* 而 Detached 和 new 状态的对象相同点也有两个方面：

  * 都和 PersistenceContext 脱离了关系； 
* 当执行 flush 操作或者 commit 操作的时候，不会进行数据库同步。

###### 第三种：Manager（persist） 状态的实体

* Manager 状态的实体，顾名思义，是指在 PersistenceContext 里面管理的实体，而此种状态的实体当我们执行事务的 commit()，或者 entityManager 的 flush 方法的时候，就会进行数据库的同步操作。可以说是和数据库的数据有映射关系。

###### 第四种：Removed 的实体状态

* Removed 的状态，顾名思义就是指删除了的实体，但是此实体还在 PersistenceContext 里面，只是在其中表示为 Removed 的状态，它和 Detached 状态的实体最主要的区别就是不在 PersistenceContext 里面，但都有 ID 属性。

* **MyBatis 是对数据库的操作所见即所得的模式；而使用 JPA，你的任何操作都不会产生 DB 的sql。**

#### Flush 的作用

* flush 重要的、唯一的作用，就是将 Persistence Context 中变化的实体转化成 sql 语句，同步执行到数据库里面。换句话来说，如果我们不执行 flush() 方法的话，通过 EntityManager 操作的任何 Entity 过程都不会同步到数据库里面。

* Flush 的机制是什么？

  * JPA 协议规定了 EntityManager 可以通过如下方法修改 FlushMode。

  ```java
  //entity manager 里面提供的修改FlushMode的方法
  public void setFlushMode(FlushModeType flushMode);
  //FlushModeType只有两个值，自动和事务提交之前
  public enum FlushModeType {
      //事务commit之前
     COMMIT,
      //自动规则，默认
     AUTO
  }
  而 Hiberbernate 还提供了一种手动触发的机制，可以通过如下代码的方式进行修改。
  @PersistenceContext(properties = {@PersistenceProperty(
          name = "org.hibernate.flushMode",
          value = "MANUAL"//手动flush
  )})
  private EntityManager entityManager;
  ```

* Flush 的自动机制

  * 事务 commit 之前，即指执行 transactionManager.commit() 之前都会触发，这个很好理解；
  * 执行任何的 JPQL 或者 native SQL（代替直接操作 Entity 的方法）都会触发 flush。

* Flush 的时候会改变 SQL 的执行顺序

  * flush() 方法调用之后，同一个事务内，sql 的执行顺序会变成如下模式：insert 的先执行、delete 的第二个执行、update 的第三个执行。
  * 这种会改变顺序的现象，主要是由 persistence context 的实体状态机制导致的，所以在 Hibernate 的环境中，顺序会变成如下的 ActionQueue 的模式：
    * `OrphanRemovalAction`
    * `EntityInsertAction`or`EntityIdentityInsertAction`
    * ` EntityUpdateAction`
    * ` CollectionRemoveAction`
    * `CollectionUpdateAction`
    * `CollectionRecreateAction`
    * `EntityDeleteAction`

* Flush 与事务 Commit 的关系

  * 在当前的事务执行 commit 的时候，会触发 flush 方法；
  * 在当前的事务执行完 commit 的时候，如果隔离级别是可重复读的话，flush 之后执行的 update、insert、delete 的操作，会被其他的新事务看到最新结果；
  * 假设当前的事务是可重复读的，当我们手动执行 flush 方法之后，没有执行事务 commit 方法，那么其他事务是看不到最新值变化的，但是最新值变化对当前没有 commit 的事务是有效的；
  * 如果执行了 flush 之后，当前事务发生了 rollback 操作，那么数据将会被回滚（数据库的机制）

#### Session、EntityManager、Connection 和 Transaction 的关系

* **Connection 和 Transaction 的关系**
  * 事务是建立在 Connection 之上的，没有连接就没有事务。
  * 以 MySQL InnoDB 为例，新开一个连接默认开启事务，默认每个 SQL 执行完之后自动提交事务。
  * 一个连接里面可以有多次串行的事务段；一个事务只能属于一个 Connection。
  * 事务与事务之间是相互隔离的，那么自然不同连接的不同事务也是隔离的。
* **EntityManager、Connection 和 Transaction 的关系**
  * EntityManager 里面有 DataSource，当 EntityManager 里面开启事务的时候，先判断当前线程里面是否有数据库连接，如果有直接用。
  * 开启事务之前先开启连接；关闭事务，不一定关闭连接。
  * 开启 EntityManager，不一定立马获得连接；获得连接，不一定立马开启事务。
  * 关闭 EntityManager，一定关闭事务，释放连接；反之不然。
* **Session、EntityManager、Connection 和 Transaction 的关系**
  * Session 是 EntityManager 的子类，SessionImpl 是 Session 和 EntityManager 的实现类。那么自然 EntityManager 和 Connection、Transaction 的关系同样适用 Session、EntityManager、Connection 和 Transaction 的关系。
  * Session 的生命周期决定了 EntityManager 的生命周期。
* **Session 和 Transaction 的关系**
  * 在 Hibernate 的 JPA 实现里面，开启 Transaction 之前，必须要先开启 Session。
  * 默认情况下，Session 的生命周期由 open-in-view 决定是请求之前开启，还是事务之前开启。
  * 事务关闭了，Session 不一定关闭。
  * Session 关闭了，事务一定关闭。

#### N+1SQL问题

```java
//UserInfo实体对象如下：
@Entity
@Data
@SuperBuilder
@AllArgsConstructor
@NoArgsConstructor
@Table
@ToString(exclude = "addressList")//exclued防止 toString打印日志的时候死循环
public class UserInfo extends BaseEntity {
   private String name;
   private String telephone;
   // UserInfo实体对象的关联关系由Address对象里面的userInfo字段维护，默认是lazy加载模式，为了方便演示fetch取EAGER模式。此处是一对多关联关系
   @OneToMany(mappedBy = "userInfo",fetch = FetchType.EAGER)
   private List<Address> addressList;
}
//Address对象如下：
@Entity
@Table
@Data
@SuperBuilder
@AllArgsConstructor
@NoArgsConstructor
@ToString(exclude = "userInfo")
public class Address extends BaseEntity {
   private String city;
   //维护UserInfo和Address的外键关系，方便演示也采用EAGER模式；
   @ManyToOne(fetch = FetchType.EAGER)
   @JsonBackReference //此注解防止JSON死循环
   private UserInfo userInfo;
}
所谓的 N+1 的 SQL，此时 1 代表的是一条 SQL 查询 UserInfo 信息；N 条 SQL 查询 Address 的信息
```

* 解决方法一:
  * hibernate.default_batch_fetch_size 配置在 AvailableSettings.class 里面，指的是批量获取数据的大小，默认是 -1，表示默认没有匹配取数据。

    ```
    # 更改批量取数据的大小为20
    spring.jpa.properties.hibernate.default_batch_fetch_size= 20
    
    在实际工作中，一定要知道我们一次操作会产生多少 SQL，有没有预期之外的 SQL 参数，这是需要关注的重点，这种情况可以利用我们之前说过的如下配置来开启打印 SQL
    ## 显示sql的执行日志，如果开了这个,show_sql就可以不用了，show_sql没有上下文，多线程情况下，分不清楚是谁打印的，所有我推荐如下配置项：
    logging.level.org.hibernate.SQL=debug
    ```

  * 但是这种配置也有个缺陷，就是只能全局配置，没办法针对不通过的实体管理关系配置不同的 Fetch Size 的值。

* 解决方法二:

  * @BatchSize 注解是 Hibernate 提供的用来解决查询关联关系的批量处理大小，默认无，可以配置在实体上，也可以配置在关联关系上面。此注解里面只有一个属性 size，用来指定关联关系 LAZY 或者是 EAGER 一次性取数据的大小。
  * @BatchSize 只能作用在 @ManyToMany、@OneToMany、实体类这三个地方。

  ```java
  @Entity
  @Data
  @SuperBuilder
  @AllArgsConstructor
  @NoArgsConstructor
  @Table
  @ToString(exclude = "addressList")
  @BatchSize(size = 2)//实体类上加@BatchSize注解，用来设置当被关联关系的时候一次查询的大小，我们设置成2，方便演示Address关联UserInfo的时候的效果
  public class UserInfo extends BaseEntity {
     private String name;
     private String telephone;
     @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
     @BatchSize(size = 20)//关联关系的属性上加@BatchSize注解，用来设置当通过UserInfo加载Address的时候一次取数据的大小
     private List<Address> addressList;
  }
  ```

  * 注意事项：@BatchSize 的使用具有局限性，不能作用于 @ManyToOne 和 @OneToOne 的关联关系上，那样代码是不起作用的，如下所示。

    ```java
    public class Address extends BaseEntity {
       private String city;
       @ManyToOne(cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
       @BatchSize(size = 30) //由于是@ManyToOne的关联关系所有没有作用
       private UserInfo userInfo;
    }
    ```

* 解决方法三:

  * Hibernate 中还提供了一种 FetchMode 的策略，包含三种模式，分别为 FetchMode.SELECT、FetchMode.JOIN，以及 FetchMode.Subselect。

    * Hibernate 中 @Fetch 数据的策略

      ```java
      // fetch注解只能用在方法和字段上面
      @Target({ElementType.METHOD, ElementType.FIELD})
      @Retention(RetentionPolicy.RUNTIME)
      public @interface Fetch {
         //注解里面，只有一个属性获取数据的模式
         FetchMode value();
      }
      //其中FetchMode的值有如下几种：
      public enum FetchMode {
         //默认模式，就是会有N+1 sql的问题；
         SELECT,
         //通过join的模式，用一个sql把主体数据和关联关系数据一口气查出来
         JOIN,
         //通过子查询的模式，查询关联关系的数据
         SUBSELECT
      }
      ```

    * 需要注意的是，不要把这个注解和 JPA 协议里面的 FetchType.EAGER、FetchType.LAZY 搞混了，JPA 协议的关联关系中的 FetchTyp 解决的是取关联关系数据时机的问题，也就是说 EAGER 代表的是立即获得关联关系的数据，LAZY 是需要的时候再获得关联关系的数据。

    * `FetchMode.SELECT`

      ```java
      @Entity
      @Data
      @SuperBuilder
      @AllArgsConstructor
      @NoArgsConstructor
      @Table
      @ToString(exclude = "addressList")
      public class UserInfo extends BaseEntity {
         private String name;
         private String telephone;
         @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
         @Fetch(value = FetchMode.SELECT)
         private List<Address> addressList;
      }
      ```

    * `FetchMode.JOIN`

      ```java
      public class UserInfo extends BaseEntity {
         private String name;
         private String telephone;
         @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
         @Fetch(value = FetchMode.JOIN) //唯一变化的地方采用JOIN模式
         private List<Address> addressList;
      }
      ```

    * `FetchMode.SUBSELECT`

      ```java
      public class UserInfo extends BaseEntity {
         @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.LAZY) //我们这里测试一下LAZY情况
         @Fetch(value = FetchMode.SUBSELECT) //唯一变化之处
         private List<Address> addressList;
      }
      ```

    * FetchMode.SUBSELECT 支持 ID 查询和各种条件查询，唯一的缺点是只能配置在 @OneToMany 和 @ManyToMany 的关联关系上，不能配置在 @ManyToOne 和 @OneToOne 的关联关系上

    * @Fetch 的不同模型，都有各自的优缺点：FetchMode.SELECT 默认，和不配置的效果一样；FetchMode.JOIN 只支持类似 findById(id) 的方法，只能根据 ID 查询才有效果；FetchMode.SUBSELECT 虽然不限使用方式，但是只支持 ToMany 的关联关系。

* 解决方法四:

  * JPA 协议企图通过 @NamedEntityGraph 注解来描述实体之间的关联关系，当被 @EntityGraph 使用的时候进行 EAGER 加载，以减少 N+1 的 SQL

  * @NamedEntityGraph 和 @EntityGraph 用法

    ```java
    //可以被@NamedEntityGraphs注解重复使用，只能配置在类上面，用来声明不同的EntityGraph；
    @Repeatable(NamedEntityGraphs.class)
    @Target({TYPE})
    @Retention(RUNTIME)
    public @interface NamedEntityGraph {
        //指定一个名字
        String name() default "";
        //哪些关联关系属性可以被EntityGraph包含进去，默认一个没有。可以配置多个
        NamedAttributeNode[] attributeNodes() default {};
    
        //是否所有的关联关系属性自动包含在内，默认false;
        boolean includeAllAttributes() default false;
    
        //配置subgraphs，子实体图(可以理解为关联关系实体图，即如果算层级，可以配置第二层级)，可以被NamedAttributeNode引用
        NamedSubgraph[] subgraphs() default {};
        //配置subclassSubgraphs的namedSubgraph有哪些。即如果算层级，可以配置第三层级
        NamedSubgraph[] subclassSubgraphs() default {};
    }
    
    // @NamedEntityGraphs 能够配置多个 @NamedEntityGraph 只能使用在实体类上面
    @Target({TYPE})
    @Retention(RUNTIME)
    public @interface NamedEntityGraphs{
        NamedEntityGraph[] value();//可以同时指定多个NamedEntityGraph
    }
    
    // 用来进行属性节点的描述
    @Target({})
    @Retention(RUNTIME)
    public @interface NamedAttributeNode {
        //要包含的关联关系的属性的名字，必填
        String value();
        //如果我们在@NamedEntityGraph里面配置了子关联关系，这个是配置subgraph的名字
        String subgraph() default "";
       //当关联关系是被Map结构引用的时候，我们可以指定key的方式，一般很少用
        String keySubgraph() default "";
    }
    
    @Target({})
    @Retention(RUNTIME)
    public @interface NamedSubgraph {
        //指定一个名字
        String name();
        //子关联关系的类的class
        Class type() default void.class;
        //二层关联关系的要包含的关联关系的属性的名字
        NamedAttributeNode[] attributeNodes();
    }
    
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })
    //EntityGraph 作用在Repository的接口里面的方法上面
    public @interface EntityGraph {
       //指@EntityGraph注解引用的@NamedEntityGraph里面定义的name，如果是空EntityGraph就不会起作用，如果为空相当于没有配置；
       String value() default "";
       //EntityGraph的类型，默认是EntityGraphType.FETCH类型，我们接着往下看EntityGraphType一共有几个值
       EntityGraphType type() default EntityGraphType.FETCH;
        //可以指定attributePaths用来覆盖@NamedEntityGraph里面的attributeNodes的配置，默认配置是空，以@NamedEntityGraph里面的为准；
       String[] attributePaths() default {};
       //JPA 2.1支持的EntityGraphType对应的枚举值
       public enum EntityGraphType {
          //LOAD模式，当被指定了这种模式、被@EntityGraph管理的attributes的时候，原来的FetchType的类型直接忽略变成Eager模式，而不被@EntityGraph管理的attributes还是保持默认的FetchType
          LOAD("javax.persistence.loadgraph"),
          //FETCH模式，当被指定了这种模式、被@EntityGraph管理的attributes的时候，原来的FetchType的类型直接忽略变成Eager模式，而不被@EntityGraph管理的attributes将会变成Lazy模式，和LOAD的区别就是对不被@NamedEntityGraph配置的关联关系的属性的FetchType不一样；
          FETCH("javax.persistence.fetchgraph");
          private final String key;
          private EntityGraphType(String value) {
             this.key = value;
          }
          public String getKey() {
             return key;
          }
       }
    }
    ```

  * @EntityGraph 使用实例

    ```java
    第一步：在实体里面配置 @EntityGraph
    @Entity
    @Table
    @Data
    @SuperBuilder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "userInfo")
    //这里我们直接使用@NamedEntityGraph，因为只需要配置一个@NamedEntityGraph，我们指定一个名字getAllUserInfo，指定被这个名字的实体试图关联的关联关系属性是userInfo
    @NamedEntityGraph(name = "getAllUserInfo",attributeNodes = @NamedAttributeNode(value = "userInfo"))
    public class Address extends BaseEntity {
       private String city;
       @JsonBackReference //防止JSON死循环
       @ManyToOne(cascade = CascadeType.PERSIST,fetch = FetchType.LAZY)//采用默认的lazy模式
       private UserInfo userInfo;
    }
    @Entity
    @Data
    @SuperBuilder
    @AllArgsConstructor
    @NoArgsConstructor
    @Table
    @ToString(exclude = "addressList")
    //UserInfo对应的关联关系，我们利用@NamedEntityGraphs配置了两个，一个是针对Address的关联关系，一个是name叫rooms的实体图包含了rooms属性；我们在UserInfo里面增加了两个关联关系；
    @NamedEntityGraphs(value = {@NamedEntityGraph(name = "addressGraph",attributeNodes = @NamedAttributeNode(value = "addressList")),@NamedEntityGraph(name = "rooms",attributeNodes = @NamedAttributeNode(value = "rooms"))})
    public class UserInfo extends BaseEntity {
       private String name;
       private String telephone;
       private Integer ages;
       //默认LAZY模式
       @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.LAZY)
       private List<Address> addressList;
       //默认EAGER模式
       @OneToMany(cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
       private List<Room> rooms;
    }
    
    第二步：在我们需要的 Repository 的方法上面直接使用 @EntityGraph
    //因为要用findAll()做测试，所以可以覆盖JpaRepository里面的findAll()方法，加上@EntityGraph注解
    public interface UserInfoRepository extends JpaRepository<UserInfo, Long>{
    @Override
    //我们指定EntityGraph引用的是，在UserInfo实例里面配置的name=addressGraph的NamedEntityGraph；
    // 这里采用的是LOAD的类型，也就是说被addressGraph配置的实体图属性address采用的fetch会变成 FetchType.EAGER模式，而没有被addressGraph实体图配置关联关系属性room还是采用默认的EAGER模式
    @EntityGraph(value = "addressGraph",type = EntityGraph.EntityGraphType.LOAD)
       List<UserInfo> findAll();
    }}
    
    public interface AddressRepository extends JpaRepository<Address, Long>{
    @Override 
    //可以覆盖原始方法，添加上不同的@EntityGraph策略
    //使用@EntityGraph查询所有Address的时候，指定name = "getAllUserInfo"的@NamedEntityGraph，采用默认的EntityGraphType.FETCH，如果Address里面有多个关联关系的时候，只有在name = "getAllUserInfo"的实体图配置的userInfo属性上采用Eager模式，其他关联关系属性没有指定，默认采用LAZY模式；
    @EntityGraph(value = "getAllUserInfo")
    List<Address> findAll();
    }
    ```

#### JPA  @Query 中SpEL的应用场景

* 通过 SpEL 取被 @Query 注解的方法参数

  ```java
  //用法一：根据下标取方法里面的参数
  @Query("select u from User u where u.age = ?#{[0]}") 
  List<User> findUsersByAge(int age);
  //用法二：#customer取@Param("customer")里面的参数
  @Query("select u from User u where u.firstname = :#{#customer.firstname}")
  List<User> findUsersByCustomersFirstname(@Param("customer") Customer customer);
  //用法三：用JPA约定的变量entityName取得当前实体的实体名字
  @Query("from #{#entityName}")
  List<UserInfo> findAllByEntityName();
  
  public interface UserInfoRepository extends JpaRepository<UserInfo, Long> {
     // JPA约定的变量entityName取得当前实体的实体名字
     @Query("from #{#entityName}")
     List<UserInfo> findAllByEntityName();
     
     //一个查询中既可以支持SpEL也可以支持普通的:ParamName的方式
     @Modifying
     @Query("update #{#entityName} u set u.name = :name where u.id =:id")
     void updateUserActiveState(@Param("name") String name, @Param("id") Long id);
     
     //演示SpEL根据数组下标取参数，和根据普通的Parma的名字:name取参数
     @Query("select u from UserInfo u where u.lastName like %:#{[0]} and u.name like %:name%")
     List<UserInfo> findContainingEscaped(@Param("name") String name);
     
     //SpEL取Parma的名字customer里面的属性
     @Query("select u from UserInfo u where u.name = :#{#customer.name}")
     List<UserInfo> findUsersByCustomersFirstname(@Param("customer") UserInfo customer);
     
     //利用SpEL根据一个写死的'jack'字符串作为参数
     @Query("select u from UserInfo u where u.name = ?#{'jack'}")
     List<UserInfo> findOliverBySpELExpressionWithoutArgumentsWithQuestionmark();
     
     //同时SpEL支持特殊函数escape和escapeCharacter
     @Query("select u from UserInfo u where u.lastName like %?#{escape([0])}% escape ?#{escapeCharacter()}")
     List<UserInfo> findByNameWithSpelExpression(String name);
     
     // #entityName和#[]同时使用
     @Query("select u from #{#entityName} u where u.name = ?#{[0]} and u.lastName = ?#{[1]}")
     List<UserInfo> findUsersByFirstnameForSpELExpressionWithParameterIndexOnlyWithEntityExpression(String name, String lastName);
     //对于 native SQL同样适用，并且同样支持取pageable分页里面的属性值
     @Query(value = "select * from (" //
           + "select u.*, rownum() as RN from (" //
           + "select * from user_info ORDER BY ucase(firstname)" //
           + ") u" //
           + ") where RN between ?#{ #pageable.offset +1 } and ?#{#pageable.offset + #pageable.pageSize}", //
           countQuery = "select count(u.id) from user_info u", //
           nativeQuery = true)
     Page<UserInfo> findUsersInNativeQueryWithPagination(Pageable pageable);
  }
  ```

* Spring-Security-Data 在 @Query 中的用法

  ```java
  // 根据当前用户email取当前用户的信息
  @Query("select u from UserInfo u where u.emailAddress = ?#{principal.email}")
  List<UserInfo> findCurrentUserWithCustomQuery();
  //如果当前用户是admin，我们就返回某业务的所有对象；如果不是admin角色，就只给当前用户的某业务数据
  @Query("select o from BusinessObject o where o.owner.emailAddress like "+
        "?#{hasRole('ROLE_ADMIN') ? '%' : principal.emailAddress}")
  List<BusinessObject> findBusinessObjectsForCurrentUser();
  ```

* SpEL 在 @Cacheable 中的应用场景

  ```java
  //缓存key取当前方法名，判断一下只有返回结果不为null或者非empty才进行缓存
  @Cacheable(value = "APP", key = "#root.methodName", cacheManager = "redis.cache", unless = "#result == null || #result.isEmpty()")
  @Override
  public Map<String, Map<String, String>> getAppGlobalSettings() {}
  //evict策略的key是当前参数customer里面的name属性
  @Caching(evict = {
  @CacheEvict(value="directory", key="#customer.name") })
  public String getAddress(Customer customer) {...}
  //在condition里面使用，当参数里面customer的name属性的值等于字符串Tom才放到缓存里面
  @CachePut(value="addresses", condition="#customer.name=='Tom'")
  public String getAddress(Customer customer) {...}
  //用在unless里面，利用SpEL的条件表达式判断，排除返回的结果地址长度小于64的请求
  @CachePut(value="addresses", unless="#result.length()<64")
  public String getAddress(Customer customer) {...}
  ```

  | 支持的属性    | 作用域     | 功能描述                                                     | 使用方法                             |
  | ------------- | ---------- | ------------------------------------------------------------ | ------------------------------------ |
  | methodName    | root 对象  | 当前被调用的方法名                                           | #root.methodName                     |
  | method        | root 对象  | 当前被调用的方法                                             | #root.method.name                    |
  | target        | root 对象  | 当前被调用的目标对象                                         | #root.target                         |
  | targetClass   | root 对象  | 当前被调用的目标对象类                                       | #root.targetClass                    |
  | args          | root 对象  | 当前被调用的方法的参数列表                                   | #root.args[0]                        |
  | caches        | root 对象  | 当前方法调用使用的缓存列表（如@Cacheable(value={“cache1”, “cache2”})），则有两个 cache | #root.caches[0].name                 |
  | argument name | 执行上下文 | 当前被调用的方法的参数，如 findById(Long id)，我们可以通过 #id 拿到参数 | #user.id<br/>表示参数 user 里面的 id |
  | result        | 执行上下文 | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache evict’的 beforeInvocation=false） | #result                              |

### 解决 In 查询条件内存泄漏的方法

* 第一种方法：修改缓存的最大条数限制

  ```
  默认 DEFAULT_QUERY_PLAN_MAX_COUNT = 2048，也就是 query plan 的最大条数限制是 2048。这样默认值可能有点大了，我们可以通过如下方式修改默认值
  #修改 默认的plan_cache_max_size，太小会影响JPQL的执行性能，所以根据实际情况可以自由调整，不宜太小，也不宜太大，太大可能会引发内存溢出
  spring.jpa.properties.hibernate.query.plan_cache_max_size=512
  #修改 默认的native query的cache大小
  spring.jpa.properties.hibernate.query.plan_parameter_metadata_max_size=128
  ```

* 第二种方法：根据 max plan count 适当增加堆内存大小

  ```
  因为 QueryPlanMaxCount 是有限制的，那么肯定最大堆内存的使用也是有封顶限制的，我们找到临界值修改最小、最大堆内存即可。
  ```

* 第三种方法：减少 In 的查询 SQL 生成条数

    ```
    ### 默认情况下，不同的in查询条件的个数会生成不同的plan query cache，我们开启了in_clause_parameter_padding之后会减少in生成cache的个数，会根据参数的格式运用几何的算法生成QueryCache；
    spring.jpa.properties.hibernate.query.in_clause_parameter_padding=true
    ```

    
