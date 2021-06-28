---
title: JPA
date: 2021-06-27 20:54:53
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- JPA
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
## JPA

### 1. 基础使用

* 市场上的ORM框架对比

  ![](https://s0.lgstatic.com/i/image/M00/4E/B1/Ciqc1F9fBfeANrGuAAOa8Y2E5fU233.png)

  ![Drawing 9.png](https://s0.lgstatic.com/i/image/M00/4E/AF/Ciqc1F9fA1uARUnvAAB2ZNS1UXc485.png)

* Spring Data Common的依赖关系

  * ![Drawing 0.png](https://s0.lgstatic.com/i/image/M00/50/6F/Ciqc1F9i18OABIgzAAGVeUj3uCU674.png)
    * 数据库连接用的是**JDBC**
    * 连接池用的是**HikariCP**
    * 强依赖**Hibernate**
    * Spring Boot Starter Data JPA 依赖Spring Data JPA 而Spring Data JPA依赖Spring Data Commons

* **Repository接口**

  * Repository是Spring Date Common里面的顶级父类接口,操作DB的入口类.

* **Respository类层次关系**

  * ReactiveCrudRepository 这条线是响应式编程，主要支持当前 NoSQL 方面的操作，因为这方面大部分操作都是分布式的，所以由此我们可以看出 Spring Data 想统一数据操作的“野心”，即想提供关于所有 Data 方面的操作。目前 Reactive 主要有 Cassandra、MongoDB、Redis 的实现。

  * RxJava2CrudRepository 这条线是为了支持 RxJava 2 做的标准响应式编程的接口。

  * CoroutineCrudRepository 这条继承关系链是为了支持 Kotlin 语法而实现的。

  * CrudRepository

* **7 个大 Repository 接口：**

  * **Repository(**org.springframework.data.repository)，没有暴露任何方法；

  * **CrudRepository**(org.springframework.data.repository)，简单的 Curd 方法；

  * **PagingAndSortingRepository**(org.springframework.data.repository)，带分页和排序的方法；

  * **QueryByExampleExecutor**(org.springframework.data.repository.query)，简单 Example 查询；

  * **JpaRepository**(org.springframework.data.jpa.repository)，JPA 的扩展方法；

  * **JpaSpecificationExecutor**(org.springframework.data.jpa.repository)，JpaSpecification 扩展查询；

  * **QueryDslPredicateExecutor**(org.springframework.data.querydsl)，QueryDsl 的封装。

* **两大 Repository 实现类：**

  * **SimpleJpaRepository**(org.springframework.data.jpa.repository.support)，JPA 所有接口的默认实现类；
  * **QueryDslJpaRepository**(org.springframework.data.jpa.repository.support)，QueryDsl 的实现类。

* **CrudRepository 接口**

  ```java
  @NoRepositoryBean
  public interface CrudRepository<T, ID> extends Repository<T, ID> {
      // 保存实体方法，参数和返回结果可以是实体的子类；
      <S extends T> S save(S entity);
  	// 批量保存，原理和 save方法相同，我们去看实现的话，就是 for 循环调用上面的 save 方法。
      <S extends T> Iterable<S> saveAll(Iterable<S> entities);
  	// 根据主键查询实体，返回 JDK 1.8 的 Optional，这可以避免 null exception
      Optional<T> findById(ID id);
  	// 根据主键判断实体是否存在
      boolean existsById(ID id);
  	//  查询实体的所有列表
      Iterable<T> findAll();
  	// 根据主键列表查询实体列表
      Iterable<T> findAllById(Iterable<ID> ids);
      // 查询总数返回 long 类型
      long count();
  	// 根据主键删除，查看源码会发现，其是先查询出来再进行删除
      void deleteById(ID id);
  	// 根据 entity 进行删除
      void delete(T entity);
  	// 批量删除
      void deleteAll(Iterable<? extends T> entities);
  	// 删除所有
      void deleteAll();
  }
  ```

* **PagingAndSortingRepository 接口**

  * 该接口也是 Repository 接口的子类，主要用于分页查询和排序查询。
  
  ```
  @NoRepositoryBean
  public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
  
     /**
      * Returns all entities sorted by the given options.
      * 根据排序参数，实现不同的排序规则获取所有的对象的集合；
      * @param sort
      * @return all entities sorted by the given options
      */
     Iterable<T> findAll(Sort sort);
  
     /**
      * Returns a {@link Page} of entities meeting the paging restriction provided in the {@code Pageable} object.
      * 根据分页和排序进行查询，并用 Page 对返回结果进行封装。而 Pageable 对象包含 Page 和 Sort 对象。
      * @param pageable
      * @return a page of entities
      */
     Page<T> findAll(Pageable pageable);
  }
  ```

* **JpaRepository 接口**

  ```java
  @NoRepositoryBean
  public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.CrudRepository#findAll()
  	 */
  	@Override
  	List<T> findAll();
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.PagingAndSortingRepository#findAll(org.springframework.data.domain.Sort)
  	 */
  	@Override
  	List<T> findAll(Sort sort);
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.CrudRepository#findAll(java.lang.Iterable)
  	 */
  	@Override
  	List<T> findAllById(Iterable<ID> ids);
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.CrudRepository#save(java.lang.Iterable)
  	 */
  	@Override
  	<S extends T> List<S> saveAll(Iterable<S> entities);
  	/**
  	 * Flushes all pending changes to the database.
  	 */
  	void flush();
  	/**
  	 * Saves an entity and flushes changes instantly.
  	 *
  	 * @param entity
  	 * @return the saved entity
  	 */
  	<S extends T> S saveAndFlush(S entity);
  	/**
  	 * Deletes the given entities in a batch which means it will create a single {@link Query}. Assume that we will clear
  	 * the {@link javax.persistence.EntityManager} after the call.
  	 *
  	 * @param entities
  	 */
  	void deleteInBatch(Iterable<T> entities);
  	/**
  	 * Deletes all entities in a batch call.
  	 */
  	void deleteAllInBatch();
  	/**
  	 * Returns a reference to the entity with the given identifier. Depending on how the JPA persistence provider is
  	 * implemented this is very likely to always return an instance and throw an
  	 * {@link javax.persistence.EntityNotFoundException} on first access. Some of them will reject invalid identifiers
  	 * immediately.
  	 *
  	 * @param id must not be {@literal null}.
  	 * @return a reference to the entity with the given identifier.
  	 * @see EntityManager#getReference(Class, Object) for details on when an exception is thrown.
  	 */
  	T getOne(ID id);
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.query.QueryByExampleExecutor#findAll(org.springframework.data.domain.Example)
  	 */
  	@Override
  	<S extends T> List<S> findAll(Example<S> example);
  	/*
  	 * (non-Javadoc)
  	 * @see org.springframework.data.repository.query.QueryByExampleExecutor#findAll(org.springframework.data.domain.Example, org.springframework.data.domain.Sort)
  	 */
  	@Override
  	<S extends T> List<S> findAll(Example<S> example, Sort sort);
  }
  
  ```

* **Repository 的实现类 SimpleJpaRepository**

  * 关系数据库的所有 Repository 接口的实现类就是 SimpleJpaRepository

* Spring Data JPA 的最大特色是利用**方法名定义查询方法（Defining Query Methods）**来做 CRUD 操作

  * 一种是直接通过方法名就可以实现;

  * 另一种是 @Query 手动在方法上定义;

  * **方法的查询策略设置**

    * `@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)`
    * QueryLookupStrategy.Key 的值共 3 个
      * **Create**：直接根据方法名进行创建，规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候会报异常，这种情况可以理解为，即使配置了 @Query 也是没有用的。
      * **USE_DECLARED_QUERY**：声明方式创建，启动的时候会尝试找到一个声明的查询，如果没有找到将抛出一个异常，可以理解为必须配置 @Query。
      * **CREATE_IF_NOT_FOUND**：这个是默认的，除非有特殊需求，可以理解为这是以上 2 种方式的兼容版。先用声明方式（@Query）进行查找，如果没有找到与方法相匹配的查询，那用 Create 的方法名创建规则创建一个查询；这两者都不满足的情况下，启动就会报错。

  * Defining Query Method（DQM）语法

    * 该语法是：带查询功能的方法名由查询策略（关键字）+ 查询字段 + 一些限制性条件组成，具有语义清晰、功能完整的特性
    * org.springframework.data.repository.query.parser.PartTree 
    * org.springframework.data.repository.query.parser.Part
    * ![Lark20200918-182821.png](https://s0.lgstatic.com/i/image/M00/51/31/Ciqc1F9ki9CAPfoLAAMOpmuNPDY563.png)

  * **特定类型的参数：Sort 排序和 Pageable 分页**

    ```java
    Page<User> findByLastname(String lastname, Pageable pageable);//根据分页参数查询User，返回一个带分页结果的Page对象（方法一）
    方法一：允许将 org.springframework.data.domain.Pageable 实例传递给查询方法，将分页参数添加到静态定义的查询中，通过 Page 返回的结果得知可用的元素和页面的总数。这种分页查询方法可能是昂贵的（会默认执行一条 count 的 SQL 语句），所以用的时候要考虑一下使用场景。
    
    Slice<User> findByLastname(String lastname, Pageable pageable);//我们根据分页参数返回一个Slice的user结果（方法二）
    方法二：返回结果是 Slice，因为只知道是否有下一个 Slice 可用，而不知道 count，所以当查询较大的结果集时，只知道数据是足够的，也就是说用在业务场景中时不用关心一共有多少页。
    
    List<User> findByLastname(String lastname, Sort sort);//根据排序结果返回一个List（方法三）
    方法三：如果只需要排序，需在 org.springframework.data.domain.Sort 参数中添加一个参数，正如上面看到的，只需返回一个 List 也是有可能的。
    
    List<User> findByLastname(String lastname, Pageable pageable);//根据分页参数返回一个List对象（方法四）
    方法四：排序选项也通过 Pageable 实例处理，在这种情况下，Page 将不会创建构建实际实例所需的附加元数据（即不需要计算和查询分页相关数据），而仅仅用来做限制查询给定范围的实体。
    
    
    //查询user里面的lastname=jk的第一页，每页大小是20条；并会返回一共有多少页的信息
    Page<User> users = userRepository.findByLastname("jk",PageRequest.of(1, 20));
    //查询user里面的lastname=jk的第一页的20条数据，不知道一共多少条
    Slice<User> users = userRepository.findByLastname("jk",PageRequest.of(1, 20));
    //查询出来所有的user里面的lastname=jk的User数据，并按照name正序返回List
    List<User> users = userRepository.findByLastname("jk",new Sort(Sort.Direction.ASC, "name"))
    //按照createdAt倒序，查询前一百条User数据
    List<User> users = userRepository.findByLastname("jk",PageRequest.of(0, 100, Sort.Direction.DESC, "createdAt"));
    ```

  * **限制查询结果 First 和 Top**

    ```java
    User findFirstByOrderByLastnameAsc();
    User findTopByOrderByAgeDesc();
    List<User> findDistinctUserTop3ByLastname(String lastname, Pageable pageable);
    List<User> findFirst10ByLastname(String lastname, Sort sort);
    List<User> findTop10ByLastname(String lastname, Pageable pageable);
    ```

    * 查询方法在使用 First 或 Top 时，数值可以追加到 First 或 Top 后面，指定返回最大结果的大小；

    * 如果数字被省略，则假设结果大小为 1；

    * 限制表达式也支持 Distinct 关键字；

    * 支持将结果包装到 Optional 中。

    * 如果将 Pageable 作为参数，以 Top 和 First 后面的数字为准，即分页将在限制结果中应用。
    
  * @NonNull、@NonNullApi、@Nullable
  
    * 从 Spring Data 2.0 开始，JPA 新增了@NonNull @NonNullApi @Nullable，是对 null 的参数和返回结果做的支持。
  
    * @NonNullApi：在包级别用于声明参数，以及返回值的默认行为是不接受或产生空值的。
  
      ```java
      @org.springframework.lang.NonNullApi
      package com.myrespository;
      ```
  
    * @NonNull：用于不能为空的参数或返回值（在 @NonNullApi 适用的参数和返回值上不需要）
  
    * @Nullable：用于可以为空的参数或返回值。
  
      ```java
      //当我们添加@Nullable 注解之后，参数和返回结果这个时候就都会允许为 null 了；          
      @Nullable
      User findByEmailAddress(@Nullable EmailAddress emailAdress);
      //返回结果允许为 null,参数不允许为 null 的情况
      Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
      ```
  
* **Repository 的返回结果**

  * 它实现的方法，以及父类接口的方法和返回类型包括：Optional、Iterable、List、Page、Long、Boolean、Entity 对象等;

  * Spring Data 里面定义了一个特殊的子类 **Steamable**，Streamable 可以替代 Iterable 或任何集合类型。它还提供了方便的方法来访问 Stream，可以直接在元素上进行 ….filter(…) 和 ….map(…) 操作，并将 Streamable 连接到其他元素;

    ```java
    class Product { (1)
      MonetaryAmount getPrice() { … }
    }
    
    @RequiredArgConstructor(staticName = "of")
    class Products implements Streamable<Product> { (2)
      private Streamable<Product> streamable;
      public MonetaryAmount getTotal() { (3)
        return streamable.stream() //
          .map(Priced::getPrice)
          .reduce(Money.of(0), MonetaryAmount::add);
      }
    }
    
    interface ProductRepository implements Repository<Product, Long> {
      Products findAllByDescriptionContaining(String text); (4)
    }
    
    ```

    ```java
    //  Page<User>
    {
       "content":[],
       "pageable":{
          "sort":{
             "sorted":false,
             "unsorted":true,
             "empty":true
          },
          "pageNumber":0,当前页码
          "pageSize":3,页码大小
          "offset":0,偏移量
          "paged":true,是否分页了
          "unpaged":false
       },
       "totalPages":3,一共有多少页
       "last":false,是否是到最后
       "totalElements":7,一共多少调数
       "numberOfElements":3,当前数据下标
       "sort":{
          "sorted":false,
          "unsorted":true,
          "empty":true
       },
       "size":3,当前content大小
       "number":0,当前页面码的索引
       "first":true,是否是第一页
       "empty":false是否有数据
    }
    
    查询分页数据
    Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.email as email3_0_, user0_.name as name4_0_, user0_.sex as sex5_0_ from user user0_ limit ?
    计算分页数据
    Hibernate: select count(user0_.id) as col_0_0_ from user user0_
    ```

    ```java
    // Slice<User>
    {
       "content":[],
       "pageable":{
          "sort":{
             "sorted":false,
             "unsorted":true,
             "empty":true
          },
          "pageNumber":1,
          "pageSize":3,
          "offset":3,
          "paged":true,
          "unpaged":false
       },
       "numberOfElements":3,
       "sort":{
          "sorted":false,
          "unsorted":true,
          "empty":true
       },
       "size":3,
       "number":1,
       "first":false,
       "last":false,
       "empty":false
    }
    Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.email as email3_0_, user0_.name as name4_0_, user0_.sex as sex5_0_ from user user0_ limit ? offset ?
    
    ```

    * 只查询偏移量，不计算分页数据，这就是 Page 和 Slice 的主要区别。

  * **Repository 对 Feature/CompletableFuture 异步返回结果的支持：**
  
    * 可以使用 Spring 的异步方法执行Repository查询，这意味着方法将在调用时立即返回，并且实际的查询执行将发生在已提交给 Spring TaskExecutor 的任务中，比较适合定时任务的实际场景。异步使用起来比较简单，直接加@Async 注解即可，如下所示：
  
    ```java
    // 使用 java.util.concurrent.Future 的返回类型；
    @Async
    Future<User> findByFirstname(String firstname);
    // 使用 java.util.concurrent.CompletableFuture 作为返回类型；
    @Async
    CompletableFuture<User> findOneByFirstname(String firstname); 
    // 使用 org.springframework.util.concurrent.ListenableFuture 作为返回类型。
    @Async
    ListenableFuture<User> findOneByLastname(String lastname);
    ```
  
    * 以上是对 @Async 的支持，关于实际使用需要注意以下三点内容：
      - 在实际工作中，直接在 Repository 这一层使用异步方法的场景不多，一般都是把异步注解放在 Service 的方法上面，这样的话，可以有一些额外逻辑，如发短信、发邮件、发消息等配合使用；
      - 使用异步的时候一定要配置线程池，这点切记，否则“死”得会很难看；
      - 万一失败我们会怎么处理？关于事务是怎么处理的呢？这种需要重点考虑的;
  
  * ![Drawing 5.png](https://s0.lgstatic.com/i/image/M00/56/1D/Ciqc1F9rDAiARh9tAAQVFWlht1s532.png)
  
* Projections 的概念

  * 从字面意思上理解就是映射，指的是和 DB 的查询结果的字段映射关系。一般情况下，返回的字段和 DB 的查询结果的字段是一一对应的；但有的时候，需要返回一些指定的字段，或者返回一些复合型的字段，而不需要全部返回。

    ```java
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class User {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       private String email;
       private String sex;
       private String address;
    }
    
    ```

  * 如果我们只想返回 User 对象里面的 name 和 email，应该怎么做？下面我们介绍三种方法。

    * 第一种方法：新建一张表的不同 Entity

      ```java
      首先，我们新增一个Entity类：通过 @Table 指向同一张表，这张表和 User 实例里面的表一样都是 user，完整内容如下：
      @Entity
      @Table(name = "user")
      @Data
      @Builder
      @AllArgsConstructor
      @NoArgsConstructor
      public class UserOnlyNameEmailEntity {
         @Id
         @GeneratedValue(strategy= GenerationType.AUTO)
         private Long id;
         private String name;
         private String email;
      }
      
      然后，新增一个 UserOnlyNameEmailEntityRepository，做单独的查询：
      package com.example.jpa.example1;
      import org.springframework.data.jpa.repository.JpaRepository;
      public interface UserOnlyNameEmailEntityRepository extends JpaRepository<UserOnlyNameEmailEntity,Long> {
      }
      
      最后，我们的测试用例里面的写法如下：
      @Test
      public void testProjections() {
        userRepository.save(User.builder().id(1L).name("jack12").email("123456@126.com").sex("man").address("shanghai").build());
          List<User> users= userRepository.findAll();
          System.out.println(users);
          UserOnlyNameEmailEntity uName = userOnlyNameEmailEntityRepository.getOne(1L);
          System.out.println(uName);
      }
      
      缺点就是通过两个实体都可以进行 update 操作，如果同一个项目里面这种实体比较多，到时候就容易不知道是谁更新的，从而导致出 bug 不好查询，实体职责划分不明确。我们来看第二种返回 DTO 的做法。
      ```

    * 第二种方法：直接定义一个 UserOnlyNameEmailDto

      ```java
      首先，我们新建一个 DTO 类来返回我们想要的字段，它是 UserOnlyNameEmailDto，用来接收 name、email 两个字段的值，具体如下：
      @Data
      @Builder
      @AllArgsConstructor
      public class UserOnlyNameEmailDto {
          private String name,email;
      }
      
      其次，在 UserRepository 里面做如下用法：
      public interface UserRepositoryByDTO extends JpaRepository<User,Long> {
          //测试只返回name和email的DTO
          UserOnlyNameEmailDto findByEmail(String email);
      }
      
      然后，测试用例里面写法如下：
      @Test
      public void testProjections() {
      userRepository.save(User.builder().id(1L).name("jack12").email("123456@126.com").sex("man").address("shanghai").build());
              UserOnlyNameEmailDto userOnlyNameEmailDto =  userRepository.findByEmail("123456@126.com");
              System.out.println(userOnlyNameEmailDto);
          }
      
      这里需要注意的是，如果我们去看源码的话，看关键的 PreferredConstructorDiscoverer 类时会发现，UserDTO 里面只能有一个全参数构造方法
      
      Constructor 选择的时候会帮我们做构造参数的选择，如果 DTO 里面有多个构造方法，就会报转化错误的异常，这一点需要注意，异常是这样的：
      No converter found capable of converting from type [com.example.jpa.example1.User] to type [com.example.jpa.example1.UserOnlyNameEmailDto
      ```

      ![Drawing 7.png](https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDD6AWKs9AAE8kGFOxmo130.png)

    * 第三种方法：返回结果是一个 POJO 的接口

      ```java
      这种方式与上面两种的区别是只需要定义接口，它的好处是只读，不需要添加构造方法，我们使用起来非常灵活，一般很难产生 Bug，
      
      首先，定义一个 UserOnlyName 的接口：
      package com.example.jpa.example1;
      public interface UserOnlyName {
          String getName();
          String getEmail();
      }
      
      其次，我们的 UserRepository 写法如下：
      package com.example.jpa.example1;
      import org.springframework.data.jpa.repository.JpaRepository;
      public interface UserRepository extends JpaRepository<User,Long> {
          /**
           * 接口的方式返回DTO
           * @param address
           * @return
           */
          UserOnlyName findByAddress(String address);
      }
      然后，测试用例的写法如下：
          @Test
          public void testProjections() {
      userRepository.save(User.builder().name("jack12").email("123456@126.com").sex("man").address("shanghai").build());
              UserOnlyName userOnlyName = userRepository.findByAddress("shanghai");
              System.out.println(userOnlyName);
          }
      
      这个时候会发现我们的 userOnlyName 接口成了一个代理对象，里面通过 Map 的格式包含了我们的要返回字段的值（如：name、email），我们用的时候直接调用接口里面的方法即可，如 userOnlyName.getName() 即可；这种方式的优点是接口为只读，并且语义更清晰。
      ```

      ![Drawing 8.png](https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDEWAdoxmAARwd1XSUzo704.png)

* **@Query**

  * JpaQueryLookupStrategy 关键源码剖析

    * 先打开 QueryExecutorMethodInterceptor 类，默认的策略是CreateIfNotFound,找到如下代码：

      ![image-20210130223105474](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210130223105474.png)

  * @Query 的基本用法

    ```java
    package org.springframework.data.jpa.repository;
    public @interface Query {
       /**
        * 指定JPQL的查询语句。（nativeQuery=true的时候，是原生的Sql语句）
    	*/
       String value() default "";
       /**
    	* 指定count的JPQL语句，如果不指定将根据query自动生成。
        * （如果当nativeQuery=true的时候，指的是原生的Sql语句）
        */
       String countQuery() default "";
       /**
        * 根据哪个字段来count，一般默认即可。
    	*/
       String countProjection() default "";
       /**
        * 默认是false，表示value里面是不是原生的sql语句
    	*/
       boolean nativeQuery() default false;
       /**
        * 可以指定一个query的名字，必须唯一的。
    	* 如果不指定，默认的生成规则是：
        * {$domainClass}.${queryMethodName}
        */
       String name() default "";
       /*
        * 可以指定一个count的query的名字，必须唯一的。
    	* 如果不指定，默认的生成规则是：
        * {$domainClass}.${queryMethodName}.count
        */
       String countName() default "";
    }
    ```

  * @Query 用法是使用 JPQL 为实体创建声明式查询方法。我们一般只需要关心 @Query 里面的 value 和 nativeQuery、countQuery 的值即可

  * JPQL的语法

    ```
    查询:
    SELECT ... FROM ...
    [WHERE ...]
    [GROUP BY ... [HAVING ...]]
    [ORDER BY ...]
    它的语法结构有点类似我们 SQL,唯一的区别就是 JPQL FROM 后面跟的是对象，而 SQL 里面的字段对应的是对象里面的属性字段。
    
    update 和 delete 的语法结构
    DELETE FROM ... [WHERE ...]
    UPDATE ... SET ... [WHERE ...]
    ```

  * 语法参考文档: https://docs.oracle.com/html/E13946_04/ejb3_langref.html

  * 示例

    ```java
    案例 1： 要在 Repository 的查询方法上声明一个注解，这里就是 @Query 注解标注的地方。
    public interface UserRepository extends JpaRepository<User, Long>{
      @Query("select u from User u where u.emailAddress = ?1")
      User findByEmailAddress(String emailAddress);
    }
    
    案例 2： LIKE 查询，注意 firstname 不会自动加上“%”关键字。
    public interface UserRepository extends JpaRepository<User, Long> {
      @Query("select u from User u where u.firstname like %?1")
      List<User> findByFirstnameEndsWith(String firstname);
    }
    
    案例 3： 直接用原始 SQL，nativeQuery = true 即可。
    public interface UserRepository extends JpaRepository<User, Long> {
      @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
      User findByEmailAddress(String emailAddress);
    }
    注意：nativeQuery 不支持直接 Sort 的参数查询。
    
    案例 4： 下面是nativeQuery 的排序错误的写法，会导致无法启动。
    public interface UserRepository extends JpaRepository<User, Long> {
    @Query(value = "select * from user_info where first_name=?1",nativeQuery = true)
    List<UserInfoEntity> findByFirstName(String firstName,Sort sort);
    }
    
    案例 5： nativeQuery 排序的正确写法。
    @Query(value = "select * from user_info where first_name=?1 order by ?2",nativeQuery = true)
    List<UserInfoEntity> findByFirstName(String firstName,String sort);
    //调用的地方写法last_name是数据里面的字段名，不是对象的字段名
    repository.findByFirstName("jackzhang","last_name");
    ```

  * @Query 的排序

    * @Query中在用JPQL的时候，想要实现排序，方法上直接用 PageRequest 或者 Sort 参数都可以做到。

    * `state_field_path_expression JPQL`的定义，并且 Sort 的对象支持一些特定的函数。

      ```java
      案例 6： Sort and JpaSort 的使用，它可以进行排序。
      public interface UserRepository extends JpaRepository<User, Long> {
        @Query("select u from User u where u.lastname like ?1%")
        List<User> findByAndSort(String lastname, Sort sort);
      
        @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
        List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);
      
      }
      
      //调用方的写法，如下：
      repo.findByAndSort("lannister", new Sort("firstname"));
      repo.findByAndSort("stark", new Sort("LENGTH(firstname)"));
      repo.findByAndSort("targaryen", JpaSort.unsafe("LENGTH(firstname)"));
      repo.findByAsArrayAndSort("bolton", new Sort("fn_len"));
      
      ```

  * @Query 的分页

    * @Query 的分页分为两种情况，分别为 JPQL 的排序和 nativeQuery 的排序。

      ```java
      案例 7：直接用 Page 对象接受接口，参数直接用 Pageable 的实现类即可。
      public interface UserRepository extends JpaRepository<User, Long> {
        @Query(value = "select u from User u where u.lastname = ?1")
        Page<User> findByLastname(String lastname, Pageable pageable);
      }
      //调用者的写法
      repository.findByFirstName("jackzhang",new PageRequest(1,10));
      
      案例 8：@Query 对原生 SQL 的分页支持，并不是特别友好，因为这种写法比较“骇客”，可能随着版本的不同会有所变化。我们以 MySQL 为例。
      这里需要注意：这个注释 / #pageable# / 必须有。
       public interface UserRepository extends JpaRepository<UserInfoEntity, Integer>, JpaSpecificationExecutor<UserInfoEntity> {
      
         @Query(value = "select * from user_info where first_name=?1 /* #pageable# */",
               countQuery = "select count(*) from user_info where first_name=?1",
               nativeQuery = true)
         Page<UserInfoEntity> findByFirstName(String firstName, Pageable pageable);
      
      }
      
      //调用者的写法
      return userRepository.findByFirstName("jackzhang",new PageRequest(1,10, Sort.Direction.DESC,"last_name"));
      
      //打印出来的sql
      
      select  *   from  user_info  where  first_name=? /* #pageable# */  order by  last_name desc limit ?, ?
      
      ```

  * @Param 用法

    * @Param 注解指定方法参数的具体名称，通过绑定的参数名字指定查询条件，这样不需要关心参数的顺序。

  * @Query 之 Projections 应用返回指定 DTO

    ```java
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class UserExtend { //用户扩展信息表
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private Long userId;
       private String idCard;
       private Integer ages;
       private String studentNumber;
    }
    
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class User { //用户基本信息表
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       private String email;
       @Version
       private Long version;
       private String sex;
       private String address;
    }
    如果我们想定义一个 DTO 对象，里面只要 name、email、idCard，这个时候我们怎么办呢？这种场景非常常见
        
    一 利用 `class UserDto` 获取我们想要的结果,首先，我们新建一个 UserDto 类的内容。
    package com.example.jpa.example1;
    import lombok.AllArgsConstructor;
    import lombok.Builder;
    import lombok.Data;
    @Data
    @Builder
    @AllArgsConstructor
    public class UserDto {
        private String name,email,idCard;
    } 
    public interface UserDtoRepository extends JpaRepository<User, Long> {
       @Query("select new com.example.jpa.example1.UserDto(CONCAT(u.name,'JK123'),u.email,e.idCard) from User u,UserExtend e where u.id= e.userId and u.id=:id")
       UserDto findByUserDtoId(@Param("id") Long id);
    }
    @Test
    public void testQueryAnnotationDto() {
      userDtoRepository.save(User.builder().name("jack").email("123456@126.com").sex("man").address("shanghai").build());
      userExtendRepository.save(UserExtend.builder().userId(1L).idCard("shengfengzhenghao").ages(18).studentNumber("xuehao001").build());
       UserDto userDto = userDtoRepository.findByUserDtoId(1L);
       System.out.println(userDto);
    }
    
    二 利用 UserDto 接口获得我们想要的结果
    首先，新增一个 UserSimpleDto 接口来得到我们想要的 name、email、idCard 信息。
    package com.example.jpa.example1;
    public interface UserSimpleDto {
       String getName();
       String getEmail();
       String getIdCard();
    }
    其次，在 UserDtoRepository 里面新增一个方法，返回结果是 UserSimpleDto 接口。
    public interface UserDtoRepository extends JpaRepository<User, Long> {
    //利用接口DTO获得返回结果，需要注意的是每个字段需要as和接口里面的get方法名字保持一样
    @Query("select CONCAT(u.name,'JK123') as name,UPPER(u.email) as email ,e.idCard as idCard from User u,UserExtend e where u.id= e.userId and u.id=:id")
    UserSimpleDto findByUserSimpleDtoId(@Param("id") Long id);
    }
    然后，测试用例写法如下。
    @Test
    public void testQueryAnnotationDto() {
       userDtoRepository.save(User.builder().name("jack").email("123456@126.com").sex("man").address("shanghai").build());
      userExtendRepository.save(UserExtend.builder().userId(1L).idCard("shengfengzhenghao").ages(18).studentNumber("xuehao001").build());
       UserSimpleDto userDto = userDtoRepository.findByUserSimpleDtoId(1L);
       System.out.println(userDto);  System.out.println(userDto.getName()+":"+userDto.getEmail()+":"+userDto.getIdCard());
    }
    
    ```

    * 打开 `org.hibernate.query.criteria.internal.expression.function.ParameterizedFunctionExpression` 会发现 Hibernate 支持的关键字有这么多

  * @Query 动态查询解决方法

    ```java
    实现 @Query 的动态参数查询。
    首先，新增一个 UserOnlyName 接口，只查询 User 里面的 name 和 email 字段。
    package com.example.jpa.example1;
    //获得返回结果
    public interface UserOnlyName {
        String getName();
        String getEmail();
    }
    其次，在我们的 UserDtoRepository 里面新增两个方法：一个是利用 JPQL 实现动态查询，一个是利用原始 SQL 实现动态查询。
    package com.example.jpa.example1;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;
    import java.util.List;
    public interface UserDtoRepository extends JpaRepository<User, Long> {
       /**
        * 利用JQPl动态查询用户信息
        * @param name
        * @param email
        * @return UserSimpleDto接口
        */
       @Query("select u.name as name,u.email as email from User u where (:name is null or u.name =:name) and (:email is null or u.email =:email)")
       UserOnlyName findByUser(@Param("name") String name,@Param("email") String email);
       /**
        * 利用原始sql动态查询用户信息
        * @param user
        * @return
        */
       @Query(value = "select u.name as name,u.email as email from user u where (:#{#user.name} is null or u.name =:#{#user.name}) and (:#{#user.email} is null or u.email =:#{#user.email})",nativeQuery = true)
       UserOnlyName findByUser(@Param("user") User user);
    }
    
    然后，我们新增一个测试类，测试一下上面方法的结果。
    @Test
    public void testQueryDinamicDto() {
     userDtoRepository.save(User.builder().name("jack").email("123456@126.com").sex("man").address("shanghai").build());
       UserOnlyName userDto = userDtoRepository.findByUser("jack", null);
       System.out.println(userDto.getName() + ":" + userDto.getEmail());
       UserOnlyName userDto2 = userDtoRepository.findByUser(User.builder().email("123456@126.com").build());
       System.out.println(userDto2.getName() + ":" + userDto2.getEmail());
    }
    
    ```

    ![Drawing 5.png](https://s0.lgstatic.com/i/image/M00/56/57/CgqCHl9rKauAfQmmAAHMquqHBVg909.png)

    ![Drawing 6.png](https://s0.lgstatic.com/i/image/M00/56/4C/Ciqc1F9rKbGAT_6FAAL_YUkV6AQ306.png)
  
* **@Entity**

  * JPA 协议中关于 Entity 的相关规定

    * 实体是直接进行数据库持久化操作的领域对象（即一个简单的 POJO，可以按照业务领域划分），必须通过 @Entity 注解进行标示。

    * 实体必须有一个 public 或者 protected 的无参数构造方法。

    * 持久化映射的注解可以标示在 Entity 的字段 field 上

      ```java
      @Column(length = 20, nullable = false)
      private String userName;
      ```

    * 也可以将持久化注解运用在 Entity 里面的 get/set 方法上，通常我们是放在 get 方法中，如下所示：

      ```java
      @Column(length = 20, nullable = false)
      public String getUserName(){
          return userName;
      }
      ```

  * **概括起来，就是 Entity 里面的注解生效只有两种方式：将注解写在字段上或者将注解写在方法上（JPA 里面称 Property）。需要注意的是，在同一个 Entity 里面只能有一种方式生效，也就是说，注解要么全部写在 field 上面，要么就全部写在 Property 上面**

    * 只要是在 @Entity 的实体里面被注解标注的字段，都会被映射到数据库中，除了使用 @Transient 注解的字段之外。
    * 实体里面必须要有一个主键，主键标示的字段可以是单个字段，也可以是复合主键字段。

  * **@Entity** 用于定义对象将会成为被 JPA 管理的实体，必填，将字段映射到指定的数据库表中，使用起来很简单，直接用在实体类上面即可，通过源码表达的语法如下:

    ```java
    @Target(TYPE) //表示此注解只能用在class上面
    public @interface Entity {
       //可选，默认是实体类的名字，整个应用里面全局唯一。
       String name() default "";
    }
    ```

  * **@Table** 用于指定数据库的表名，表示此实体对应的数据库里面的表名，非必填，默认表名和 entity 名字一样。

    ```java
    @Target(TYPE) //一样只能用在类上面
    public @interface Table {
       //表的名字，可选。如果不填写，系统认为好实体的名字一样为表名。
       String name() default "";
       //此表所在schema，可选
       String schema() default "";
       //唯一性约束，在创建表的时候有用，表创建之后后面就不需要了。
       UniqueConstraint[] uniqueConstraints() default { };
       //索引，在创建表的时候使用，表创建之后后面就不需要了。
       Index[] indexes() default {};
    }
    ```

  * **@Access** 用于指定 entity 里面的注解是写在字段上面，还是 get/set 方法上面生效，非必填。在默认不填写的情况下，当实体里面的第一个注解出现在字段上或者 get/set 方法上面，就以第一次出现的方式为准；也就是说，一个实体里面的注解既有用在 field 上面，又有用在 properties 上面的时候.

    ```java
    @Id
    private Long id;
    @Column(length = 20, nullable = false)
    public String getUserName(){
        return userName;
    }
    那么由于 @Id 是实体里面第一个出现的注解，并且作用在字段上面，所以所有写在 get/set 方法上面的注解就会失效。而 @Access 可以干预默认值，指定是在 fileds 上面生效还是在 properties 上面生效
    
    //表示此注解可以运用在class上(那么这个时候就可以指定此实体的默认注解生效策略了)，也可以用在方法上或者字段上(表示可以独立设置某一个字段或者方法的生效策略)；
    @Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Access {
        //指定是字段上面生效还是方法上面生效
        AccessType value();
    }
    public enum AccessType {
        FIELD,
        PROPERTY;
    
        private AccessType() {
        }
    }
    ```

  * **@Id** 定义属性为数据库的主键，一个实体里面必须有一个主键，但不一定是这个注解，可以和 @GeneratedValue 配合使用或成对出现。

  * **@GeneratedValue** 主键生成策略

    ```java
    public @interface GeneratedValue {
        //Id的生成策略
        GenerationType strategy() default AUTO;
        //通过Sequences生成Id,常见的是Orcale数据库ID生成规则，这个时候需要配合@SequenceGenerator使用
        String generator() default "";
    }
    public enum GenerationType {
        //通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。
        TABLE,
        //通过序列产生主键，通过 @SequenceGenerator 注解指定序列名， MySql 不支持这种方式；
        SEQUENCE,
        //采用数据库ID自增长， 一般用于mysql数据库
        IDENTITY,
    	//JPA 自动选择合适的策略，是默认选项；
        AUTO
    }
    ```

  * **@Enumerated** 这个注解很好用，因为它对 enum 提供了下标和 name 两种方式，用法直接映射在 enum 枚举类型的字段上。

    ```java
    //作用在方法和字段上
    @Target({METHOD, FIELD})
    public @interface Enumerated {
    //枚举映射的类型，默认是ORDINAL（即枚举字段的下标）。
        EnumType value() default ORDINAL;
    }
    public enum EnumType {
        //映射枚举字段的下标
        ORDINAL,
        //映射枚举的Name
        STRING
    }
    经验分享： 如果我们用 @Enumerated（EnumType.ORDINAL），这时候数据库里面的值是 0、1。但是实际工作中，不建议用数字下标，因为枚举里面的属性值是会不断新增的，如果新增一个，位置变化了就惨了。并且 0、1、2 这种下标在数据库里面看着非常痛苦，时间长了就会一点也看不懂了。
    ```

  * **@Basic** 表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为 @Basic。也就是说默认所有的字段肯定是和数据库进行映射的，并且默认为 Eager 类型。

    ```java
    public @interface Basic {
        //可选，EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面）
        FetchType fetch() default EAGER;
        //可选。这个字段是否可以为null，默认是true。
        boolean optional() default true;
    }
    ```

  * **@Transient** 表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 有相反的作用。也就是每个字段上面 @Transient 和 @Basic 必须二选一，而什么都不指定的话，默认是 @Basic。

  * **@Column** 定义该属性对应数据库中的列名。

    ```java
    public @interface Column {
        //数据库中的表的列名；可选，如果不填写认为字段名和实体属性名一样。
        String name() default "";
        //是否唯一。默认flase，可选。
        boolean unique() default false;
        //数据字段是否允许空。可选，默认true。
        boolean nullable() default true;
        //执行insert操作的时候是否包含此字段，默认，true，可选。
        boolean insertable() default true;
        //执行update的时候是否包含此字段，默认，true，可选。
        boolean updatable() default true;
        //表示该字段在数据库中的实际类型。
        String columnDefinition() default "";
       //数据库字段的长度，可选，默认255
        int length() default 255;
    }
    ```

  * **@Temporal** 用来设置 Date 类型的属性映射到对应精度的字段，存在以下三种情况

    * @Temporal(TemporalType.DATE)映射为日期 // date （**只有日期**）

    * @Temporal(TemporalType.TIME)映射为日期 // time （**只有时间**）

    * @Temporal(TemporalType.TIMESTAMP)映射为日期 // date time （**日期+时间**）

  * ```java
    package com.example.jpa.example1;
    
    import lombok.Data;
    import javax.persistence.*;
    import java.util.Date;
    
    @Entity
    @Table(name = "user_topic")
    @Access(AccessType.FIELD)
    @Data
    public class UserTopic {
    
       @Id
       @Column(name = "id", nullable = false)
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
       
       @Column(name = "title", nullable = true, length = 200)
       private String title;
    
       @Basic
       @Column(name = "create_user_id", nullable = true)
       private Integer createUserId;
    
       @Basic(fetch = FetchType.LAZY)
       @Column(name = "content", nullable = true, length = -1)
       @Lob
       private String content;
    
       @Basic(fetch = FetchType.LAZY)
       @Column(name = "image", nullable = true)
       @Lob
       private byte[] image;
    
       @Basic
       @Column(name = "create_time", nullable = true)
       @Temporal(TemporalType.TIMESTAMP)
       private Date createTime;
       
       @Basic
       @Column(name = "create_date", nullable = true)
       @Temporal(TemporalType.DATE)
       private Date createDate;
    
       @Enumerated(EnumType.STRING)
       @Column(name = "topic_type")
       private Type type;
       
       @Transient
       private String transientSimple;
       //非数据库映射字段，业务类型的字段
       public String getTransientSimple() {
          return title + "auto:jack" + type;
       }
       //有一个枚举类，主题的类型
       public enum Type {
          EN("英文"), CN("中文");
          private final String des;
          Type(String des) {
             this.des = des;
          }
       }
    }
    ```

  * 联合主键

    * 可以通过 javax.persistence.EmbeddedId 和 javax.persistence.IdClass 两个注解实现联合主键的效果。

      ```java
      第一步：新建一个 UserInfoID 类里面是联合主键。
      package com.example.jpa.example1;
      import lombok.AllArgsConstructor;
      import lombok.Builder;
      import lombok.Data;
      import lombok.NoArgsConstructor;
      import java.io.Serializable;
      
      @Data
      @Builder
      @AllArgsConstructor
      @NoArgsConstructor
      public class UserInfoID implements Serializable {
         private String name,telephone;
      }
      
      第二步：再新建一个 UserInfo 的实体，采用 @IdClass 引用联合主键类。
      @Entity
      @Data
      @Builder
      @IdClass(UserInfoID.class)
      @AllArgsConstructor
      @NoArgsConstructor
      public class UserInfo {
         private Integer ages;
         @Id
         private String name;
         @Id
         private String telephone;
      }
      
      第三步：新增一个 UserInfoReposito 类来做 CRUD 操作。
      package com.example.jpa.example1;
      import org.springframework.data.jpa.repository.JpaRepository;
      public interface UserInfoRepository extends JpaRepository<UserInfo,UserInfoID> {
      }
      
      第四步：写一个测试用例，测试一下。
      package com.example.jpa.example1;
      import org.junit.jupiter.api.Test;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
      import java.util.Optional;
      @DataJpaTest
      public class UserInfoRepositoryTest {
         @Autowired
         private UserInfoRepository userInfoRepository;
         @Test
         public void testIdClass() {
         userInfoRepository.save(UserInfo.builder().ages(1).name("jack").telephone("123456789").build());
            Optional<UserInfo> userInfo = 		userInfoRepository.findById(UserInfoID.builder().name("jack").telephone("123456789").build());
            System.out.println(userInfo.get());
         }
      }
      Hibernate: create table user_info (name varchar(255) not null, telephone varchar(255) not null, ages integer, primary key (name, telephone))
      Hibernate: select userinfo0_.name as name1_3_0_, userinfo0_.telephone as telephon2_3_0_, userinfo0_.ages as ages3_3_0_ from user_info userinfo0_ where userinfo0_.name=? and userinfo0_.telephone=?
      UserInfo(ages=1, name=jack, telephone=123456789)
      ```

    * **@Embeddable 与 @EmbeddedId 注解使用**

      ```java
      第一步：在我们上面例子中的 UserInfoID 里面添加 @Embeddable 注解。
      @Data
      @Builder
      @AllArgsConstructor
      @NoArgsConstructor
      @Embeddable
      public class UserInfoID implements Serializable {
         private String name,telephone;
      }
      
      第二步：改一下我们刚才的 User 对象，删除 @IdClass，添加 @EmbeddedId 注解，如下：
      @Entity
      @Data
      @Builder
      @AllArgsConstructor
      @NoArgsConstructor
      public class UserInfo {
         private Integer ages;
         @EmbeddedId
         private UserInfoID userInfoID;
         @Column(unique = true)
         private String uniqueNumber;
      }
      
      第三步：UserInfoRepository 不变，我们直接修改一下测试用例。
      @Test
      public void testIdClass() {
       userInfoRepository.save(UserInfo.builder().ages(1).userInfoID(UserInfoID.builder().name("jack").telephone("123456789").build()).build());
         Optional<UserInfo> userInfo = userInfoRepository.findById(UserInfoID.builder().name("jack").telephone("123456789").build());
         System.out.println(userInfo.get());
      }
      ```

    * @IdClass 和 @EmbeddedId 的区别是什么？

      * 如上面测试用例，在使用的时候，Embedded 用的是对象，而 IdClass 用的是具体的某一个字段；

      * 二者的JPQL 也会不一样：

        *  用 @IdClass JPQL 的写法：SELECT u.name FROM UserInfo u
        * 用 @EmbeddedId 的 JPQL 的写法：select u.userInfoId.name FROM UserInfo u

      * 联合主键还有需要注意的就是，它与唯一性索引约束的区别是写法不同，如上面所讲，唯一性索引的写法如下：

        ```java
        @Column(unique = true)
        private String uniqueNumber;
        ```

  * 实体之间的继承关系如何实现？

    * 在 Java 面向对象的语言环境中，@Entity 之间的关系多种多样，而根据 JPA 的规范，我们大致可以将其分为以下几种：

      * 纯粹的继承，和表没关系，对象之间的字段共享。利用注解 @MappedSuperclass，协议规定父类不能是 @Entity。
      * 单表多态问题，同一张 Table，表示了不同的对象，通过一个字段来进行区分。利用`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`注解完成，只有父类有 @Table。
      * 多表多态，每一个子类一张表，父类的表拥有所有公用字段。通过`@Inheritance(strategy = InheritanceType.JOINED)`注解完成，父类和子类都是表，有公用的字段在父表里面。
      * Object 的继承，数据库里面每一张表是分开的，相互独立不受影响。通过`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`注解完成，父类（可以是一张表，也可以不是）和子类都是表，相互之间没有关系。

    * **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**

      * 父类实体对象与各个子实体对象共用一张表，通过一个字段的不同值代表不同的对象。

        ```java
        我们抽象一个 Book 对象，如下所示：
        package com.example.jpa.example1.book;
        import lombok.Data;
        import javax.persistence.*;
        @Entity(name="book")
        @Data
        @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
        @DiscriminatorColumn(name="color", discriminatorType = DiscriminatorType.STRING)
        public class Book {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private String title;
        }
        
        再新建一个 BlueBook 对象，作为 Book 的子对象。
        package com.example.jpa.example1.book;
        import lombok.Data;
        import lombok.EqualsAndHashCode;
        import javax.persistence.DiscriminatorValue;
        import javax.persistence.Entity;
        @Entity
        @Data
        @EqualsAndHashCode(callSuper=false)
        @DiscriminatorValue("blue")
        public class BlueBook extends Book{
           private String blueMark;
        }
        
        再新建一个 RedBook 对象，作为 Book 的另一子对象。
        //红皮书
        @Entity
        @DiscriminatorValue("red")
        @Data
        @EqualsAndHashCode(callSuper=false)
        public class RedBook extends Book {
           private String redMark;
        }
        
        这时，我们一共新建了三个 Entity 对象，其实都是指 book 这一张表，通过 book 表里面的 color 字段来区分红书还是绿书。我们继续做一下测试看看结果。
        我们再新建一个 RedBookRepositor 类，操作一下 RedBook 会看到如下结果：
        package com.example.jpa.example1.book;
        import org.springframework.data.jpa.repository.JpaRepository;
        public interface RedBookRepository extends JpaRepository<RedBook,Long>{
        }
        然后再新建一个测试用例。
        package com.example.jpa.example1;
        import com.example.jpa.example1.book.RedBook;
        import com.example.jpa.example1.book.RedBookRepository;
        import org.junit.jupiter.api.Test;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
        @DataJpaTest
        public class RedBookRepositoryTest {
           @Autowired
           private RedBookRepository redBookRepository;
           @Test
           public void testRedBook() {
              RedBook redBook = new RedBook();
              redBook.setTitle("redbook");
              redBook.setRedMark("redmark");
              redBook.setId(1L);
              redBookRepository.saveAndFlush(redBook);
              RedBook r = redBookRepository.findById(1L).get();
            System.out.println(r.getId()+":"+r.getTitle()+":"+r.getRedMark());
           }
        }
        Hibernate: create table book (color varchar(31) not null, id bigint not null, title varchar(255), blue_mark varchar(255), red_mark varchar(255), primary key (id))
        Hibernate: insert into book (title, red_mark, color, id) values (?, ?, 'red', ?)
        ```

    * **@Inheritance(strategy = InheritanceType.JOINED)**

      * 在这种映射策略里面，继承结构中的每一个实体（entity）类都会映射到数据库里一个单独的表中。也就是说，每个实体（entity）都会被映射到数据库中，一个实体（entity）类对应数据库中的一个表。

      * 其中根实体（root entity）对应的表中定义了主键（primary key），所有的子类对应的数据库表都要共同使用 Book 里面的 @ID 这个主键。

        ```java
        首先，我们改一下上面的三个实体，测试一下InheritanceType.JOINED，改动如下：
        package com.example.jpa.example1.book;
        import lombok.Data;
        import javax.persistence.*;
        @Entity(name="book")
        @Data
        @Inheritance(strategy = InheritanceType.JOINED)
        public class Book {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private String title;
        }
        
        其次，我们 Book 父类、改变 Inheritance 策略、删除 DiscriminatorColumn，你会看到如下结果。
        package com.example.jpa.example1.book;
        import lombok.Data;
        import lombok.EqualsAndHashCode;
        import javax.persistence.Entity;
        import javax.persistence.PrimaryKeyJoinColumn;
        @Entity
        @Data
        @EqualsAndHashCode(callSuper=false)
        @PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")
        public class BlueBook extends Book{
           private String blueMark;
        }
        package com.example.jpa.example1.book;
        import lombok.Data;
        import lombok.EqualsAndHashCode;
        import javax.persistence.Entity;
        import javax.persistence.PrimaryKeyJoinColumn;
        @Entity
        @PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")
        @Data
        @EqualsAndHashCode(callSuper=false)
        public class RedBook extends Book {
           private String redMark;
        }
        然后，BlueBook和RedBook也删除DiscriminatorColumn，新增@PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")，和 book 父类共用一个主键值，而 RedBookRepository 和测试用例不变
        Hibernate: create table blue_book (blue_mark varchar(255), book_id bigint not null, primary key (book_id))
        Hibernate: create table book (id bigint not null, title varchar(255), primary key (id))
        Hibernate: create table red_book (red_mark varchar(255), book_id bigint not null, primary key (book_id))
        Hibernate: alter table blue_book add constraint FK9uuwgq7a924vtnys1rgiyrlk7 foreign key (book_id) references book
        Hibernate: alter table red_book add constraint FKk8rvl61bjy9lgsr9nhxn5soq5 foreign key (book_id) references book
        ```

    * **@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)**

      * 我们在使用 @MappedSuperClass 主键的时候，如果不指定 @Inhertance，默认就是此种TABLE_PER_CLASS模式。当然了，我们也显示指定，要求继承基类的都是一张表，而父类不是表，是 java 对象的抽象类。我们看一个例子。

        ```java
        首先，还是改一下上面的三个实体。
        package com.example.jpa.example1.book;
        import lombok.Data;
        import javax.persistence.*;
        @Entity(name="book")
        @Data
        @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
        public class Book {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private String title;
        }
        
        其次，Book 表采用 TABLE_PER_CLASS 策略，其子实体类都代表各自的表，实体代码如下：
        package com.example.jpa.example1.book;
        import lombok.Data;
        import lombok.EqualsAndHashCode;
        import javax.persistence.Entity;
        @Entity
        @Data
        @EqualsAndHashCode(callSuper=false)
        public class RedBook extends Book {
           private String redMark;
        }
        package com.example.jpa.example1.book;
        import lombok.Data;
        import lombok.EqualsAndHashCode;
        import javax.persistence.Entity;
        @Entity
        @Data
        @EqualsAndHashCode(callSuper=false)
        public class BlueBook extends Book{
           private String blueMark;
        }
        
        这时，从 RedBook 和 BlueBook 里面去掉 PrimaryKeyJoinColumn，而 RedBookRepository 和测试用例不变，我们执行看一下结果。
        Hibernate: create table blue_book (id bigint not null, title varchar(255), blue_mark varchar(255), primary key (id))
        Hibernate: create table book (id bigint not null, title varchar(255), primary key (id))
        Hibernate: create table red_book (id bigint not null, title varchar(255), red_mark varchar(255), primary key (id))
        ```

  * 实体与实体之间的关联关系

    * 实体与实体之间的关联关系一共分为四种，分别为 OneToOne、OneToMany、ManyToOne 和 ManyToMany；而实体之间的关联关系又分为双向的和单向的。

    * **@OneToOne 关联关系**

      * @OneToOne 一般表示对象之间一对一的关联关系，它可以放在 field 上面，也可以放在 get/set 方法上面。其中 JPA 协议有规定，如果是配置双向关联，维护关联关系的是拥有外键的一方，而另一方必须配置 mappedBy；如果是单项关联，直接配置在拥有外键的一方即可。

        ```java
        举个例子：user 表是用户的主信息，user_info 是用户的扩展信息，两者之间是一对一的关系。user_info 表里面有一个 user_id 作为关联关系的外键，如果是单项关联，我们的写法如下
        package com.example.jpa.example1;
        import lombok.AllArgsConstructor;
        import lombok.Builder;
        import lombok.Data;
        import lombok.NoArgsConstructor;
        import javax.persistence.*;
        @Entity
        @Data
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor
        public class User {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private String name;
           private String email;
           private String sex;
           private String address;
        }
        
        User 实体里面什么都没变化，不需要添加 @OneToOne 注解。我们只需要在拥有外键的一方配置就可以，所以 UserInfo 的代码如下：
        package com.example.jpa.example1;
        import lombok.*;
        import javax.persistence.*;
        @Entity
        @Data
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor
        @ToString(exclude = "user")
        public class UserInfo {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private Integer ages;
           private String telephone;
           @OneToOne //维护user的外键关联关系，配置一对一
           private User user;
        }
        我们看到，UserInfo 实体对象里面添加了 @OneToOne 注解，这时我们写一个测试用例跑一下看看有什么效果：
        Hibernate: create table user (id bigint not null, address varchar(255), email varchar(255), name varchar(255), sex varchar(255), primary key (id))
        Hibernate: create table user_info (id bigint not null, ages integer, telephone varchar(255), user_id bigint, primary key (id))
        Hibernate: alter table user_info add constraint FKn8pl63y4abe7n0ls6topbqjh2 foreign key (user_id) references user
        
        那么双向关联应该怎么配置呢？我们保持 UserInfo 不变，在 User 实体对象里面添加这一段代码即可。
        @OneToOne(mappedBy = "user")
        private UserInfo userInfo;
        
        完整的 User 实体对象就会变成如下模样。
        @Entity
        @Data
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor
        public class User {
           @Id
           @GeneratedValue(strategy= GenerationType.AUTO)
           private Long id;
           private String name;
           private String email;
           @OneToOne(mappedBy = "user")
           private UserInfo userInfo;//变化之处
           private String sex;
           private String address;
        }
        ```

    * @interface OneToOne 源码解读

      ```java
      public @interface OneToOne {
          //表示关系目标实体，默认该注解标识的返回值的类型的类。
          Class targetEntity() default void.class;
          //cascade 级联操作策略，就是我们常说的级联操作
          CascadeType[] cascade() default {};
          //数据获取方式EAGER(立即加载)/LAZY(延迟加载)
          FetchType fetch() default EAGER;
          //是否允许为空，默认是可选的，也就表示可以为空；
          boolean optional() default true;
          //关联关系被谁维护的一方对象里面的属性名字。 双向关联的时候必填
          String mappedBy() default "";
          //当被标识的字段发生删除或者置空操作之后，是否同步到关联关系的一方，即进行通过删除操作，默认flase，注意与CascadeType.REMOVE 级联删除的区别
          boolean orphanRemoval() default false;
      }
      ```

    * mappedBy 注意事项

      > 只有关联关系的维护方才能操作两个实体之间外键的关系。被维护方即使设置了维护方属性进行存储也不会更新外键关联。
      >
      > mappedBy 不能与 @JoinColumn 或者 @JoinTable 同时使用，因为没有意义，关联关系不在这里面维护。
      >
      > 此外，mappedBy 的值是指另一方的实体里面属性的字段，而不是数据库字段，也不是实体的对象的名字。也就是维护关联关系的一方属性字段名称，或者加了 @JoinColumn / @JoinTable 注解的属性字段名称。如上面的 User 例子 user 里面 mappedBy 的值，就是 UserInfo 里面的 user 字段的名字。

    * CascadeType用法

      * 在 CascadeType 的用法中，CascadeType 的枚举值只有五个，分别如下：

        * CascadeType.PERSIST 级联新建
        * CascadeType.REMOVE 级联删除
        * CascadeType.REFRESH 级联刷新
        * CascadeType.MERGE 级联更新
        * CascadeType.ALL 四项全选

      * 其中，默认是没有级联操作的，关系表不会产生任何影响。此外，JPA 2.0 还新增了 CascadeType.DETACH，即级联实体到 Detach 状态。

        ```java
        public class UserInfo {
            ...
           @OneToOne(cascade ={CascadeType.PERSIST,CascadeType.REMOVE})
           private User user;
        }
        ```

    * orphanRemoval 属性用法

      * orphanRemoval 表示当关联关系被删除的时候，是否应用级联删除，默认 false。

        ```java
        public class UserInfo {
           @OneToOne(cascade = {CascadeType.PERSIST},orphanRemoval = true)
           private User user;
           ....其他没变的代码省了
        }
        ```

  * 主键和外键都是同一个字段

    ```java
    我们假设 user 表是主表，user_info 的主键是 user_id，并且 user_id=user 是表里面的 id
    public class UserInfo implements Serializable {
       @Id
       private Long userId;
       private Integer ages;
       private String telephone;
       @MapsId
       @OneToOne(cascade = {CascadeType.PERSIST},orphanRemoval = true)
       private User user;
    }
    这里的做法很简单，我们直接把 userId 设置为主键，在 @OneToOne 上面添加 @MapsId 注解即可。@MapsId 注解的作用是把关联关系实体里面的 ID（默认）值 copy 到 @MapsId 标注的字段上面（这里指的是 user_id 字段）。
    Hibernate: create table user (id bigint not null, address varchar(255), email varchar(255), name varchar(255), sex varchar(255), primary key (id))
    Hibernate: create table user_info (ages integer, telephone varchar(255), user_id bigint not null, primary key (user_id))
    Hibernate: alter table user_info add constraint FKn8pl63y4abe7n0ls6topbqjh2 foreign key (user_id) references user
    ```

  * @OneToOne 延迟加载，我们只需要 ID 值

    * 在 @OneToOne 延迟加载的情况下，我们假设只想查下 user_id，而不想查看 user 表其他的信息，因为当前用不到，可以有以下几种做法。

      ```java
      第一种做法：还是 User 实体不变，我们改一下 UserInfo 对象，如下所示：
      package com.example.jpa.example1;
      import lombok.*;
      import javax.persistence.*;
      @Entity
      @Data
      @Builder
      @AllArgsConstructor
      @NoArgsConstructor
      @ToString(exclude = "user")
      public class UserInfo{
         @Id
         @GeneratedValue(strategy= GenerationType.AUTO)
         private Long id;
         private Integer ages;
         private String telephone;
         @MapsId
         @OneToOne(cascade = {CascadeType.PERSIST},orphanRemoval = true,fetch = FetchType.LAZY)
         private User user;
      }
      从上面这段代码中，可以看到做的更改如下：
      id 字段我们先用原来的
      @OneToOne 上面我们添加 @MapsId 注解
      @OneToOne 里面的 fetch = FetchType.LAZY 设置延迟加载
      
      @DataJpaTest
      @TestInstance(TestInstance.Lifecycle.PER_CLASS)
      public class UserInfoRepositoryTest {
          @Autowired
          private UserInfoRepository userInfoRepository;
          @BeforeAll
          @Rollback(false)
          @Transactional
          void init() {
              User user = User.builder().name("jackxx").email("123456@126.com").build();
              UserInfo userInfo = UserInfo.builder().ages(12).user(user).telephone("12345678").build();
              userInfoRepository.saveAndFlush(userInfo);
          }
          /**
           * 测试用User关联关系操作
           *
           * @throws JsonProcessingException
           */
          @Test
          @Rollback(false)
          public void testUserRelationships() throws JsonProcessingException {
              UserInfo userInfo1 = userInfoRepository.getOne(1L);
              System.out.println(userInfo1);
              System.out.println(userInfo1.getUser().getId());
          }
      }
      接下来介绍第二种做法，这种做法很简单，只要在 UserInfo 对象里面直接去掉 @OneToOne 关联关系，新增下面的字段即可。
      @Column(name = "user_id")
      private Long userId;
      
      第三做法是利用 Hibernate，它给我们提供了一种字节码增强技术，通过编译器改变 `class` 解决了延迟加载问题。这种方式有点复杂，需要在编译器引入 hibernateEnhance 的相关 jar 包，以及编译器需要改变 class 文件并添加 lazy 代理来解决延迟加载。我不太推荐这种方式，因为太复杂，你知道有这回事就行了。
      ```

  * **@JoinCloumns & JoinColumn**

    ```java
    public @interface JoinColumn {
        //关键的字段名,默认注解上的字段名，在@OneToOne代表本表的外键字段名字；
        String name() default "";
        //与name相反关联对象的字段，默认主键字段
        String referencedColumnName() default "";
        //外键字段是否唯一
        boolean unique() default false;
        //外键字段是否允许为空
        boolean nullable() default true;
        //是否跟随一起新增
        boolean insertable() default true;
        //是否跟随一起更新
        boolean updatable() default true;
        //JPA2.1新增，外键策略
        ForeignKey foreignKey() default @ForeignKey(PROVIDER_DEFAULT);
    }
    
    其次，我们看一下 @ForeignKey(PROVIDER_DEFAULT) 里面枚举值有几个。
    public enum ConstraintMode {
        //创建外键约束
       CONSTRAINT,
        //不创建外键约束
       NO_CONSTRAINT,
       //采用默认行为
       PROVIDER_DEFAULT
    }
    然后，我们看看这个注解的语法，就可以解答我们上面的两个问题。修改一下 UserInfo，如下所示：
    public class UserInfo{
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private Integer ages;
       private String telephone;
       @OneToOne(cascade = {CascadeType.PERSIST},orphanRemoval = true,fetch = FetchType.LAZY)
       @JoinColumn(foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT),name = "my_user_id")
       private User user;
       ...其他不变
    }
    可以看到，我们在其中指定了字段的名字：my_user_id，并且指定 NO_CONSTRAINT 不生成外键。而测试用例不变，我们看下运行结果。
    Hibernate: create table user (id bigint not null, address varchar(255), email varchar(255), name varchar(255), sex varchar(255), primary key (id))
    Hibernate: create table user_info (id bigint not null, ages integer, telephone varchar(255), my_user_id bigint, primary key (id))
        
    而 @JoinColumns 是 JoinColumns 的复数形式，就是通过两个字段进行的外键关联，这个不常用，我们看一个 demo 了解一下就好。
    @Entity
    public class CompanyOffice {
       @ManyToOne(fetch = FetchType.LAZY)
       @JoinColumns({
             @JoinColumn(name="ADDR_ID", referencedColumnName="ID"),
             @JoinColumn(name="ADDR_ZIP", referencedColumnName="ZIP")
       })
       private Address address;
    }
    ```

  * **@ManyToOne& @OneToMany**

  * @ManyToOne 代表多对一的关联关系，而 @OneToMany 代表一对多，一般两个成对使用表示双向关联关系。而 JPA 协议中也是明确规定：维护关联关系的是拥有外键的一方，而另一方必须配置 mappedBy.

    ```java
    public @interface ManyToOne {
        Class targetEntity() default void.class;
        CascadeType[] cascade() default {};
        FetchType fetch() default EAGER;
        boolean optional() default true;
    }
     public @interface OneToMany {
        Class targetEntity() default void.class;
     	//cascade 级联操作策略：(CascadeType.PERSIST、CascadeType.REMOVE、	CascadeType.REFRESH、CascadeType.MERGE、CascadeType.ALL)
    	// 如果不填，默认关系表不会产生任何影响。
        CascadeType[] cascade() default {};
    	// 数据获取方式EAGER(立即加载)/LAZY(延迟加载)
        FetchType fetch() default LAZY;
        // 关系被谁维护，单项的。注意：只有关系维护方才能操作两者的关系。
        String mappedBy() default "";
    	// 是否级联删除。和CascadeType.REMOVE的效果一样。两种配置了一个就会自动级联删除
        boolean orphanRemoval() default false;
    }
    我们看到上面的字段和 @OneToOne 里面的基本一样，用法是一样的，不过需要注意以下几点：
    @ManyToOne 一定是维护外键关系的一方，所以没有 mappedBy 字段；
    @ManyToOne 删除的时候一定不能把 One 的一方删除了，所以也没有 orphanRemoval 的选项；
    @ManyToOne 的 Lazy 效果和 @OneToOne 的一样，所以和上面的用法基本一致；
    @OneToMany 的 Lazy 是有效果的。
    ```

  * **@ManyToMany**

    ```java
    假设 user 表和 room 表是多对多的关系
    package com.example.jpa.example1;
    import lombok.*;
    import javax.persistence.*;
    import java.io.Serializable;
    import java.util.List;
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class User{
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       @ManyToMany(mappedBy = "users")
       private List<Room> rooms;
    }
    接着，我们让 Room 维护关联关系。
    package com.example.jpa.example1;
    import lombok.*;
    import javax.persistence.*;
    import java.util.List;
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "users")
    public class Room {
       @Id
       @GeneratedValue(strategy = GenerationType.AUTO)
       private Long id;
       private String title;
       @ManyToMany
       private List<User> users;
    }
    Hibernate: create table room (id bigint not null, title varchar(255), primary key (id))
    Hibernate: create table room_users (rooms_id bigint not null, users_id bigint not null)
    Hibernate: create table user (id bigint not null, email varchar(255), name varchar(255), sex varchar(255), primary key (id))
    Hibernate: alter table room_users add constraint FKld9phr4qt71ve3gnen43qxxb8 foreign key (users_id) references user
    Hibernate: alter table room_users add constraint FKtjvf84yquud59juxileusukvk foreign key (rooms_id) references room
    从结果上我们看到 JPA 帮我们创建的三张表中，room_users 表维护了 user 和 room 的多对多关联关系。其实这个情况还告诉我们一个道理：当用到 @ManyToMany 的时候一定是三张表，不要想着建两张表，两张表肯定是违背表的设计原则的。
    @ManyToMany 的语法。
    public @interface ManyToMany {
        Class targetEntity() default void.class;
        CascadeType[] cascade() default {};
        FetchType fetch() default LAZY;
        String mappedBy() default "";
    }
    ```

  * **@JoinTable。**

    ```java
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "users")
    public class Room {
       @Id
       @GeneratedValue(strategy = GenerationType.AUTO)
       private Long id;
       private String title;
       @ManyToMany
       @JoinTable(name = "user_room_ref",
             joinColumns = @JoinColumn(name = "room_id_x"),
             inverseJoinColumns = @JoinColumn(name = "user_id_x")
       )
       private List<User> users;
    }
    Hibernate: create table room (id bigint not null, title varchar(255), primary key (id))
    Hibernate: create table user (id bigint not null, email varchar(255), name varchar(255), sex varchar(255), primary key (id))
    Hibernate: create table user_room_ref (room_id_x bigint not null, user_id_x bigint not null)
    Hibernate: alter table user_room_ref add constraint FKoxolr1eyfiu69o45jdb6xdule foreign key (user_id_x) references user
    Hibernate: alter table user_room_ref add constraint FK2sl9rtuxo9w130d83e19f3dd9 foreign key (room_id_x) references room
    
    到这里可以看到，我们创建了一张中间表，并且添加了两个在预想之内的外键关系。
    public @interface JoinTable {
        //中间关联关系表名
        String name() default "";
        //表的catalog
        String catalog() default "";
        //表的schema
        String schema() default "";
        //维护关联关系一方的外键字段的名字
        JoinColumn[] joinColumns() default {};
        //另一方的表外键字段
        JoinColumn[] inverseJoinColumns() default {};
        //指定维护关联关系一方的外键创建规则
        ForeignKey foreignKey() default @ForeignKey(PROVIDER_DEFAULT);
        //指定另一方的外键创建规则
        ForeignKey inverseForeignKey() default @Forei gnKey(PROVIDER_DEFAULT);
    }
    ```

  * 利用 @ManyToOne 和 @OneToMany 表达多对多的关联关系

    ```java
    我们新建一张表 user_room_relation 来存储双方的关联关系和额外字段，实体如下：
    package com.example.jpa.example1;
    import lombok.*;
    import javax.persistence.*;
    import java.util.Date;
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class UserRoomRelation {
       @Id
       @GeneratedValue(strategy = GenerationType.AUTO)
       private Long id;
       private Date createTime,udpateTime;
       @ManyToOne
       private Room room;
       @ManyToOne
       private User user;
    }
    而 User 变化如下：
    public class User implements Serializable {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       @OneToMany(mappedBy = "user")
       private List<UserRoomRelation> userRoomRelations;
    ....}
    
    Room 变化如下：
    public class Room {
       @Id
       @GeneratedValue(strategy = GenerationType.AUTO)
       private Long id;
       @OneToMany(mappedBy = "room")
       private List<UserRoomRelation> userRoomRelations;
    ...}
    
    Hibernate: create table user_room_relation (id bigint not null, create_time timestamp, udpate_time timestamp, room_id bigint, user_id bigint, primary key (id))
    Hibernate: create table room (id bigint not null, title varchar(255), primary key (id))
    Hibernate: create table user (id bigint not null, email varchar(255), name varchar(255), sex varchar(255), primary key (id))
    Hibernate: alter table user_room_relation add constraint FKaesy2rg60vtaxxv73urprbuwb foreign key (room_id) references room
    Hibernate: alter table user_room_relation add constraint FK45gha85x63026r8q8hs03uhwm foreign key (user_id) references user
    ```

  * @ManyToMany 的最佳实践

    * 上面我们介绍的 @OneToMany 的最佳实践同样适用，我为了说明方便，采用的是双向关联，而实际生产一般是在中间表对象里面做单向关联，这样会让实体之间的关联关系简单很多。
    * 与 @OneToMany 一样的道理，不要用级联删除和 orphanRemoval=true。
    * FetchType 采用默认方式：fetch = FetchType.LAZY 的方式。

* 核心模块有三个

  * **jackson-core：核心包**，提供基于“流模式”解析的相关 API，它包括 JsonPaser 和 JsonGenerator。Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。

  * **jackson-annotations：注解包**，提供标准注解功能，这是我们必须要掌握的基础语法。

  * **jackson-databind：数据绑定包**，提供基于“对象绑定”解析的相关 API（ ObjectMapper ） 和“树模型”解析的相关 API（JsonNode）；基于“对象绑定”解析的 API 和“树模型”解析的 API 依赖基于“流模式”解析的 API。如下图中一些标准的类型转换：

    ![Drawing 1.png](https://s0.lgstatic.com/i/image/M00/59/F4/CgqCHl9y6LCAZOFqAAGiK2TqQR8365.png)

  * ![Lark20201009-105051.png](https://s0.lgstatic.com/i/image/M00/5B/A6/CgqCHl9_0CiAWB2rAAL0pfxIviE487.png)

    ```java
    package com.example.jpa.example1;
    import com.fasterxml.jackson.annotation.*;
    import lombok.*;
    import javax.persistence.*;
    import java.time.Instant;
    import java.util.*;
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @JsonPropertyOrder({"createDate","email"})
    public class UserJson {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       @JsonProperty("my_name")
       private String name;
       private Instant createDate;
       @JsonFormat(timezone ="GMT+8", pattern = "yyyy-MM-dd HH:mm")
       private Date updateDate;
       private String email;
       @JsonIgnore
       private String sex;
       @JsonCreator
       public UserJson(@JsonProperty("email") String email) {
          System.out.println("其他业务逻辑");
          this.email = email;
       }
       @Transient
       @JsonAnySetter
       private Map<String,Object> other = new HashMap<>();
       @JsonAnyGetter
       public Map<String, Object> getOther() {
          return other;
       }
    }
    ```

  * @JsonAutoDetect JsonAutoDetect.Visibility 类包含与 Java 中的可见性级别匹配的常量，表示 ANY、DEFAULT、NON_PRIVATE、NONE、PROTECTED_AND_PRIVATE和PUBLIC_ONLY。

  * Jackson 默认不是所有的属性都可以被序列化和反序列化。默认的属性可视化的规则如下：

    * 若该属性修饰符是 public，该属性可序列化和反序列化。
    * 若属性的修饰符不是 public，但是它的 getter 方法和 setter 方法是 public，该属性可序列化和反序列化。因为 getter 方法用于序列化，而 setter 方法用于反序列化。
    * 若属性只有 public 的 setter 方法，而无 public 的 getter 方法，该属性只能用于反序列化。

    ```java
    ObjectMapper mapper = new ObjectMapper();
    // PropertyAccessor 支持的类型有 ALL,CREATOR,FIELD,GETTER,IS_GETTER,NONE,SETTER
    // Visibility 支持的类型有ANY,DEFAULT,NON_PRIVATE,NONE,PROTECTED_AND_PUBLIC,PUBLIC_ONLY
    mapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
    ```

  * 反序列化最重要的方法

    ```java
    public <T> T readValue(String content, Class<T> valueType)
    public <T> T readValue(String content, TypeReference<T> valueTypeRef)
    public <T> T readValue(String content, JavaType valueType)
    ```

    ```java
    String json = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(userJson);
    //单个对象的写法：
    UserJson user = objectMapper.readValue(json,UserJson.class);
    //返回List的返回结果的写法：
    List<User> personList2 = mapper.readValue(jsonListString, new TypeReference<List<User>>(){});
    ```

  * Jackson 与 JPA 常见的问题

    * 死循环问题如何解决

      ```java
      第一种情况：我们在写 ToString 方法，特别是 JPA 的实体的时候，很容易陷入死循环，因为实体之间的关联关系配置是双向的，我们就需要 ToString 的时候把一方排除掉，如下所示：
      @ToString(exclude="address")
      
      
      第二种情况：在转化JSON的时候，双向关联也会死循环。按照我们上面讲的方法，这是时候我们要想到通过 @JsonIgnoreProperties(value={"address"})或者字段上面配置@JsonIgnore，如下：
      @JsonIgnore
      private List<UserAddress> address;
      
      此外，通过 @JsonBackReference 和 @JsonManagedReference 注解也可以解决死循环。
      public class UserAddress {
         @JsonManagedReference
         private User user;
      ....}
      public class User implements Serializable {
         @OneToMany(mappedBy = "user",fetch = FetchType.LAZY)
         @JsonBackReference
         private List<UserAddress> address;
      ...}
      JPA 实体 JSON 序列化的常见报错
      No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: com.example.jpa.example1.User$HibernateProxy$MdjeSaTz["hibernateLazyInitializer"])
      com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: com.example.jpa.example1.User$HibernateProxy$MdjeSaTz["hibernateLazyInitializer"])
      ```

    * 常见报错解决方法

      * **解决方法一：引入 Hibernate5Module**

        ```java
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new Hibernate5Module());
        String json = objectMapper.writeValueAsString(user);
        System.out.println(json);
        这样子就不会报错了。
        
        Hibernate5Module 里面还有很多 Feature 配置，例如FORCE_LAZY_LOADING，强制 lazy 里面加载就不会有上面的问题了。但是这个会有性能问题，我不建议使用。
        
        还有 USE_TRANSIENT_ANNOTATION，利用 JPA 的 @Transient 注解配置，这个默认是开启的。所以基本上 feature 默认配置都是 ok 的，不需要我们动手，只要知道这回事就行了。
        ```

      * **解决方法二：关闭 SerializationFeature.FAIL_ON_EMPTY_BEANS 的 feature**

        ```java
        ObjectMapper objectMapper = new ObjectMapper();
        //直接关闭SerializationFeature.FAIL_ON_EMPTY_BEANS       		objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS,false);
        String json = objectMapper.writeValueAsString(user);
        System.out.println(json)
        因为是 lazy，所以 empty 的 bean 的时候不报错也可以。
        ```

      * **解决方法三：对象上面排除“hibernateLazyInitializer”“handler”“fieldHandler”等**

        ```java
        @JsonIgnoreProperties(value={"address","hibernateLazyInitializer","handler","fieldHandler"})
        public class User implements Serializable {}
        ```

      * **ObjectMapper 实战经验推荐配置项**

        ```java
        ObjectMapper objectMapper = new ObjectMapper();
        //empty beans不需要报错，没有就是没有了
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS,false);
        //遇到不可识别字段的时候不要报错，因为前端传进来的字段不可信，可以不要影响正常业务逻辑
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,false);
        //遇到不可以识别的枚举的时候，为了保证服务的强壮性，建议也不要关心未知的，甚至给个默认的，特别是微服务大家的枚举值随时在变，但是老的服务是不需要跟着一起变的
        objectMapper.configure(DeserializationFeature.READ_UNKNOWN_ENUM_VALUES_AS_NULL,true);
        objectMapper.configure(DeserializationFeature.READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE,true);
        ```

      * **时间类型的最佳实践，如何返回 ISO 格式的标准时间**

        ```java
        有的时候我们会发现，默认的 ObjectMapper 里面的 module 提供的时间转化格式可能不能满足我们的要求，可能要进行扩展，如下：
        @Test
        @Rollback(false)
        public void testUserJson() throws JsonProcessingException {
            UserJson userJson = userJsonRepository.findById(1L).get();
            userJson.setOther(Maps.newHashMap("address","shanghai"));
            //自定义 myInstant解析序列化和反序列化DateTimeFormatter.ISO_ZONED_DATE_TIME这种格式
           SimpleModule myInstant = new SimpleModule("instant", Version.unknownVersion())
                    .addSerializer(java.time.Instant.class, new JsonSerializer<Instant>() {
                        @Override
                        public void serialize(java.time.Instant instant,
                                              JsonGenerator jsonGenerator,
                                              SerializerProvider serializerProvider)
                                throws IOException {
                            if (instant == null) {
                                jsonGenerator.writeNull();
                            } else {
                                jsonGenerator.writeObject(instant.toString());
                            }
                        }
                    })
                    .addDeserializer(Instant.class, new JsonDeserializer<Instant>() {
                        @Override
                        public Instant deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
                            Instant result = null;
                            String text = jsonParser.getText();
                            if (!StringUtils.isEmpty(text)) {
                                result = ZonedDateTime.parse(text, DateTimeFormatter.ISO_ZONED_DATE_TIME).toInstant();
                            }
                            return result;
                        }
                    });
            ObjectMapper objectMapper = new ObjectMapper();
            //注册自定义的module
            objectMapper.registerModule(myInstant);
            String json = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(userJsn);
            System.out.println(json);
        }
        ```

### 2. 进阶用法

* **QueryByExampleExecutor**

  * QueryByExampleExecutor（QBE）是一种用户友好的查询技术，具有简单的接口，它允许动态查询创建，并且不需要编写包含字段名称的查询。

    ![img](https://s0.lgstatic.com/i/image/M00/5D/4B/CgqCHl-EE7WAAfi5AACTjc0iffY586.png)

  * QBE 的基本语法

    ```java
    public interface QueryByExampleExecutor<T> { 
         //根据“实体”查询条件，查找一个对象
        <S extends T> S findOne(Example<S> example);
        //根据“实体”查询条件，查找一批对象
        <S extends T> Iterable<S> findAll(Example<S> example); 
        //根据“实体”查询条件，查找一批对象，可以指定排序参数
        <S extends T> Iterable<S> findAll(Example<S> example, Sort sort);
         //根据“实体”查询条件，查找一批对象，可以指定排序和分页参数 
        <S extends T> Page<S> findAll(Example<S> example, Pageable pageable);
        //根据“实体”查询条件，查找返回符合条件的对象个数
        <S extends T> long count(Example<S> example); 
        //根据“实体”查询条件，判断是否有符合条件的对象
        <S extends T> boolean exists(Example<S> example); 
    }
    ```

    ```java
    @DataJpaTest
    @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    public class UserAddressRepositoryTest {
       @Autowired
       private UserAddressRepository userAddressRepository;
       private Date now = new Date();
       /**
        * 负责添加数据，假设数据库里面已经有的数据
        */
       @BeforeAll
       @Rollback(false)
       @Transactional
       void init() {
          User user = User.builder()
                .name("jack")
                .email("123456@126.com")
                .sex(SexEnum.BOY)
                .age(20)
                .createDate(Instant.now())
                .updateDate(now)
                .build();
          userAddressRepository.saveAll(Lists.newArrayList(UserAddress.builder().user(user).address("shanghai").build(),
                UserAddress.builder().user(user).address("beijing").build()));
       }
       @Test
       @Rollback(false)
       public void testQBEFromUserAddress() throws JsonProcessingException {
          User request = User.builder()
                .name("jack").age(20).email("12345")
                .build();
          UserAddress address = UserAddress.builder().address("shang").user(request).build();
          ObjectMapper objectMapper = new ObjectMapper();
    //    System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(address)); //可以打印出来看看参数是什么
    //创建匹配器，即如何使用查询条件
          ExampleMatcher exampleMatcher = ExampleMatcher.matching()
                .withMatcher("user.email", ExampleMatcher.GenericPropertyMatchers.startsWith())
                .withMatcher("address", ExampleMatcher.GenericPropertyMatchers.startsWith());
          Page<UserAddress> u = userAddressRepository.findAll(Example.of(address,exampleMatcher), PageRequest.of(0,2));
        System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(u));
       }
    }
    ```

    ```java
    Example 语法详解
    public interface Example<T> {
       static <T> Example<T> of(T probe) {
          return new TypedExample<>(probe, ExampleMatcher.matching());
       }
       static <T> Example<T> of(T probe, ExampleMatcher matcher) {
          return new TypedExample<>(probe, matcher);
       }
       //实体参数
       T getProbe();
       //匹配器
       ExampleMatcher getMatcher();
       //回顾一下我们上一课时讲解的类型，这个是返回实体参数的Class Type；
       @SuppressWarnings("unchecked")
       default Class<T> getProbeType() {
          return (Class<T>) ProxyUtils.getUserClass(getProbe().getClass());
       }
    }
    
    @ToString
    @EqualsAndHashCode
    @RequiredArgsConstructor(access = AccessLevel.PACKAGE)
    @Getter
    class TypedExample<T> implements Example<T> {
       private final @NonNull T probe;
       private final @NonNull ExampleMatcher matcher;
    }
    ```

    > 其中我们发现三个类：Probe、ExampleMatcher 和 Example，分别做如下解释：
    >
    > - Probe：这是具有填充字段的域对象的实际实体类，即查询条件的封装类（又可以理解为查询条件参数），必填。
    > - ExampleMatcher：ExampleMatcher 有关如何匹配特定字段的匹配规则，它可以重复使用在多个实例中，必填。
    > - Example：Example 由 Probe 探针和 ExampleMatcher 组成，它用于创建查询，即组合查询参数和参数的匹配规则。
    >
    > 通过 Example 的源码，我们发现想创建 Example 的话，只有两个方法：
    >
    > 1. static `<T>` Example`<T>` of(T probe)：需要一个实体参数，即查询的条件。而里面的 ExampleMatcher 采用默认的 ExampleMatcher.matching()； 表示忽略 Null，所有字段采用精准匹配。
    > 2. static `<T>` Example`<T>` of(T probe, ExampleMatcher matcher)：需要两个参数构建 Example，也就表示了 ExampleMatcher 自由组合规则，正如我们上面的测试用例里面的代码一样。

  * **ExampleMatcher** 

    ```java
    //默认matching方法
    static ExampleMatcher matching() {
       return matchingAll();
    }
    //matchingAll，默认的方法
    static ExampleMatcher matchingAll() {
       return new TypedExampleMatcher().withMode(MatchMode.ALL);
    }
    两个方法所表达的意思是一样的，只不过一个是默认，一个是方法名上面有语义的。两者采用的都是 MatchMode.ALL 的模式，即 AND 模式
    Hibernate: select useraddres0_.id as id1_2_, useraddres0_.address as address2_2_, useraddres0_.user_id as user_id3_2_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id=user1_.id where user1_.age=20 and user1_.name=? and (user1_.email like ? escape ?) and (useraddres0_.address like ? escape ?) limit ?
    ，这些查询条件之间都是 AND 的关系
    
    static ExampleMatcher matchingAny() {
       return new TypedExampleMatcher().withMode(MatchMode.ANY);
    }
    第三个方法和前面两个方法的区别在于：第三个 MatchMode.ANY，表示查询条件是 or 的关系，我们看一下 SQL
    Hibernate: select count(useraddres0_.id) as col_0_0_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id=user1_.id where useraddres0_.address like ? escape ? or user1_.age=20 or user1_.email like ? escape ? or user1_.name=?
    ```

  * ExampleMatcher 语法暴露的方法

    * **忽略大小写**

      ```java
      //默认忽略大小写的方式，默认 False。
      ExampleMatcher withIgnoreCase(boolean defaultIgnoreCase);
      //提供了一个默认的实现方法，忽略大小写；
      default ExampleMatcher withIgnoreCase() {
         return withIgnoreCase(true);
      }
      //哪些属性的paths忽略大小写，可以指定多个参数；
      ExampleMatcher withIgnoreCase(String... propertyPaths);
      ```

    * **暴露的 Null 值处理方式如下：**

      ```java
      ExampleMatcher withNullHandler(NullHandler nullHandler);
      参数 NullHandler枚举值即可，有两个可选值：INCLUDE（包括）、IGNORE（忽略），其中要注意：
      标识作为条件的实体对象中，一个属性值（条件值）为 Null 时，是否参与过滤；
      当该选项值是 INCLUDE 时，表示仍参与过滤，会匹配数据库表中该字段值是 Null 的记录；
      若为 IGNORE 值，表示不参与过滤。
      
      //提供一个默认实现方法，忽略 NULL 属性；
      default ExampleMatcher withIgnoreNullValues() {
         return withNullHandler(NullHandler.IGNORE);
      }
      //把 NULL 属性值作为查询条件
      default ExampleMatcher withIncludeNullValues() {
         return withNullHandler(NullHandler.INCLUDE);
      }
      ```

    * **忽略某些 Paths，不参加查询条件**

      ```java
      //忽略某些属性列表，不参与查询过滤条件。
      ExampleMatcher withIgnorePaths(String... ignoredPaths);
      ```

    * ##### 字符串字段默认的匹配规则

      ```java
      ExampleMatcher withStringMatcher(StringMatcher defaultStringMatcher);
      关于默认字符串的匹配方式，枚举类型有 6 个可选值，
          DEFAULT（默认，效果同 EXACT）、
          EXACT（相等）、STARTING（开始匹配）、
          ENDING（结束匹配）、
          CONTAINING（包含，模糊匹配）、
          REGEX（正则表达式）。
      ExampleMatcher withMatcher(String propertyPath, GenericPropertyMatcher genericPropertyMatcher);
      
      ```

      ![Drawing 4.png](https://s0.lgstatic.com/i/image/M00/5D/41/Ciqc1F-EFHuAXsn3AABiCE6_I0I978.png)

    * 示例

      ```java
    //创建匹配器，即如何使用查询条件
      ExampleMatcher exampleMatcher = ExampleMatcher
            //采用默认and的查询方式
            .matchingAll()
            //忽略大小写
            .withIgnoreCase()
            //忽略所有null值的字段
            .withIgnoreNullValues()
            .withIgnorePaths("id","createDate")
            //默认采用精准匹配规则
            .withStringMatcher(ExampleMatcher.StringMatcher.EXACT)
            //级联查询，字段user.email采用字符前缀匹配规则
            .withMatcher("user.email", ExampleMatcher.GenericPropertyMatchers.startsWith())
            //特殊指定address字段采用后缀匹配
            .withMatcher("address", ExampleMatcher.GenericPropertyMatchers.endsWith());
      Page<UserAddress> u = userAddressRepository.findAll(Example.of(address,exampleMatcher), PageRequest.of(0,2));
      ```
    
    * ExampleExceutor 使用中需要考虑的因素
    
      * Null 值的处理：当某个条件值为 Null 时，是应当忽略这个过滤条件，还是应当去匹配数据库表中该字段值是 Null 的记录呢？
      * 忽略某些属性值：一个实体对象，有许多个属性，是否每个属性都参与过滤？是否可以忽略某些属性？
      * 不同的过滤方式：同样是作为 String 值，可能“姓名”希望精确匹配，“地址”希望模糊匹配，如何做到？
  
* JpaSpecificationExecutor 使用案例

  ```java
      @Test
      public void testSPE() {
          //模拟请求参数
          User userQuery = User.builder()
                  .name("jack")
                  .email("123456@126.com")
                  .sex(SexEnum.BOY)
                  .age(20)
                  .addresses(Lists.newArrayList(UserAddress.builder().address("shanghai").build()))
                  .build();
                  //假设的时间范围参数
          Instant beginCreateDate = Instant.now().plus(-2, ChronoUnit.HOURS);
          Instant endCreateDate = Instant.now().plus(1, ChronoUnit.HOURS);
          //利用Specification进行查询
          Page<User> users = userRepository.findAll(new Specification<User>() {
              @Override
              public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                  List<Predicate> ps = new ArrayList<Predicate>();
                  if (StringUtils.isNotBlank(userQuery.getName())) {
                      //我们模仿一下like查询，根据name模糊查询
                      ps.add(cb.like(root.get("name"),"%" +userQuery.getName()+"%"));
                  }
                  if (userQuery.getSex()!=null){
                      //equal查询条件，这里需要注意，直接传递的是枚举
                      ps.add(cb.equal(root.get("sex"),userQuery.getSex()));
                  }
                  if (userQuery.getAge()!=null){
                      //greaterThan大于等于查询条件
                      ps.add(cb.greaterThan(root.get("age"),userQuery.getAge()));
                  }
                  if (beginCreateDate!=null&&endCreateDate!=null){
                      //根据时间区间去查询创建
                      ps.add(cb.between(root.get("createDate"),beginCreateDate,endCreateDate));
                  }
                  if (!ObjectUtils.isEmpty(userQuery.getAddresses())) {
                      //联表查询，利用root的join方法，根据关联关系表里面的字段进行查询。
                      ps.add(cb.in(root.join("addresses").get("address")).value(userQuery.getAddresses().stream().map(a->a.getAddress()).collect(Collectors.toList())));
                  }
                  return query.where(ps.toArray(new Predicate[ps.size()])).getRestriction();
              }
          }, PageRequest.of(0, 2));
          System.out.println(users);
      }
  ```

  ```java
  public interface JpaSpecificationExecutor<T> {
     //根据 Specification 条件查询单个对象，要注意的是，如果条件能查出来多个会报错
     T findOne(@Nullable Specification<T> spec);
     //根据 Specification 条件，查询 List 结果
     List<T> findAll(@Nullable Specification<T> spec);
     //根据 Specification 条件，分页查询
     Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);
     //根据 Specification 条件，带排序的查询结果
     List<T> findAll(@Nullable Specification<T> spec, Sort sort);
     //根据 Specification 条件，查询数量
     long count(@Nullable Specification<T> spec);
  }
  ```

  * Root`<User>` root

    * 代表了可以查询和操作的实体对象的根，如果将实体对象比喻成表名，那 root 里面就是这张表里面的字段，而这些字段只是 JPQL 的实体字段而已。我们可以通过里面的 Path get（String attributeName），来获得我们想要操作的字段。
    
  * CriteriaQuery<?> query
  
    * 代表一个 specific 的顶层查询对象，它包含着查询的各个部分，比如 select 、from、where、group by、order by 等。CriteriaQuery 对象只对实体类型或嵌入式类型的 Criteria 查询起作用。简单理解为，它提供了查询 ROOT 的方法。
  
  * CriteriaBuilder cb
  
    * CriteriaBuilder 是用来构建 CritiaQuery 的构建器对象，其实就相当于条件或者条件组合，并以 Predicate 的形式返回。
  
    ```java
    package com.example.jpa.example1.spe;
    import org.springframework.data.jpa.domain.Specification;
    import javax.persistence.criteria.*;
    public class MySpecification<Entity> implements Specification<Entity> {
       private SearchCriteria criteria;
       public MySpecification (SearchCriteria criteria) {
          this.criteria = criteria;
       }
       /**
        * 实现实体根据不同的字段、不同的Operator组合成不同的Predicate条件
        *
        * @param root            must not be {@literal null}.
        * @param query           must not be {@literal null}.
        * @param builder  must not be {@literal null}.
        * @return a {@link Predicate}, may be {@literal null}.
        */
       @Override
       public Predicate toPredicate(Root<Entity> root, CriteriaQuery<?> query, CriteriaBuilder builder) {
          if (criteria.getOperation().compareTo(Operator.GT)==0) {
             return builder.greaterThanOrEqualTo(
                   root.<String> get(criteria.getKey()), criteria.getValue().toString());
          }
          else if (criteria.getOperation().compareTo(Operator.LT)==0) {
             return builder.lessThanOrEqualTo(
                   root.<String> get(criteria.getKey()), criteria.getValue().toString());
          }
          else if (criteria.getOperation().compareTo(Operator.LK)==0) {
             if (root.get(criteria.getKey()).getJavaType() == String.class) {
                return builder.like(
                      root.<String>get(criteria.getKey()), "%" + criteria.getValue() + "%");
             } else {
                return builder.equal(root.get(criteria.getKey()), criteria.getValue());
             }
          }
          return null;
       }
    }
    
    package com.example.jpa.example1.spe;
    import lombok.*;
    /**
     * @author jack，实现不同的查询条件，不同的操作，针对Value;
     */
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class SearchCriteria {
       private String key;
       private Operator operation;
       private Object value;
    }
    
    package com.example.jpa.example1.spe;
    public enum Operator {
       /**
        * 等于
        */
       EQ("="),
       /**
        * 等于
        */
       LK(":"),
       /**
        * 不等于
        */
       NE("!="),
       /**
        * 大于
        */
       GT(">"),
       /**
        * 小于
        */
       LT("<"),
       /**
        * 大于等于
        */
       GE(">=");
       Operator(String operator) {
          this.operator = operator;
       }
       private String operator;
    }
    
    /**
     * 测试自定义的Specification语法
     */
    @Test
    public void givenLast_whenGettingListOfUsers_thenCorrect() {
        MySpecification<User> name =
            new MySpecification<User>(new SearchCriteria("name", Operator.LK, "jack"));
    MySpecification<User> age =
            new MySpecification<User>(new SearchCriteria("age", Operator.GT, 2));
    List<User> results = userRepository.findAll(Specification.where(name).and(age));
        System.out.println(results.get(0).getName());
    }
    
    先创建一个 Controller，用来接收 search 这样的查询条件：类似 userssearch=lastName:doe,age>25 的参数。
    @RestController
    public class UserController {
        @Autowired
        private UserRepository repo;
        @RequestMapping(method = RequestMethod.GET, value = "/users")
        @ResponseBody
        public List<User> search(@RequestParam(value = "search") String search) {
            Specification<User> spec = new SpecificationsBuilder<User>().buildSpecification(search);
            return repo.findAll(spec);
        }
    }
    
    Controller 里面非常简单，利用 SpecificationsBuilder 生成我们需要的 Specification 即可。
    package com.example.jpa.example1.spe;
    import com.example.jpa.example1.User;
    import org.springframework.data.jpa.domain.Specification;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;
    import java.util.stream.Collectors;
    /**
     * 处理请求参数
     * @param <Entity>
     */
    public class SpecificationsBuilder<Entity> {
       private final List<SearchCriteria> params;
    
       //初始化params，保证每次实例都是一个新的ArrayList
       public SpecificationsBuilder() {
          params = new ArrayList<SearchCriteria>();
       }
    
       //利用正则表达式取我们search参数里面的值，解析成SearchCriteria对象
       public Specification<Entity> buildSpecification(String search) {
          Pattern pattern = Pattern.compile("(\\w+?)(:|<|>)(\\w+?),");
          Matcher matcher = pattern.matcher(search + ",");
          while (matcher.find()) {
             this.with(matcher.group(1), Operator.fromOperator(matcher.group(2)), matcher.group(3));
          }
          return this.build();
       }
       //根据参数返回我们刚才创建的SearchCriteria
       private SpecificationsBuilder with(String key, Operator operation, Object value) {
          params.add(new SearchCriteria(key, operation, value));
          return this;
       }
       //根据我们刚才创建的MySpecification返回所需要的Specification
       private Specification<Entity> build() {
          if (params.size() == 0) {
             return null;
          }
          List<Specification> specs = params.stream()
                .map(MySpecification<User>::new)
                .collect(Collectors.toList());
          Specification result = specs.get(0);
          for (int i = 1; i < params.size(); i++) {
             result = Specification.where(result)
                   .and(specs.get(i));
          }
          return result;
       }
    }
    ```
  
* EntityManager 

  * **获得 EntityManager 的方式：通过 @PersistenceContext 注解。**

    * 将 @PersistenceContext 注解标注在 EntityManager 类型的字段上，这样得到的 EntityManager 就是容器管理的 EntityManager。由于是容器管理的，所以我们不需要、也不应该显式关闭注入的 EntityManager 实例。

      ```java
      @DataJpaTest
      @TestInstance(TestInstance.Lifecycle.PER_CLASS)
      public class UserRepositoryTest {
          //利用该方式获得entityManager
          @PersistenceContext
          private EntityManager entityManager;
          @Autowired
          private UserRepository userRepository;
          /**
           * 测试entityManager用法
           *
           * @throws JsonProcessingException
           */
          @Test
          @Rollback(false)
          public void testEntityManager() throws JsonProcessingException {
              //测试找到一个User对象
              User user = entityManager.find(User.class,2L);
              Assertions.assertEquals(user.getAddresses(),"shanghai");
      
              //我们改变一下user的删除状态
              user.setDeleted(true);
              //merger方法
              entityManager.merge(user);
              //更新到数据库里面
              entityManager.flush();
      
              //再通过createQuery创建一个JPQL，进行查询
              List<User> users =  entityManager.createQuery("select u From User u where u.name=?1")
                      .setParameter(1,"jack")
                      .getResultList();
              Assertions.assertTrue(users.get(0).getDeleted());
          }
      }
      ```

* @EnableJpaRepositories

  ```java
  public @interface EnableJpaRepositories {
     // value 等于 basePackage 用于配置扫描 Repositories 所在的 package 及子 package。
     // @EnableJpaRepositories(basePackages = "com.example")
     // 默认 @SpringBootApplication 注解出现目录及其子目录。
     String[] value() default {};
     
     // basePackageClasses 指定 Repository 类所在包，可以替换 basePackage 的使用。
     // 一样可以单个字符，下面例子表示 BookRepository.class 所在 Package 下面的所有 Repositories 都会被扫描注册。
     // @EnableJpaRepositories(basePackageClasses = BookRepository.class)
     String[] basePackages() default {};
     Class<?>[] basePackageClasses() default {};
     // includeFilters
     // 指定包含的过滤器，该过滤器采用 ComponentScan 的过滤器，可以指定过滤器类型。
     // @EnableJpaRepositories( includeFilters={@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Repository.class)})
     Filter[] includeFilters() default {};
     // 指定不包含过滤器，该过滤器也是采用 ComponentScan 的过滤器里面的类。
     // @EnableJpaRepositories(excludeFilters={@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Service.class),@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Controller.class)})
     Filter[] excludeFilters() default {};
     // repositoryImplementationPostfix 当我们自定义 Repository 的时候，约定的接口 Repository 的实现类的后缀是什么
     String repositoryImplementationPostfix() default "Impl";
     // namedQueriesLocation named SQL 存放的位置，默认为 META-INF/jpa-named-queries.properties
     // Todo.findBySearchTermNamedFile=SELECT t FROM Table t WHERE LOWER(t.description) LIKE LOWER(CONCAT('%', :searchTerm, '%')) ORDER BY t.title ASC
     String namedQueriesLocation() default "";
     // queryLookupStrategy 构建条件查询的查找策略，包含三种方式：CREATE、USE_DECLARED_QUERY、CREATE_IF_NOT_FOUND。
     // CREATE：按照接口名称自动构建查询方法，即我们前面说的 Defining Query Methods；
     // USE_DECLARED_QUERY：用 @Query 这种方式查询；
     // CREATE_IF_NOT_FOUND：如果有 @Query 注解，先以这个为准；如果不起作用，再用 Defining Query Methods；这个是默认的，基本不需要修改.
     Key queryLookupStrategy() default Key.CREATE_IF_NOT_FOUND;
     // 指定生产 Repository 的工厂类，默认 JpaRepositoryFactoryBean。JpaRepositoryFactoryBean 的主要作用是以动态代理的方式，帮我们所有 Repository 的接口生成实现类。例如当我们通过断点，看到 UserRepository 的实现类是 SimpleJpaRepository 代理对象的时候，就是这个工厂类干的，一般我们很少会去改变这个生成代理的机制。
     Class<?> repositoryFactoryBeanClass() default JpaRepositoryFactoryBean.class;
     // 用来指定我们自定义的 Repository 的实现类是什么。默认是 DefaultRepositoryBaseClass，即表示没有指定的 Repository 的实现基类。
     Class<?> repositoryBaseClass() default DefaultRepositoryBaseClass.class;
     // 用来指定创建和生产 EntityManager 的工厂类是哪个，默认是 name=“entityManagerFactory” 的 Bean。一般用于多数据配置。
     String entityManagerFactoryRef() default "entityManagerFactory";
     // 用来指定默认的事务处理是哪个类，默认是 transactionManager，一般用于多数据源。
     String transactionManagerRef() default "transactionManager";
     boolean considerNestedRepositories() default false;
     boolean enableDefaultTransactions() default true;
  }
  ```

* 通过 @EnableJpaRepositories 定义默认的 Repository 的实现类

  ```java
  第一步：正如上面我们讲的利用 @EnableJpaRepositories 指定 repositoryBaseClass，代码如下：
  @SpringBootApplication
  @EnableWebMvc
  @EnableJpaRepositories(repositoryImplementationPostfix = "Impl",repositoryBaseClass = CustomerBaseRepository.class)
  public class JpaApplication {
     public static void main(String[] args) {
        SpringApplication.run(JpaApplication.class, args);
     }
  }
  第二步：创建 CustomerBaseRepository 继承 SimpleJpaRepository 即可。
  package com.example.jpa.example1.customized;
  import org.springframework.data.jpa.repository.support.JpaEntityInformation;
  import org.springframework.data.jpa.repository.support.SimpleJpaRepository;
  import org.springframework.transaction.annotation.Transactional;
  import javax.persistence.EntityManager;
  @Transactional(readOnly = true)
  public class CustomerBaseRepository<T extends BaseEntity,ID> extends SimpleJpaRepository<T,ID>  {
      private final JpaEntityInformation<T, ?> entityInformation;
      private final EntityManager em;
      public CustomerBaseRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
          super(entityInformation, entityManager);
          this.entityInformation = entityInformation;
          this.em = entityManager;
      }
      public CustomerBaseRepository(Class<T> domainClass, EntityManager em) {
          super(domainClass, em);
          entityInformation = null;
          this.em = em;
      }
      //覆盖删除方法，实现逻辑删除，换成更新方法
      @Transactional
      @Override
      public void delete(T entity) {
          entity.setDeleted(Boolean.TRUE);
          em.merge(entity);
      }
  }
  
  第三步：写一个测试用例测试一下。
@Test
  public void testCustomizedBaseRepository() {
      User user = userRepository.findById(2L).get();
      userRepository.logicallyDelete(user);
      userRepository.delete(user);
      List<User> users = userRepository.findAll();
      Assertions.assertEquals(users.get(0).getDeleted(),Boolean.TRUE);
  }
  ```
  
* 实际应用场景

  * 首先肯定是我们做框架的时候、解决一些通用问题的时候，如逻辑删除，正如我们上面的实例所示的样子。
  *  在实际生产中经常会有这样的场景：对外暴露的是 UUID 查询方法，而对内暴露的是 Long 类型的 ID，这时候我们就可以自定义一个 FindByIdOrUUID 的底层实现方法，可以选择在自定义的 Respository 接口里面实现。
  * Defining Query Methods 和 @Query 满足不了我们的查询，但是我们又想用它的方法语义的时候，就可以考虑实现不同的 Respository 的实现类，来满足我们不同业务场景的复杂查询。

  ```java
  @SQLDelete(sql = "UPDATE user SET deleted = true where deleted =false and id = ?")
  public class User implements Serializable {
  ....
  }
  ```

* **JAP审计功能**

  * Auditing 是帮我们做审计用的，当我们操作一条记录的时候，需要知道这是谁创建的、什么时间创建的、最后修改人是谁、最后修改时间是什么时候，甚至需要修改记录……这些都是 Spring Data JPA 里面的 Auditing 支持的，它为我们提供了四个注解来完成上面说的一系列事情，如下：

    *  @CreatedBy 是哪个用户创建的。

    *  @CreatedDate 创建的时间。

    *  @LastModifiedBy 最后修改实体的用户。

    * @LastModifiedDate 最后一次修改的时间。

  * 具体实现步骤

    ```java
    第一种方式：直接在实例里面添加上述四个注解
    第一步：在 @Entity：User 里面添加四个注解，并且新增 @EntityListeners(AuditingEntityListener.class) 注解。
@Entity
    @Data
@Builder
    @AllArgsConstructor
@NoArgsConstructor
    @ToString(exclude = "addresses")
@EntityListeners(AuditingEntityListener.class)
    public class User implements Serializable {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       private String email;
       @Enumerated(EnumType.STRING)
       private SexEnum sex;
       private Integer age;
       @OneToMany(mappedBy = "user")
       @JsonIgnore
       private List<UserAddress> addresses;
       private Boolean deleted;
       @CreatedBy
       private Integer createUserId;
       @CreatedDate
       private Date createTime;
       @LastModifiedBy
       private Integer lastModifiedUserId;
       @LastModifiedDate
       private Date lastModifiedTime;
    }
    
    第二步：实现 AuditorAware 接口，告诉 JPA 当前的用户是谁。
    我们需要实现 AuditorAware 接口，以及 getCurrentAuditor 方法，并返回一个 Integer 的 user ID。
    public class MyAuditorAware implements AuditorAware<Integer> {
       //需要实现AuditorAware接口，返回当前的用户ID
       @Override
       public Optional<Integer> getCurrentAuditor() {
          ServletRequestAttributes servletRequestAttributes =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
          Integer userId = (Integer) servletRequestAttributes.getRequest().getSession().getAttribute("userId");
          return Optional.ofNullable(userId);
       }
    }
    这里关键的一步，是实现 AuditorAware 接口的方法，如下所示：
    public interface AuditorAware<T> {
       T getCurrentAuditor();
    }
    需要注意的是：这里获得用户 ID 的方法不止这一种，实际工作中，我们可能将当前的 user 信息放在 Session 中，可能把当前信息放在 Redis 中，也可能放在 Spring 的 security 里面管理。此外，这里的实现会有略微差异，我们以 security 为例：
    Authentication authentication =  SecurityContextHolder.getContext().getAuthentication();
    if (authentication == null || !authentication.isAuthenticated()) {
      return null;
    }
    Integer userId = ((LoginUserInfo) authentication.getPrincipal()).getUser().getId();
    
    第三步：通过 @EnableJpaAuditing 注解开启 JPA 的 Auditing 功能。
    第三步是最重要的一步，如果想使上面的配置生效，我们需要开启 JPA 的 Auditing 功能（默认没开启）。这里需要用到的注解是 @EnableJpaAuditing，代码如下：
    @Inherited
    @Documented
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Import(JpaAuditingRegistrar.class)
    public @interface EnableJpaAuditing {
    //auditor用户的获取方法，默认是找AuditorAware的实现类；
    String auditorAwareRef() default "";
    //是否在创建修改的时候设置时间，默认是true
    boolean setDates() default true;
    //在创建的时候是否同时作为修改，默认是true
    boolean modifyOnCreate() default true;
    //时间的生成方法，默认是取当前时间(为什么提供这个功能呢？因为测试的时候有可能希望时间保持不变，它提供了一种自定义的方法)；
    String dateTimeProviderRef() default "";
    }
    
    在了解了@EnableJpaAuditing注解之后，我们需要创建一个Configuration 文件，添加 @EnableJpaAuditing 注解，并且把我们的 MyAuditorAware 加载进去即可，如下所示：
        @Configuration
    @EnableJpaAuditing
    public class JpaConfiguration {
       @Bean
       @ConditionalOnMissingBean(name = "myAuditorAware")
       MyAuditorAware myAuditorAware() {
          return new MyAuditorAware();
       }
    }
    
    第四步：我们写个测试用例测试一下。
    @DataJpaTest
    @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    @Import(JpaConfiguration.class)
    public class UserRepositoryTest {
        @Autowired
        private UserRepository userRepository;
        @MockBean
        MyAuditorAware myAuditorAware;
        @Test
        public void testAuditing() {
            //由于测试用例模拟web context环境不是我们的重点，我们这里利用@MockBean，mock掉我们的方法，期待返回13这个用户ID
            Mockito.when(myAuditorAware.getCurrentAuditor()).thenReturn(Optional.of(13));
            //我们没有显式的指定更新时间、创建时间、更新人、创建人
            User user = User.builder()
                    .name("jack")
                    .email("123456@126.com")
                    .sex(SexEnum.BOY)
                    .age(20)
                    .build();
            userRepository.save(user);
            //验证是否有创建时间、更新时间，UserID是否正确；
            List<User> users = userRepository.findAll();
            Assertions.assertEquals(13,users.get(0).getCreateUserId());
            Assertions.assertNotNull(users.get(0).getLastModifiedTime());
            System.out.println(users.get(0));
        }
    }
    
    ```
    
    ```java
    第二种方式：实体里面实现Auditable 接口
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "addresses")
    @EntityListeners(AuditingEntityListener.class)
    public class User implements Auditable<Integer,Long, Instant> {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       private String email;
       @Enumerated(EnumType.STRING)
       private SexEnum sex;
       private Integer age;
       @OneToMany(mappedBy = "user")
       @JsonIgnore
       private List<UserAddress> addresses;
       private Boolean deleted;
       private Integer createUserId;
       private Instant createTime;
       private Integer lastModifiedUserId;
       private Instant lastModifiedTime;
       @Override
       public Optional<Integer> getCreatedBy() {
          return Optional.ofNullable(this.createUserId);
       }
       @Override
       public void setCreatedBy(Integer createdBy) {
          this.createUserId = createdBy;
       }
       @Override
       public Optional<Instant> getCreatedDate() {
          return Optional.ofNullable(this.createTime);
       }
       @Override
       public void setCreatedDate(Instant creationDate) {
          this.createTime = creationDate;
       }
       @Override
       public Optional<Integer> getLastModifiedBy() {
          return Optional.ofNullable(this.lastModifiedUserId);
       }
       @Override
       public void setLastModifiedBy(Integer lastModifiedBy) {
          this.lastModifiedUserId = lastModifiedBy;
       }
       @Override
       public void setLastModifiedDate(Instant lastModifiedDate) {
          this.lastModifiedTime = lastModifiedDate;
       }
       @Override
       public Optional<Instant> getLastModifiedDate() {
          return Optional.ofNullable(this.lastModifiedTime);
       }
       @Override
       public boolean isNew() {
          return id==null;
       }
    }
    ```
    
    ```java
    第三种方式：利用 @MappedSuperclass 注解
    第一步：创建一个 BaseEntity，里面放一些实体的公共字段和注解。
    package com.example.jpa.example1.base;
    import org.springframework.data.annotation.*;
    import javax.persistence.MappedSuperclass;
    import java.time.Instant;
    @Data
    @MappedSuperclass
    @EntityListeners(AuditingEntityListener.class)
    public class BaseEntity {
       @CreatedBy
       private Integer createUserId;
       @CreatedDate
       private Instant createTime;
       @LastModifiedBy
       private Integer lastModifiedUserId;
       @LastModifiedDate
       private Instant lastModifiedTime;
    }
    注意： BaseEntity里面需要用上面提到的四个注解，并且加上@EntityListeners(AuditingEntityListener.class)，这样所有的子类就不需要加了。
    
    第二步：实体直接继承 BaseEntity 即可
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "addresses")
    public class User extends BaseEntity {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
       private String name;
       private String email;
       @Enumerated(EnumType.STRING)
       private SexEnum sex;
       private Integer age;
       @OneToMany(mappedBy = "user")
       @JsonIgnore
       private List<UserAddress> addresses;
       private Boolean deleted;
    }
    ```
  
* Java Persistence API 里面规定的回调方法有哪些？

  * @PrePersist

  * @PostPersist

  * @PreRemove

  * @PostRemove

  * @PreUpdate

  * @PostUpdate

  * @PostLoad

    ![image (5).png](https://s0.lgstatic.com/i/image/M00/62/8F/Ciqc1F-SoLyAODuaAADhS0Urg_0032.png)

  * **语法注意事项**

    * 回调函数都是和 EntityManager.flush 或 EntityManager.commit 在同一个线程里面执行的，只不过调用方法有先后之分，都是同步调用，所以当任何一个回调方法里面发生异常，都会触发事务进行回滚，而不会触发事务提交。
    * Callbacks 注解可以放在实体里面，可以放在 super-class 里面，也可以定义在 entity 的 listener 里面，但需要注意的是：放在实体（或者 super-class）里面的方法，签名格式为“void ()”，即没有参数，方法里面操作的是 this 对象自己；放在实体的 EntityListener 里面的方法签名格式为“void (Object)”，也就是方法可以有参数，参数是代表用来接收回调方法的实体。
    * 使上述注解生效的回调方法可以是 public、private、protected、friendly 类型的，但是不能是 static 和 finnal 类型的方法。

  * 实操

    ```java
    第一种用法：在实体和 super-class 中使用
    第一步：修改 BaseEntity，在里面新增回调函数和注解，代码如下:
    import lombok.Data;
    import org.springframework.data.annotation.*;
    import org.springframework.data.jpa.domain.support.AuditingEntityListener;
    import javax.persistence.*;
    import java.time.Instant;
    @Data
    @MappedSuperclass
    @EntityListeners(AuditingEntityListener.class)
    public class BaseEntity {
       @Id
       @GeneratedValue(strategy= GenerationType.AUTO)
       private Long id;
    // @CreatedBy 这个可能会被 AuditingEntityListener覆盖，为了方便测试，我们先注释掉
       private Integer createUserId;
       @CreatedDate
       private Instant createTime;
       @LastModifiedBy
       private Integer lastModifiedUserId;
       @LastModifiedDate
       private Instant lastModifiedTime;
    //  @Version 由于本身有乐观锁机制，这个我们测试的时候先注释掉，改用手动设置的值；
       private Integer version;
       @PreUpdate
       public void preUpdate() {
          System.out.println("preUpdate::"+this.toString());
          this.setCreateUserId(200);
       }
       @PostUpdate
       public void postUpdate() {
          System.out.println("postUpdate::"+this.toString());
       }
       @PreRemove
       public void preRemove() {
          System.out.println("preRemove::"+this.toString());
       }
       @PostRemove
       public void postRemove() {
          System.out.println("postRemove::"+this.toString());
       }
       @PostLoad
       public void postLoad() {
          System.out.println("postLoad::"+this.toString());
       }
    }
    第二步：修改一下 User 类，也新增两个回调函数，并且和 BaseEntity 做法一样，代码如下：
    import com.example.jpa.example1.base.BaseEntity;
    import com.fasterxml.jackson.annotation.JsonIgnore;
    import lombok.*;
    import javax.persistence.*;
    import java.util.List;
    @Entity
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    @ToString(exclude = "addresses",callSuper = true)
    @EqualsAndHashCode(callSuper=false)
    public class User extends BaseEntity {// implements Auditable<Integer,Long, Instant> {
       private String name;
       private String email;
       @Enumerated(EnumType.STRING)
       private SexEnum sex;
       private Integer age;
       @OneToMany(mappedBy = "user")
       @JsonIgnore
       private List<UserAddress> addresses;
       private Boolean deleted;
       @PrePersist
       private void prePersist() {
          System.out.println("prePersist::"+this.toString());
          this.setVersion(1);
       }
       @PostPersist
       public void postPersist() {
          System.out.println("postPersist::"+this.toString());
       }
    }
    
    第二种用法：自定义 EntityListener
    第一步：自定义一个 EntityLoggingListenner 用来记录操作日志，通过 listener 的方式配置回调函数注解，代码如下:
    import com.example.jpa.example1.User;
    import lombok.extern.log4j.Log4j2;
    import javax.persistence.*;
    @Log4j2
    public class EntityLoggingListener {
        @PrePersist
        private void prePersist(BaseEntity entity) {
            log.info("prePersist::{}",entity.toString());
        }
        @PostPersist
        public void postPersist(Object entity) {
            log.info("postPersist::{}",entity.toString());
        }
        @PreUpdate
        public void preUpdate(BaseEntity entity) {
            log.info("preUpdate::{}",entity.toString());
        }
        @PostUpdate
        public void postUpdate(Object entity) {
            log.info("postUpdate::{}",entity.toString());
        }
        @PreRemove
        public void preRemove(Object entity) {
            log.info("preRemove::{}",entity.toString());
        }
        @PostRemove
        public void postRemove(Object entity) {
            log.info("postRemove::{}",entity.toString());
        }
        @PostLoad
        public void postLoad(Object entity) {
        //查询方法里面可以对一些敏感信息做一些日志
            if (User.class.isInstance(entity)) {
                log.info("postLoad::{}",entity.toString());
            }
        }
    }
    
    
    如果在 @PostLoad 里面记录日志，不一定每个实体、每次查询都需要记录日志，只需要对一些敏感的实体或者字段做日志记录即可。
    
    回调函数时我们可以加上参数，这个参数可以是父类 Object，可以是 BaseEntity，也可以是具体的某一个实体；我推荐用 BaseEntity，因为这样的方法是类型安全的，它可以约定一些框架逻辑，比如 getCreateUserId、getLastModifiedUserId 等。
    ```

  * 关于 @EntityListeners 加载顺序的说明

    * 默认如果子类和父类都有 EntityListeners，那么 listeners 会按照加载的顺序执行所有 EntityListeners；

    * EntityListeners 和实体里面的回调函数注解可以同时使用，但需要注意顺序问题；

    * 如果我们不想加载super-class里面的EntityListeners，那么我们可以通过注解 @ExcludeSuperclassListeners，排除所有父类里面的实体监听者，需要用到的时候，我们再在子类实体里面重新引入即可，代码如下：

      ```java
      @ExcludeSuperclassListeners
      public class User extends BaseEntity {
      ......}
      ```

* JPA Callbacks 的最佳实践

  * 注意回调函数方法要在同一个事务中进行，异常要可预期，非可预期的异常要进行捕获，以免出现意想不到的线上 Bug；

  * 回调函数方法是同步的，如果一些计算量大的和一些耗时的操作，可以通过发消息等机制异步处理，以免阻塞主流程，影响接口的性能。比如上面说的日志，如果我们要将其记录到数据库里面，可以在回调方法里面发个消息，改进之后将变成如下格式：

    ```java
    public class AuditLoggingListener {
       @PostLoad
       private void postLoad(Object entity) {
          this.notice(entity, OperateType.load);
       }
       @PostPersist
       private void postPersist(Object entity) {
          this.notice(entity, OperateType.create);
       }
       @PostRemove
       private void PostRemove(Object entity) {
          this.notice(entity, OperateType.remove);
       }
       @PostUpdate
       private void PostUpdate(Object entity) {
          this.notice(entity, OperateType.update);
       }
       private void notice(Object entity, OperateType type) {
          //我们通过active mq 异步发出消息处理事件
          ActiveMqEventManager.notice(new ActiveMqEvent(type, entity));
       }
       @Getter
       enum OperateType {
          create("创建"), remove("删除"),update("修改"),load("查询");
          private final String description;
          OperateType(String description) {
             this.description=description;
          }
       }
    }
    ```

  * 在回调函数里面，尽量不要直接在操作 EntityManager 后再做 session 的整个生命周期的其他持久化操作，以免破坏事务的处理流程；也不要进行其他额外的关联关系更新动作，业务性的代码一定要放在 service 层面，否则太过复杂，时间长了代码很难维护；

  * 回调函数里面比较适合用一些计算型的transient方法，如下面这个操作：

    ```java
    public class UserListener {
        @PrePersist
        public void prePersist(User user) {
            //通过一些逻辑计算年龄；
            user.calculationAge();
        }
    }
    ```

* 乐观锁

  * 乐观锁在实际开发过程中很常用，它没有加锁、没有阻塞，在多线程环境以及高并发的情况下 CPU 的利用率是最高的，吞吐量也是最大的。

  * 而 Java Persistence API 协议也对乐观锁的操作做了规定：**通过指定 @Version 字段对数据增加版本号控制，进而在更新的时候判断版本号是否有变化。如果没有变化就直接更新；如果有变化，就会更新失败并抛出“OptimisticLockException”异常**。我们用 SQL 表示一下乐观锁的做法，代码如下：

    ```sql
    select uid,name,version from user where id=1;
    update user set name='jack', version=version+1 where id=1 and version=1
    ```

  * 乐观锁的实现方法

    * JPA 协议规定，想要实现乐观锁可以通过 @Version 注解标注在某个字段上面，并且可以持久化到 DB 即可。其支持的类型有如下四种：
      * ` int`or`Integer`
      * ` short`or`Short`
      * ` long`or`Long`
      * ` java.sql.Timestamp`
      * 注意：Spring Data JPA 里面有两个 @Version 注解，请使用 **@javax.persistence.Version**，而不是 @org.springframework.data.annotation.Version。
    * 注意：乐观锁异常不仅仅是同一个方法多线程才会出现的问题，我们只是为了方便测试而采用同一个方法；不同的方法、不同的项目，都有可能导致乐观锁异常。乐观锁的本质是 SQL 层面发生的，和使用的框架、技术没有关系。

* Spring 支持的重试机制

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

    *  include：和 value 一样，默认为空，当 exclude 也为空时，默认异常；

    *  exclude：指定不处理的异常；

    *  backoff：重试等待策略，默认使用 @Backoff@Backoff 的 value，默认为 1s

      * value=delay：隔多少毫秒后重试，默认为 1000L，单位是毫秒；
      * multiplier（指定延迟倍数）默认为 0，表示固定暂停 1 秒后进行重试，如果把 multiplier 设置为 1.5，则第一次重试为 2 秒，第二次为 3 秒，第三次为 4.5 秒。
    
    ```
    @Service 
    public interface MyService { 
      @Retryable( value = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
                backoff = @Backoff(delayExpression = "${retry.maxDelay}")) 
      void retryServiceWithExternalizedConfiguration(String sql) throws SQLException; 
    }
    ```
    
    * https://github.com/spring-projects/spring-retry
    
    ```
    @Retryable(value = ObjectOptimisticLockingFailureException.class,backoff = @Backoff(multiplier = 1.5,random = true))
    ```

  ```
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

* JPA 对 Web MVC 

  * 支持在 Controller 层直接返回实体，而不使用其显式的调用方法；
  *  对 MVC 层支持标准的分页和排序功能；
  * 扩展的插件支持 Querydsl，可以实现一些通用的查询逻辑。

  ```
  @Configuration
  @EnableWebMvc
  //开启支持Spring Data Web的支持
  @EnableSpringDataWebSupport
  public class WebConfiguration { }
  ```

  * DomainClassConverter 组件

    * 这个组件的主要作用是帮我们把 Path 中 ID 的变量，或 Request 参数中的变量 ID 的参数值，直接转化成实体对象注册到 Controller 方法的参数里面。

  * @DynamicUpdate & @DynamicInsert 详解

    ```
    @DynamicInsert：这个注解表示 insert 的时候，会动态生产 insert SQL 语句，其生成 SQL 的规则是：只有非空的字段才能生成 SQL。
    @Target( TYPE )
    @Retention( RUNTIME )
    public @interface DynamicInsert {
       //默认是true，如果设置成false，就表示空的字段也会生成sql语句；
       boolean value() default true;
    }
    这个注解主要是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 insert SQL 的 Columns 只包含非空的字段；如果实体中不加这个注解，默认的情况是空的，字段也会作为 insert 语句里面的 Columns。
    
    @DynamicUpdate：和 insert 是一个意思，只不过这个注解指的是在 update 的时候，会动态产生 update SQL 语句，生成 SQL 的规则是：只有非空的字段才会生成到 update SQL 的 Columns 里面
    @Target( TYPE )
    @Retention( RUNTIME )
    public @interface DynamicUpdate {
       //和insert里面一个意思，默认true;
       boolean value() default true;
    }
    和上一个注解的原理类似，这个注解也是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 update SQL 的 Columns 只包含非空的字段；如果不加这个注解，默认的情况是空的字段也会作为 update 语句里面的 Columns。
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

  ```java
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
  ```

  *  用 Result 对 JSON 的返回结果进行统一封装

    ```
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

  * 数据源

    ```yaml
    https://github.com/brettwooldridge/HikariCP
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

  * Granfan 图表或者 Prometheus 

    * https://github.com/prometheus-operator/prometheus-operator

* Naming 命名策略详解及其实践

  * 第一步：通过ImplicitNamingStrategy先找到实例里面定义的逻辑的字段名字。

    ```java
    这是通过ImplicitNamingStrategy 的实现类指定逻辑字段查找策略，也就是当实体里面定义了 @Table、@Column 注解的时候，以注解指定名字返回；而当没有这些注解的时候，返回的是实体里面的字段的名字。
    其中，org.hibernate.boot.model.naming.ImplicitNamingStrategy 是一个接口，ImplicitNamingStrategyJpaCompliantImpl 这个实现类兼容 JPA 2.0 的字段映射规范。除此之外，还有如下四个实现类：
    * ImplicitNamingStrategyLegacyHbmImpl：兼容 Hibernate 老版本中的命名规范；
    * ImplicitNamingStrategyLegacyJpaImpl：兼容 JPA 1.0 规范中的命名规范；
    * ImplicitNamingStrategyComponentPathImpl：@Embedded 等注解标志的组件处理是通过 attributePath 完成的，因此如果我们在使用 @Embedded 注解的时候，如果要指定命名规范，可以直接继承这个类来实现；
    * SpringImplicitNamingStrategy：默认的 spring data 2.2.3 的策略，只是扩展了 ImplicitNamingStrategyJpaCompliantImpl 里面的 JoinTableName 的方法
    ```

  * 第二步：通过 PhysicalNamingStrategy 将逻辑字段转化成数据库的物理字段名字。

    ```
    它的实现类负责将逻辑字段转化成带下划线，或者统一给字段加上前缀，又或者加上双引号等格式的数据库字段名字，其主要的接口是：org.hibernate.boot.model.naming.PhysicalNamingStrategy，而它的实现类也只有两个.
    
    PhysicalNamingStrategyStandardImpl：这个类什么都没干，即直接将第一个步骤得到的逻辑字段名字当成数据库的字段名字使用。这个主要的应用场景是，如果某些字段的命名格式不是下划线的格式，我们想通过 @Column 的方式显示声明的话，可以把默认第二步的策略改成 PhysicalNamingStrategyStandardImpl。
    userInfo -> userInfo
    id->id
    ages->ages
    lastName -> lastName
    myAddress -> myAddress
    
    SpringPhysicalNamingStrategy：这个类是将第一步得到的逻辑字段名字的大写字母前面加上下划线，并且全部转化成小写，将会标识出是否需要加上双引号。此种是默认策略。
    userInfo -> user_info
    id->id
    ages->ages
    lastName -> last_name
    myAddress -> my_address
    ```

  * 加载原理与自定义方法

    ```
    如果我们修改默认策略，只需要在 application.properties 里面修改下面代码所示的两个配置，换成自己的自定义的类即可。
    spring.jpa.hibernate.naming.implicit-strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
    spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
    ```

* **生产环境多数据源的处理方法**

  * 第一种方式：多个数据源的 @Configuration 的配置方法

    ```
    这种方式的主要思路是，不同 Package 下面的实体和 Repository 采用不同的 Datasource。所以我们改造一下我们的 example 目录结构，来看看不同 Repositories 的数据源是怎么处理的。
    ```

  * **第一步：规划 Entity 和 Repository 的目录结构，为了方便配置多数据源。**

    ```
    将 User 和 UserAddress、UserRepository 和 UserAddressRepository 移动到 db1 里面；将 UserInfo 和 UserInfoRepository 移动到 db2 里面。
    
    我们把实体和 Repository 分别放到了 db1 和 db2 两个目录里面，这时我们假设数据源 1 是 MySQL，User 表和 UserAddress 在数据源 1 里面，那么我们需要配置一个 DataSource1 的 Configuration 类，并且在里面配置 DataSource、TransactionManager 和 EntityManager。
    ```

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

  * **第三步：配置 DataSource2Config类，加载数据源 2。**

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

    ```
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

* Datasource 与 TransactionManager、EntityManagerFactory 的关系分析

  ```
  HikariDataSource 负责实现 DataSource，交给 EntityManager 和 TransactionManager 使用；
  
  EntityManager 是利用 Datasouce 来操作数据库，而其实现类是 SessionImpl；
  
  EntityManagerFactory 是用来管理和生成 EntityManager 的，而 EntityManagerFactory 的实现类是 LocalContainerEntityManagerFactoryBean，通过实现 FactoryBean 接口实现，利用了 FactoryBean 的 Spring 中的 bean 管理机制，所以需要我们在 Datasource1Config 里面配置 LocalContainerEntityManagerFactoryBean 的 bean 的注入方式；
  
  JpaTransactionManager 是用来管理事务的，实现了 TransactionManager 并且通过 EntityFactory 和 Datasource 进行 db 操作，所以我们要在 DataSourceConfig 里面告诉 JpaTransactionManager 用的 TransactionManager 是 db1EntityManagerFactory。
  ```

* 第二种方式：利用 AbstractRoutingDataSource 配置多数据源

  ```java
  第一步：定一个数据源的枚举类，用来标示数据源有哪些。
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
  第二步：新增 DataSourceRoutingHolder，用来存储当前线程需要采用的数据源。
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
  第三步：配置 RoutingDataSourceConfig，用来指定哪些 Entity 和 Repository 采用动态数据源。
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
  
  第四步：写一个 MVC 拦截器，用来指定请求分别采用什么数据源。
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

*  多数据源实战注意事项

  ```
  此种方式利用了当前线程事务不变的原理，所以要注意异步线程的处理方式；
  
  此种方式利用了 DataSource 的原理，动态地返回不同的 db 连接，一般需要在开启事务之前使用，需要注意事务的生命周期；
  
  比较适合读写操作分开的业务场景；
  
  多数据的情况下，避免一个事务里面采用不同的数据源，这样会有意想不到的情况发生，比如死锁现象；
  
  学会通过日志检查我们开启请求的方法和开启的数据源是否正确，可以通过 Debug 断点来观察数据源是否选择的正确
  ```

* 微服务下的实战建议

  ```
  微服务的大环境下，服务越小，内聚越高，低耦合服务越健壮，所以一般跨库之间一定是是通过 REST 的 API 协议，进行内部服务之间的调用，这是最稳妥的方式，原因有如下几点：
  
  REST 的 API 协议更容易监控，更容易实现事务的原子性；
  
  db 之间解耦，使业务领域代码职责更清晰，更容易各自处理各种问题；
  
  只读和读写的 API 更容易分离和管理。
  ```

* 事务

  * 事务最主要的作用是保证数据 ACID 的特性，即原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）.

    * **原子性**： 是指一个事务（Transaction）中的所有操作，要么全部完成，要么全部回滚，而不会有中间某个数据单独更新的操作。事务在执行过程中一旦发生错误，会被回滚（Rollback）到此次事务开始之前的状态，就像这个事务从来没有执行过一样。
    * **一致性**： 是指事务操作开始之前，和操作异常回滚以后，数据库的完整性没有被破坏。数据库事务 Commit 之后，数据也是按照我们预期正确执行的。即要通过事务保证数据的正确性。
    * **持久性**： 是指事务处理结束后，对数据的修改进行了持久化的永久保存，即便系统故障也不会丢失，其实就是保存到硬盘。
    * **隔离性**： 是指数据库允许多个连接，同时并发多个事务，又对同一个数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时，由于交叉执行而导致数据不一致的现象。而 MySQL 里面就是我们经常说的事务的四种隔离级别，即读未提交（Read Uncommitted）、读提交（Read Committed）、可重复读（Repeatable Read）和串行化（Serializable）。

  * 事务的隔离级别

    * **Read Uncommitted（读取未提交内容）**：此隔离级别，表示所有正在进行的事务都可以看到其他未提交事务的执行结果。不同的事务之间读取到其他事务中未提交的数据，通常这种情况也被称之为脏读（Dirty Read），会造成数据的逻辑处理错误，也就是我们在多线程里面经常说的数据不安全了。在业务开发中，几乎很少见到使用的，因为它的性能也不比其他级别要好多少。
    * **Read Committed（读取提交内容）**： 此隔离级别是指，在一个事务相同的两次查询可能产生的结果会不一样，也就是第二次查询能读取到其他事务已经提交的最新数据。也就是我们常说的**不可重复读（Nonrepeatable Read）**的事务隔离级别。因为同一事务的其他实例在该实例处理期间，可能会对其他事务进行新的 commit，所以在同一个事务中的同一 select 上，多次执行可能返回不同结果。这是大多数数据库系统的默认隔离级别（但不是 MySQL 默认的隔离级别）。
    * **Repeatable Read（可重读）**： 这是 MySQL 的默认事务隔离级别，它确保同一个事务多次查询相同的数据，能读到相同的数据。即使多个事务的修改已经 commit，本事务如果没有结束，永远读到的是相同数据，要注意它与Read Committed 的隔离级别的区别，是正好相反的。这会导致另一个棘手的问题：**幻读 （Phantom Read）**，即读到的数据可能不是最新的。这个是最常见的，我们举个例子来说明。
    * **Serializable（可串行化）**：这是最高的隔离级别，它保证了每个事务是串行执行的，即强制事务排序，所有事务之间不可能产生冲突，从而解决幻读问题。如果配置在这个级别的事务，处理时间比较长，并发比较大的时候，就会导致大量的 db 连接超时现象和锁竞争，从而降低了数据处理的吞吐量。也就是这个性能比较低，所以除了某些财务系统之外，用的人不是特别多。

  * MySQL 事务与连接的关系

    * 事务必须在同一个连接里面的，离开连接没有事务可言；
    * MySQL 数据库默认 autocommit=1，即每一条 SQL 执行完自动提交事务；
    * 数据库里面的每一条 SQL 执行的时候必须有事务环境；
    * MySQL 创建连接的时候默认开启事务，关闭连接的时候如果存在事务没有 commit 的情况，则自动执行 rollback 操作；
    * 不同的 connect 之间的事务是相互隔离的。

  * MySQL 事务的两种操作方式

    * 第一种：用 BEGIN、ROLLBACK、COMMIT 来实现。
      * BEGIN开始一个事务
      * ROLLBACK事务回滚
      * COMMIT事务确认
    * 第二种：直接用 SET 来改变 MySQL 的自动提交模式。
      * SET AUTOCOMMIT=0禁止自动提交
      * SET AUTOCOMMIT=1开启自动提交

  * MySQL 数据库的最大连接数

    * `show variables like 'max_connections' `查看此数据库的最大连接数、通过 `show global status like 'Max_used_connections'` 查看正在使用的连接数，还可以通过 `set global max_connections=1500` 来设置数据库的最大连接数。
    * 默认连接超时时间8小时;

  * Spring里面事务的配置方法

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

      ![图片1.png](https://s0.lgstatic.com/i/image/M00/6E/36/Ciqc1F-yC3-AP_fhAArdN5coRWQ007.png)

    * 隔离级别

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

      * @Transactional 的局限性

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

        * 解决方法: 可以引入一个类 TransactionTemplate

          ```java
          public UserInfo save(Long userId) {
             return transactionTemplate.execute(status -> this.calculate(userId));
          }
          ```

        * 自定义 TransactionHelper

          ```java
          第一步：新建一个 TransactionHelper 类，进行事务管理，代码如下。
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
          第二步：直接在 service 中就可以使用了，代码如下。
          @Autowired
          private TransactionHelper transactionHelper;
          /**
          * 调用外部的transactionHelper类，利用transactionHelper方法上面的@Transaction注解使事务生效
          */
          public UserInfo save(Long userId) {
             return transactionHelper.transactional((uid)->this.calculate(uid),userId);
          }
          ```

      * 隐式事务 / AspectJ 事务配置

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

      * 通过日志分析配置方法的过程

        * 第一步，在数据连接中加上 logger=Slf4JLogger&profileSQL=true，用来显示 MySQL 执行的 SQL 日志

        * 第二步，打开 Spring 的事务处理日志，用来观察事务的执行过程.

          ```
          # Log Transactions Details
          logging.level.org.springframework.orm.jpa=DEBUG
          logging.level.org.springframework.transaction=TRACE
          logging.level.org.hibernate.engine.transaction.internal.TransactionImpl=DEBUG
          # 监控连接的情况
          logging.level.org.hibernate.resource.jdbc=trace
          logging.level.com.zaxxer.hikari=DEBUG
          ```

        * 第三步，执行一个 saveOrUpdate 的操作，详细的执行日志

### 3. 原理

* JpaProperties 属性

  ```
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

* Debug 时候，日志的配置

  ```
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

* @PersistenceUnit  @PersistenceContext

  >EntityManagerFactory 和 Persistence Unit 是什么？
  >按照 JPA 协议里面的定义：persistence unit 是一些持久化配置的集合，里面包含了数据源的配置、EntityManagerFactory 的配置，spring 3.1 之前主要是通过 persistence.xml 的方式来配置一个 persistence unit。
  >
  >
  >
  >EntityManager 和 PersistenceContext 是什么？
  >按照 JPA 协议的规范，我们先理解一下 PersistenceContext，它是用来管理会话里面的 Entity 状态的一个上下文环境，使 Entity 的实例有了不同的状态，也就是我们所说的实体实例的生命周期。
  >
  >而这些实体在 PersistenceContext 中的不同状态都是通过 EntityManager 提供的一些方法进行管理的，也就是说：
  >
  >PersistenceContext 是持久化上下文，是 JPA 协议定义的，而 Hibernate 的实现是通过 Session 创建和销毁的，也就是说一个 Session 有且仅有一个 PersistenceContext；
  >
  >PersistenceContext 既然是持久化上下文，里面管理的是 Entity 的状态；
  >
  >EntityManager 是通过 PersistenceContext 创建的，用来管理 PersistenceContext 中 Entity 状态的方法，离开 PersistenceContext 持久化上下文，EntityManager 没有意义；
  >
  >EntityManger 是操作对象的唯一入口，一个请求里面可能会有多个 EntityManger 对象。

* 实体对象的生命周期

  > 既然 PersistenceContext 是存储 Entity 的，那么 Entity 在 PersistenceContext 里面肯定有不同的状态。对此，JPA 协议定义了四种状态：new、manager、detached、removed。
  >
  > ![Drawing 0.png](https://s0.lgstatic.com/i/image/M00/70/03/CgqCHl-3m3OAPiQmAAB8FdvAFnE298.png)
  >
  > 第一种：New 状态的对象
  >
  > 当我们使用关键字 new 的时候创建的实体对象，称为 new 状态的 Entity 对象。它需要同时满足两个条件：new 状态的实体 Id 和 Version 字段都是 null；new 状态的实体没有在 PersistenceContext 中出现过。
  >
  > 那么如果我们要把 new 状态的 Entity 放到 PersistenceContext 里面，有两种方法：执行 entityManager.persist(entity) 方法；通过关联关系的实体关系配置 cascade=PERSIST or cascade=ALL 这种类型，并且关联关系的一方，也执行了 entityManager.persist(entity) 方法。
  >
  > 
  >
  > 第二种：Detached（游离）的实体对象
  >
  > Detached 状态的对象表示和 PersistenceContext 脱离关系的 Entity 对象。它和 new 状态的对象的不同点在于：
  >
  > * Detached 是 new 状态的实体对象没有持久化 ID（即没有 ID 和 version）；
  >
  > * 变成持久化对象需要进行 merger 操作，merger 操作会 copy 一个新的实体对象，然后把新的实体对象变成 Manager 状态。  
  >
  > 而 Detached 和 new 状态的对象相同点也有两个方面：
  >
  > * 都和 PersistenceContext 脱离了关系； 
  >
  > * 当执行 flush 操作或者 commit 操作的时候，不会进行数据库同步。
  >
  >   
  >
  > 第三种：Manager（persist） 状态的实体
  >
  > Manager 状态的实体，顾名思义，是指在 PersistenceContext 里面管理的实体，而此种状态的实体当我们执行事务的 commit()，或者 entityManager 的 flush 方法的时候，就会进行数据库的同步操作。可以说是和数据库的数据有映射关系。
  >
  > 
  >
  > 第四种：Removed 的实体状态
  >
  > Removed 的状态，顾名思义就是指删除了的实体，但是此实体还在 PersistenceContext 里面，只是在其中表示为 Removed 的状态，它和 Detached 状态的实体最主要的区别就是不在 PersistenceContext 里面，但都有 ID 属性。

* MyBatis 是对数据库的操作所见即所得的模式；而使用 JPA，你的任何操作都不会产生 DB 的sql。

* Flush 的作用

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
        *  `EntityInsertAction`or`EntityIdentityInsertAction`
        * ` EntityUpdateAction`
        * ` CollectionRemoveAction`
        *  `CollectionUpdateAction`
        *  `CollectionRecreateAction`
        *  `EntityDeleteAction`

    * Flush 与事务 Commit 的关系

      * 在当前的事务执行 commit 的时候，会触发 flush 方法；
      * 在当前的事务执行完 commit 的时候，如果隔离级别是可重复读的话，flush 之后执行的 update、insert、delete 的操作，会被其他的新事务看到最新结果；
      * 假设当前的事务是可重复读的，当我们手动执行 flush 方法之后，没有执行事务 commit 方法，那么其他事务是看不到最新值变化的，但是最新值变化对当前没有 commit 的事务是有效的；
      * 如果执行了 flush 之后，当前事务发生了 rollback 操作，那么数据将会被回滚（数据库的机制）。
  
* Session、EntityManager、Connection 和 Transaction 的关系

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
  
* **N+1SQL问题**

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
  所谓的 N+1 的 SQL，此时 1 代表的是一条 SQL 查询 UserInfo 信息；N 条 SQL 查询 Address 的信息。
  ```

  * 解决方法一:

    * hibernate.default_batch_fetch_size 配置

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

        ```
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

      * FetchMode.SELECT

        ```
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

      * FetchMode.JOIN

        ```
        public class UserInfo extends BaseEntity {
           private String name;
           private String telephone;
           @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)
           @Fetch(value = FetchMode.JOIN) //唯一变化的地方采用JOIN模式
           private List<Address> addressList;
        }
        ```

      * FetchMode.SUBSELECT

        ```
        public class UserInfo extends BaseEntity {
           @OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.LAZY) //我们这里测试一下LAZY情况
           @Fetch(value = FetchMode.SUBSELECT) //唯一变化之处
           private List<Address> addressList;
        }
        ```

      * FetchMode.SUBSELECT 支持 ID 查询和各种条件查询，唯一的缺点是只能配置在 @OneToMany 和 @ManyToMany 的关联关系上，不能配置在 @ManyToOne 和 @OneToOne 的关联关系上

      * @Fetch 的不同模型，都有各自的优缺点：FetchMode.SELECT 默认，和不配置的效果一样；FetchMode.JOIN 只支持类似 findById(id) 的方法，只能根据 ID 查询才有效果；FetchMode.SUBSELECT 虽然不限使用方式，但是只支持 **ToMany 的关联关系。

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
      @Override //可以覆盖原始方法，添加上不同的@EntityGraph策略
      //使用@EntityGraph查询所有Address的时候，指定name = "getAllUserInfo"的@NamedEntityGraph，采用默认的EntityGraphType.FETCH，如果Address里面有多个关联关系的时候，只有在name = "getAllUserInfo"的实体图配置的userInfo属性上采用Eager模式，其他关联关系属性没有指定，默认采用LAZY模式；
      @EntityGraph(value = "getAllUserInfo")
      List<Address> findAll();
      }
      ```

* JPA  @Query 中SpEL的应用场景

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

* spring-security-data 在 @Query 中的用法

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

* 解决 In 查询条件内存泄漏的方法

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

* @Cacheable

    * 应用到读取数据的方法上，就是可以缓存的方法，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中。

        ```java
        public @interface Cacheable {
           @AliasFor("cacheNames")
           String[] value() default {};
        //cache的名字。可以根据名字设置不同cache处理类。redis里面可以根据cache名字设置不同的失效时间。
           @AliasFor("value")
           String[] cacheNames() default {};
        //缓存的key的名字，支持spel
           String key() default "";
        //key的生成策略，不指定可以用全局的默认的。
           String keyGenerator() default "";
           //客户选择不同的CacheManager
           String cacheManager() default "";
           //配置不同的cache resolver
           String cacheResolver() default "";
           //满足什么样的条件才能被缓存，支持SpEL，可以去掉方法名、参数
           String condition() default "";
        //排除哪些返回结果不加入缓存里面去，支持SpEL，实际工作中常见的是result ==null等
           String unless() default "";
           //是否同步读取缓存、更新缓存
           boolean sync() default false;
        }
        
        @Cacheable(cacheNames="book", condition="#name.length() < 32", unless="#result.notNeedCache")//利用SPEL表达式只有当name参数长度小于32的时候再进行缓存，排除notNeedCache的对象
        public Book findBook(String name)
        ```

    * @CachePut

        * 调用方法时会自动把相应的数据放入缓存，它与 @Cacheable 不同的是所有注解的方法每次都会执行，一般配置在 Update 和 insert 方法上。其源码里面的字段和用法基本与 @Cacheable 相同，只是使用场景不一样.

    * @CacheEvict

        *  删除缓存，一般配置在删除方法上面。

            ```
            public @interface CacheEvict {
            //与@Cacheable相同的部分咱我就不重复叙述了。
            ......
            	//是否删除所有的实体对象
               boolean allEntries() default false;
               //是否方法执行之前执行。默认在方法调用成功之后删除
               boolean beforeInvocation() default false;
            }
            	@Caching 所有Cache注解的组合配置方法，源码如下：
            	public @interface Caching {
               Cacheable[] cacheable() default {};
               CachePut[] put() default {};
               CacheEvict[] evict() default {};
            }
            ```

    * org.springframework.cache.annotation.CachingConfigurerSupport

        * 通过此类可以自定义 Cache 里面的 CacheManager、CacheResolver、KeyGenerator、CacheErrorHandler，

            ```
            public class CachingConfigurerSupport implements CachingConfigurer {
              // cache的manager，主要是管理不同的cache的实现方式，如redis还是ehcache等
               @Override
               @Nullable
               public CacheManager cacheManager() {
                  return null;
               }
               // cache的不同实现者的操作方法，CacheResolver解析器，用于根据实际情况来动态解析使用哪个Cache
               @Override
               @Nullable
               public CacheResolver cacheResolver() {
                  return null;
               }
               //cache的key的生成规则
               @Override
               @Nullable
               public KeyGenerator keyGenerator() {
                  return null;
               }
               //cache发生异常的回调处理，一般情况下我会打印个warn日志，方便知道发生了什么事情
               @Override
               @Nullable
               public CacheErrorHandler errorHandler() {
                  return null;
               }
            }
            ```

    * Spring Cache 结合 Redis 使用的最佳实践

        * 不同 cache 的 name 在 redis 里面配置不同的过期时间

        * 默认情况下所有 redis 的 cache 过期时间是一样的，实际工作中一般需要自定义不同 cache 的 name 的过期时间，我们这里 cache 的 name 就是指 @Cacheable 里面 value 属性对应的值。主要步骤如下。

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
            # 设置我们刚才的例子 @Cacheable(value="userInfo")5分钟过期
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

* Spring Data JPA 单元测试的最佳实践

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

    





