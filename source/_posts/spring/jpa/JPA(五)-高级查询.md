---
title: JPA(五)-高级查询
date: 2021-08-24 15:32:18
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 高级查询
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

### 参考文献

* 拉钩教育--Spring Data JPA原理与实战

#### `QueryByExampleExecutor`

* QueryByExampleExecutor（QBE）是一种用户友好的查询技术，具有简单的接口，它允许动态查询创建，并且不需要编写包含字段名称的查询。

<img src="https://www.chenjunlin.vip/img/spring/jpa/QueryByExampleExecutor.png" alt="img" style="zoom:50%;" />

##### QBE 的基本语法

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

* 三个类：Probe、ExampleMatcher 和 Example，分别做如下解释：
  * Probe：这是具有填充字段的域对象的实际实体类，即查询条件的封装类（又可以理解为查询条件参数），必填。
  * ExampleMatcher：ExampleMatcher 有关如何匹配特定字段的匹配规则，它可以重复使用在多个实例中，必填。
  * Example：Example 由 Probe 探针和 ExampleMatcher 组成，它用于创建查询，即组合查询参数和参数的匹配规则。
* 通过 Example 的源码，我们发现想创建 Example 的话，只有两个方法
  * static `<T>` Example`<T>` of(T probe)：需要一个实体参数，即查询的条件。而里面的 ExampleMatcher 采用默认的 ExampleMatcher.matching()； 表示忽略 Null，所有字段采用精准匹配。
  * static `<T>` Example`<T>` of(T probe, ExampleMatcher matcher)：需要两个参数构建 Example，也就表示了 ExampleMatcher 自由组合规则，正如我们上面的测试用例里面的代码一样。

#### **ExampleMatcher** 

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

##### ExampleMatcher 语法暴露的方法

###### **忽略大小写**

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

###### **暴露的 Null 值处理方式如下：**

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

###### 忽略某些 Paths，不参加查询条件

```java
//忽略某些属性列表，不参与查询过滤条件。
ExampleMatcher withIgnorePaths(String... ignoredPaths);
```

###### 字符串字段默认的匹配规则

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

<img src="https://www.chenjunlin.vip/img/spring/jpa/%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8C%B9%E9%85%8D%E8%A7%84%E5%88%99.png" alt="img" style="zoom: 67%;" />

###### 示例

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

#### JpaSpecificationExecutor 使用案例

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

###### `Root<User> root`

* 代表了可以查询和操作的实体对象的根，如果将实体对象比喻成表名，那 root 里面就是这张表里面的字段，而这些字段只是 JPQL 的实体字段而已。我们可以通过里面的 Path get（String attributeName），来获得我们想要操作的字段。

###### `CriteriaQuery<?> query`

* 代表一个 specific 的顶层查询对象，它包含着查询的各个部分，比如 select 、from、where、group by、order by 等。CriteriaQuery 对象只对实体类型或嵌入式类型的 Criteria 查询起作用。简单理解为，它提供了查询 ROOT 的方法。

###### `CriteriaBuilder cb`

* CriteriaBuilder 是用来构建 CritiaQuery 的构建器对象，其实就相当于条件或者条件组合，并以 Predicate 的形式返回.

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

#### EntityManager 

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

##### `@EnableJpaRepositories`

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

##### 通过 @EnableJpaRepositories 定义默认的 Repository 的实现类

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

  * 在实际生产中经常会有这样的场景：对外暴露的是 UUID 查询方法，而对内暴露的是 Long 类型的 ID，这时候我们就可以自定义一个 FindByIdOrUUID 的底层实现方法，可以选择在自定义的 Respository 接口里面实现。

  * Defining Query Methods 和 @Query 满足不了我们的查询，但是我们又想用它的方法语义的时候，就可以考虑实现不同的 Respository 的实现类，来满足我们不同业务场景的复杂查询。

    ```java
    @SQLDelete(sql = "UPDATE user SET deleted = true where deleted =false and id = ?")
    public class User implements Serializable {
    ....
    }
    ```

    
