---
title: JPA(三)-审计功能
date: 2021-08-24 15:50:48
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 审计
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

#### JAP审计功能

* Auditing 是帮我们做审计用的，当我们操作一条记录的时候，需要知道这是谁创建的、什么时间创建的、最后修改人是谁、最后修改时间是什么时候，甚至需要修改记录……这些都是 Spring Data JPA 里面的 Auditing 支持的，它为我们提供了四个注解来完成上面说的一系列事情，如下：

  *  `@CreatedBy` 是哪个用户创建的。

  *  `@CreatedDate` 创建的时间。

  *  `@LastModifiedBy` 最后修改实体的用户。

  *  `@LastModifiedDate` 最后一次修改的时间。

##### 具体实现步骤

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

#### Java Persistence API 里面规定的回调方法

* `@PrePersist`
* `@PostPersist`
* `@PreRemove`
* `@PostRemove`
* `@PreUpdate`
* `@PostUpdate`
* `@PostLoad`

<img src="https://www.chenjunlin.vip/img/spring/jpa/Java%20Persistence%20API.png" alt="img" style="zoom:67%;" />

##### **语法注意事项**

* 回调函数都是和 EntityManager.flush 或 EntityManager.commit 在同一个线程里面执行的，只不过调用方法有先后之分，都是同步调用，所以当任何一个回调方法里面发生异常，都会触发事务进行回滚，而不会触发事务提交。
* Callbacks 注解可以放在实体里面，可以放在 super-class 里面，也可以定义在 entity 的 listener 里面，但需要注意的是：放在实体（或者 super-class）里面的方法，签名格式为“void ()”，即没有参数，方法里面操作的是 this 对象自己；放在实体的 EntityListener 里面的方法签名格式为“void (Object)”，也就是方法可以有参数，参数是代表用来接收回调方法的实体。
* 使上述注解生效的回调方法可以是 public、private、protected、friendly 类型的，但是不能是 static 和 finnal 类型的方法。

##### 示例

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
```

```java
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
    ......
    }
    ```

##### JPA Callbacks 的最佳实践

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

    
