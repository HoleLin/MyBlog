---
title: JPA(一)-基础理论
date: 2021-08-24 11:52:09
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 基础理论
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

#### 基本使用

* ORM框架对比

  <img src="https://www.chenjunlin.vip/img/spring/jpa/ORM%E6%A1%86%E6%9E%B6%E5%AF%B9%E6%AF%94.png" alt="img" style="zoom: 50%;" />

  ![img](https://www.chenjunlin.vip/img/spring/jpa/SpringDataJPA.png)

#### Spring Data Common的依赖关系

  ![img](https://www.chenjunlin.vip/img/spring/jpa/Spring%20Data%20Common%E7%9A%84%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB.png)

  * 数据库连接用的是**JDBC**
  * 连接池用的是**HikariCP**
  * 强依赖**Hibernate**
  * Spring Boot Starter Data JPA 依赖Spring Data JPA 而Spring Data JPA依赖Spring Data Commons
  
* **Repository接口**

  * **Repository**是Spring Date Common里面的顶级父类接口,操作DB的入口类.

* **Repository类层次关系**

  * **ReactiveCrudRepository** 这条线是响应式编程，主要支持当前 NoSQL 方面的操作，因为这方面大部分操作都是分布式的，所以由此我们可以看出 Spring Data 想统一数据操作的“野心”，即想提供关于所有 Data 方面的操作。目前 Reactive 主要有 Cassandra、MongoDB、Redis 的实现。

  * **RxJava2CrudRepository** 这条线是为了支持 RxJava 2 做的标准响应式编程的接口。

  * **CoroutineCrudRepository** 这条继承关系链是为了支持 Kotlin 语法而实现的。

  * **CrudRepository**

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

#### 接口说明

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

  ```java
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

#### 方法名定义查询方法（Defining Query Methods）

* 一种是直接通过方法名就可以实现;

* 另一种是 `@Query `手动在方法上定义;

* **方法的查询策略设置**

  * `@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)`
  * `QueryLookupStrategy.Key` 的值共 3 个
    * **Create**：直接根据方法名进行创建，规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候会报异常，这种情况可以理解为，即使配置了 @Query 也是没有用的。
    * **USE_DECLARED_QUERY**：声明方式创建，启动的时候会尝试找到一个声明的查询，如果没有找到将抛出一个异常，可以理解为必须配置 @Query。
    * **CREATE_IF_NOT_FOUND**：这个是默认的，除非有特殊需求，可以理解为这是以上 2 种方式的兼容版。先用声明方式（`@Query`）进行查找，如果没有找到与方法相匹配的查询，那用 Create 的方法名创建规则创建一个查询；这两者都不满足的情况下，启动就会报错。

* `Defining Query Method（DQM）`语法

  * 该语法是：带查询功能的方法名由查询策略（关键字）+ 查询字段 + 一些限制性条件组成，具有语义清晰、功能完整的特性
  * `org.springframework.data.repository.query.parser.PartTree `
  * `org.springframework.data.repository.query.parser.Part` 
    <img src="https://www.chenjunlin.vip/img/spring/jpa/DQM.png" alt="img" style="zoom: 50%;" />

##### **特定类型的参数：Sort 排序和 Pageable 分页**

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

##### **限制查询结果 First 和 Top**

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

##### Repository 的返回结果

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

##### Repository 对 Feature/CompletableFuture 异步返回结果的支持

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

  以上是对 @Async 的支持，关于实际使用需要注意以下三点内容：

  - 在实际工作中，直接在 Repository 这一层使用异步方法的场景不多，一般都是把异步注解放在 Service 的方法上面，这样的话，可以有一些额外逻辑，如发短信、发邮件、发消息等配合使用；
  - 使用异步的时候一定要配置线程池，这点切记，否则“死”得会很难看；
  - 万一失败我们会怎么处理？关于事务是怎么处理的呢？这种需要重点考虑的;

  <img src="https://www.chenjunlin.vip/img/spring/jpa/%E5%BC%82%E6%AD%A5%E8%BF%94%E5%9B%9E%E7%BB%93%E6%9E%9C.png" alt="img" style="zoom:67%;" />

##### Projections 的概念

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

    ![img](https://www.chenjunlin.vip/img/spring/jpa/QueryStructure.png)

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
    
    这个时候会发现我们的 userOnlyName 接口成了一个代理对象，里面通过 Map 的格式包含了我们的要返回字段的值（如：name、email），我们用的时候直接调用接口里面的方法即可，如 userOnlyName.getName() 即可；这种方式的优点是接口为只读，并且语义更清晰
    ```

    ![img](https://www.chenjunlin.vip/img/spring/jpa/QueryStructure2.png)
