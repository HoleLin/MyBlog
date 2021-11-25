---
title: JPA(二)-注解
date: 2021-08-24 13:13:22
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 注解
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

#### @NonNull、@NonNullApi、@Nullable

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

#### **@Query**

* JpaQueryLookupStrategy 关键源码剖析

  * 先打开 QueryExecutorMethodInterceptor 类，默认的策略是CreateIfNotFound,找到如下代码：

   ![img](https://www.holelin.cn/img/spring/jpa/JpaQueryLookupStrategy.png)

##### @Query 的基本用法

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

##### JPQL的语法

```java
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


##### @Query 的排序

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

##### @Query 的分页

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

##### @Query 之 Projections 应用返回指定 DTO

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

##### @Query 动态查询解决方法

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

![img](https://www.holelin.cn/img/spring/jpa/Query%20%E5%8A%A8%E6%80%81%E6%9F%A5%E8%AF%A2%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%951.png)

![img](https://www.holelin.cn/img/spring/jpa/Query%20%E5%8A%A8%E6%80%81%E6%9F%A5%E8%AF%A2%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%952.png)

#### @Param 用法

* @Param 注解指定方法参数的具体名称，通过绑定的参数名字指定查询条件，这样不需要关心参数的顺序。

#### @Entity

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

#### @Table

* **@Table** 用于指定数据库的表名，表示此实体对应的数据库里面的表名，非必填，默认表名和 entity 名字一样。

  ```
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

#### @Access

* 用于指定 entity 里面的注解是写在字段上面，还是 get/set 方法上面生效，非必填。在默认不填写的情况下，当实体里面的第一个注解出现在字段上或者 get/set 方法上面，就以第一次出现的方式为准；也就是说，一个实体里面的注解既有用在 field 上面，又有用在 properties 上面的时候.

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

#### **@Id**

* 定义属性为数据库的主键，一个实体里面必须有一个主键，但不一定是这个注解，可以和 @GeneratedValue 配合使用或成对出现

#### **@GeneratedValue** 

* 主键生成策略

```
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

#### @Enumerated

* 这个注解很好用，因为它对 enum 提供了下标和 name 两种方式，用法直接映射在 enum 枚举类型的字段上。

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

#### @Basic

* 表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为 @Basic。也就是说默认所有的字段肯定是和数据库进行映射的，并且默认为 Eager 类型。

```java
public @interface Basic {
    //可选，EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面）
    FetchType fetch() default EAGER;
    //可选。这个字段是否可以为null，默认是true。
    boolean optional() default true;
}
```

#### @Transient

* 表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 有相反的作用。也就是每个字段上面 @Transient 和 @Basic 必须二选一，而什么都不指定的话，默认是 @Basic。

#### @Column

* 定义该属性对应数据库中的列名。

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

#### **@Temporal** 

* 用来设置 Date 类型的属性映射到对应精度的字段，存在以下三种情况
  * @Temporal(TemporalType.DATE)映射为日期 // date （**只有日期**）
  * @Temporal(TemporalType.TIME)映射为日期 // time （**只有时间**）
  * @Temporal(TemporalType.TIMESTAMP)映射为日期 // date time （**日期+时间**）

#### 示例

```java
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

#### 联合主键

* 可以通过 javax.persistence.EmbeddedId 和 javax.persistence.IdClass 两个注解实现联合主键的效果.

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

#### @Embeddable 与 @EmbeddedId 注解使用

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

#### `@IdClass` 和 `@EmbeddedId` 的区别是什么？

* 如上面测试用例，在使用的时候，Embedded 用的是对象，而 IdClass 用的是具体的某一个字段；

* 二者的JPQL 也会不一样：

  *  用 `@IdClass` JPQL 的写法：`SELECT u.name FROM UserInfo u`
  *  用 `@EmbeddedId` 的 JPQL 的写法：`SELECT u.userInfoId.name FROM UserInfo u`

* 联合主键还有需要注意的就是，它与唯一性索引约束的区别是写法不同，如上面所讲，唯一性索引的写法如下：

  ```java
  @Column(unique = true)
  private String uniqueNumber;
  ```

#### 实体之间的继承关系如何实现？

* 在 Java 面向对象的语言环境中，@Entity 之间的关系多种多样，而根据 JPA 的规范，我们大致可以将其分为以下几种：
  * 纯粹的继承，和表没关系，对象之间的字段共享。利用注解 @MappedSuperclass，协议规定父类不能是 @Entity。
  * 单表多态问题，同一张 Table，表示了不同的对象，通过一个字段来进行区分。利用`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`注解完成，只有父类有 @Table。
  * 多表多态，每一个子类一张表，父类的表拥有所有公用字段。通过`@Inheritance(strategy = InheritanceType.JOINED)`注解完成，父类和子类都是表，有公用的字段在父表里面。
  * Object 的继承，数据库里面每一张表是分开的，相互独立不受影响。通过`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`注解完成，父类（可以是一张表，也可以不是）和子类都是表，相互之间没有关系。

##### **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**

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

##### @Inheritance(strategy = InheritanceType.JOINED)

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

  ##### @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)

  * 我们在使用 @MappedSuperClass 主键的时候，如果不指定 @Inhertance，默认就是此种TABLE_PER_CLASS模式。当然了，我们也显示指定，要求继承基类的都是一张表，而父类不是表，是 java 对象的抽象类。我们看一个例子。

    ```
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

#### 实体与实体之间的关联关系

  * 实体与实体之间的关联关系一共分为四种，分别为 **OneToOne、OneToMany、ManyToOne 和 ManyToMany**；而实体之间的关联关系又分为双向的和单向的。

##### **@OneToOne 关联关系**

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

* `@interface OneToOne` 源码解读

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

  * 只有关联关系的维护方才能操作两个实体之间外键的关系。被维护方即使设置了维护方属性进行存储也不会更新外键关联。

  * mappedBy 不能与 @JoinColumn 或者 @JoinTable 同时使用，因为没有意义，关联关系不在这里面维护。

  * 此外，mappedBy 的值是指另一方的实体里面属性的字段，而不是数据库字段，也不是实体的对象的名字。也就是维护关联关系的一方属性字段名称，或者加了 @JoinColumn / @JoinTable 注解的属性字段名称。如上面的 User 例子 user 里面 mappedBy 的值，就是 UserInfo 里面的 user 字段的名字。

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
  
#### **@JoinCloumns & JoinColumn**

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

#### @ManyToOne& @OneToMany

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

#### @ManyToMany

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

#### @JoinTable

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

##### 利用 @ManyToOne 和 @OneToMany 表达多对多的关联关系

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

#### Jackson与JPA

##### Jackson核心模块

* **jackson-core：核心包**，提供基于“流模式”解析的相关 API，它包括 JsonPaser 和 JsonGenerator。Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。

* **jackson-annotations：注解包**，提供标准注解功能，这是我们必须要掌握的基础语法。

* **jackson-databind：数据绑定包**，提供基于“对象绑定”解析的相关 API（ ObjectMapper ） 和“树模型”解析的相关 API（JsonNode）；基于“对象绑定”解析的 API 和“树模型”解析的 API 依赖基于“流模式”解析的 API。如下图中一些标准的类型转换：

  ![img](https://www.holelin.cn/img/spring/jpa/Jackson%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97.png)

  ![img](https://www.holelin.cn/img/spring/jpa/Jackson%E6%B3%A8%E8%A7%A3.png)

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

  ```
  public <T> T readValue(String content, Class<T> valueType)
  public <T> T readValue(String content, TypeReference<T> valueTypeRef)
  public <T> T readValue(String content, JavaType valueType)
  ```

  ```
  String json = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(userJson);
  //单个对象的写法：
  UserJson user = objectMapper.readValue(json,UserJson.class);
  //返回List的返回结果的写法：
  List<User> personList2 = mapper.readValue(jsonListString, new TypeReference<List<User>>(){});
  ```

##### Jackson 与 JPA 常见的问题

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

#### **ObjectMapper 实战经验推荐配置项**

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

####   **时间类型的最佳实践，如何返回 ISO 格式的标准时间**

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

#### @Cacheable

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

#### @CachePut

* 调用方法时会自动把相应的数据放入缓存，它与 @Cacheable 不同的是所有注解的方法每次都会执行，一般配置在 Update 和 insert 方法上。其源码里面的字段和用法基本与 @Cacheable 相同，只是使用场景不一样.

#### @CacheEvict

* 删除缓存，一般配置在删除方法上面。

  ```java
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

#### `org.springframework.cache.annotation.CachingConfigurerSupport`

* 通过此类可以自定义 Cache 里面的 CacheManager、CacheResolver、KeyGenerator、CacheErrorHandler

  ```java
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
