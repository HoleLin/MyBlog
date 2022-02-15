---
title: SpringBoot-整合SpringDataJPA+MyBatisPlus多数据源配置
date: 2022-01-29 13:55:57
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- integration
- 多数据源
categories:
- SpringBoot
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

* [Springboot整合mybatis实现多数据源所遇到的问题](https://www.codenong.com/cs109706915/)
* [springboot + mybatis-plus 分包实现多数据源配置](https://houbb.github.io/2020/06/21/hand-write-orm-09-mybatis-multi-datasource-package)
* [required a bean of type 'org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder' that could not be found](https://stackoverflow.com/questions/67966956/required-a-bean-of-type-org-springframework-boot-orm-jpa-entitymanagerfactorybu)

### 背景

* 公司项目最初数据库访问层使用的是`Spring Data JPA`后期由于内部原因改换使用`MyBatis Plus`,由于业务需要对接第三方数据需要直接访问第三方的数据库,故而要配置`MyBatis Plus`多数据源.

### 环境以及项目技术栈

```
SpringBoot 2.5.6
MyBatis Plus  3.5.1
```

### 公共代码

#### Entity

```java
package com.holelin.mysql.entity;

import lombok.Data;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Data
@Entity
// 此处故意写成驼峰形式 应该为multiple_data_sources
@Table(name = "MultipleDataSources")
public class MultipleDataSourcesEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "number")
    private Integer number;

    //
    @Column
    private String multipleDataSources;
}
```

#### Service

```java
package com.holelin.mysql.service.impl;

import com.holelin.mysql.entity.MultipleDataSourcesEntity;
import com.holelin.mysql.jpa.dao.MultipleDataSourcesEntityDao;
import com.holelin.mysql.mybatis.mapper.MultipleDataSourcesEntityMapper;
import com.holelin.mysql.service.MultipleDataSourceService;
import com.holelin.mysql.vo.JpaRequest;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class MultipleDataSourceServiceImpl implements MultipleDataSourceService {

    @Autowired
    private MultipleDataSourcesEntityDao multipleDataSourcesEntityDao;
    @Autowired
    private MultipleDataSourcesEntityMapper multipleDataSourcesEntityMapper;

    @Override
    public List<MultipleDataSourcesEntity> queryFromJpa() {
        return multipleDataSourcesEntityDao.findAll();
    }

    @Override
    public void addJpa(JpaRequest request) {
        MultipleDataSourcesEntity entity = new MultipleDataSourcesEntity();
        BeanUtils.copyProperties(request, entity);
        multipleDataSourcesEntityDao.save(entity);
    }

    @Override
    public List<MultipleDataSourcesEntity> queryFromMyBatis() {
        return multipleDataSourcesEntityMapper.query();
    }

    @Override
    public List<MultipleDataSourcesEntity> queryFromMyBatisOtherSource() {

        return null;
    }
}
```

#### Controller

```java
package com.holelin.mysql.controller;

import com.holelin.mysql.entity.MultipleDataSourcesEntity;
import com.holelin.mysql.service.MultipleDataSourceService;
import com.holelin.mysql.vo.JpaRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * @Description:
 * @Author: HoleLin
 * @CreateDate: 2020/7/30 10:21
 * @UpdateUser: HoleLin
 * @UpdateDate: 2020/7/30 10:21
 * @UpdateRemark: 修改内容
 * @Version: 1.0
 */

@RestController
@RequestMapping("/multiple-data-source")
public class MultipleDataSourceController {

    @Autowired
    private MultipleDataSourceService multipleDataSourceService;

    @GetMapping("/jpa")
    public List<MultipleDataSourcesEntity> testJpa() {
        return multipleDataSourceService.queryFromJpa();
    }
    @PostMapping("/jpa")
    public void addJpaData(@RequestBody JpaRequest request) {
        multipleDataSourceService.addJpa(request);
    }

    @GetMapping("/mybatis")
    public List<MultipleDataSourcesEntity> testMybatis() {
        return multipleDataSourceService.queryFromMyBatis();
    }

    @PostMapping("/mybatis")
    public List<MultipleDataSourcesEntity> addMybatis() {
        return multipleDataSourceService.queryFromMyBatis();
    }

}
```

#### 启动类

```java
package com.holelin.mysql;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@EnableAsync
@EnableTransactionManagement
//@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class})
@SpringBootApplication
public class MysqlApplication {

    public static void main(String[] args) {
        SpringApplication.run(MysqlApplication.class, args);
    }

}
```

#### application配置

```java
server:
  port: 8092

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test_data?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=false
    username: root
    password: holelin..
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: update
    show-sql: true
```

### 问题: 手动配置MyBatis数据源,JPA插入数据时,无法入库

* MyBatis配置

  ```java
  package com.holelin.mysql.mybatis.config;
  
  import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
  import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
  import org.apache.ibatis.plugin.Interceptor;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.mybatis.spring.SqlSessionTemplate;
  import org.mybatis.spring.annotation.MapperScan;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.boot.jdbc.DataSourceBuilder;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Primary;
  import org.springframework.core.io.Resource;
  import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
  import org.springframework.jdbc.datasource.DataSourceTransactionManager;
  
  import javax.sql.DataSource;
  
  /**
   * @Description: MyBaits 配置类
   * @Author: HoleLin
   * @CreateDate: 2022/1/29 2:22 PM
   * @UpdateUser: HoleLin
   * @UpdateDate: 2022/1/29 2:22 PM
   * @UpdateRemark: 修改内容
   * @Version: 1.0
   */
  @Configuration
  @MapperScan(basePackages = {"com.holelin.mysql.mybatis.mapper"},
          sqlSessionFactoryRef = "mybatisSqlSessionFactory")
  public class MyBatisPlusConfig {
  
      @Value("${spring.datasource.url}")
      private String url;
      @Value("${spring.datasource.username}")
      private String username;
      @Value("${spring.datasource.password}")
      private String password;
      @Value("${spring.datasource.driver-class-name}")
      private String driverClassName;
  
      @Primary
      @Bean(name = "dataSource")
      public DataSource dataSource() {
          final DataSource dataSource = DataSourceBuilder.create().
                  url(url).
                  username(username).
                  password(password).
                  driverClassName(driverClassName).
                  build();
          return dataSource;
      }
  
      /**
       * 配置  SqlSessionFactory TransactionManager SqlSessionTemplate
       */
      @Primary
      @Bean(name = "mybatisSqlSessionFactory")
      public SqlSessionFactory mybatisSqlSessionFactory(@Qualifier("dataSource") DataSource dataSource) throws Exception {
          final MybatisSqlSessionFactoryBean mybatisSqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
          mybatisSqlSessionFactoryBean.setDataSource(dataSource);
          // 指定XML配置路径
          // 注 此处需要使用 getResources()
          // getResources() 获取所有类路径下的指定文件
          // getResource()  表示从当前类的根路径去查找资源，能获取到同一个包下的文件
          final Resource[] resources = new PathMatchingResourcePatternResolver().getResources("classpath*:/mapping/**/*.xml");
          mybatisSqlSessionFactoryBean.setMapperLocations(resources);
  
          // 配置分页插件
          Interceptor[] plugins = new Interceptor[]{mybatisPlusInterceptor()};
          mybatisSqlSessionFactoryBean.setPlugins(plugins);
  
          return mybatisSqlSessionFactoryBean.getObject();
      }
  
      @Primary
      @Bean(name = "transactionManager")
      public DataSourceTransactionManager transactionManager(@Qualifier("dataSource") DataSource dataSource) {
          return new DataSourceTransactionManager(dataSource);
      }
  
      @Primary
      @Bean(name = "sqlSessionTemplate")
      public SqlSessionTemplate sqlSessionTemplate(@Qualifier("mybatisSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
          return new SqlSessionTemplate(sqlSessionFactory);
      }
  
      @Bean("mybatisPlusInterceptor")
      public MybatisPlusInterceptor mybatisPlusInterceptor() {
          final MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
          interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
          return interceptor;
      }
  }
  ```

* 现象为控制台未打印出插入日志(配置文件中`spring.jpa.show-sql: true`),按道理插入成功的话应该会打印插入日志;推测可能是手动配置了`MyBatis Plus`数据源配置,导致`JPA`失效了.既然JPA失效了,那也手动配置一下`JPA`的数据源

* 处理方式: 补充`JPA`数据源手动配置

  ```java
  package com.holelin.mysql.jpa.config;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
  import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
  import org.springframework.orm.jpa.JpaTransactionManager;
  import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
  import org.springframework.transaction.PlatformTransactionManager;
  import org.springframework.transaction.annotation.EnableTransactionManagement;
  
  import javax.persistence.EntityManager;
  import javax.persistence.EntityManagerFactory;
  import javax.sql.DataSource;
  
  @Configuration
  @EnableTransactionManagement
  @EnableJpaRepositories(
          entityManagerFactoryRef = "jpaEntityManagerFactory",
          transactionManagerRef = "jpaTransactionManager",
          basePackages = {"com.holelin.mysql.jpa.dao"}
  )
  public class JpaConfig {
  
  
      @Autowired
      private JpaProperties jpaProperties;
  
      @Bean(name = "jpaEntityManagerFactory")
      public LocalContainerEntityManagerFactoryBean entityManagerFactory(
              EntityManagerFactoryBuilder builder,
              @Qualifier("dataSource") DataSource dataSource) {
          return builder.dataSource(dataSource).
                  properties(jpaProperties.getProperties()).
                  packages("com.holelin.mysql.entity").
                  build();
      }
  
  
      @Bean(name = "jpaEntityManager")
      public EntityManager entityManager(EntityManagerFactoryBuilder builder,
                                         @Qualifier("dataSource") DataSource dataSource) {
          final EntityManagerFactory managerFactory = entityManagerFactory(builder, dataSource).getObject();
          return managerFactory.createEntityManager();
      }
  
      @Bean(name = "jpaTransactionManager")
      public PlatformTransactionManager transactionManager(
              EntityManagerFactoryBuilder builder,
              @Qualifier("dataSource") DataSource dataSource) {
          final EntityManagerFactory managerFactory = entityManagerFactory(builder, dataSource).getObject();
          return new JpaTransactionManager(managerFactory);
  
      }
  }
  ```

* 配置完成后,调用`POST /multiple-data-source/jpa`接口,会报出`Table 'test_data.multipledatasources' doesn't exist`异常信息

* 分析:

  * 异常信息显示`multipledatasources`这个表不存在,但是数据库中创建的表名为`multiple_data_sources`,将`Entity`类中的`@Table(name = "MultipleDataSources")`修改为`@Table(name = "multiple_data_sources")`后重新运行程序,报出`Unknown column 'multipleDataSources' in 'field list'`异常

    ![img](https://www.holelin.cn/img/questions/%E6%95%B4%E5%90%88%E6%95%B0%E6%8D%AE%E6%BA%90%E9%97%AE%E9%A2%98%E7%B4%A0%E6%9D%902.png)

  * 进一步推测应该是`JPA`中的[命名策略(`ImplicitNamingStrategy`)](https://www.holelin.cn/2021/08/24/spring/jpa/JPA(%E5%9B%9B)-%E6%95%B0%E6%8D%AE%E6%BA%90/#Naming%20%E5%91%BD%E5%90%8D%E7%AD%96%E7%95%A5%E8%AF%A6%E8%A7%A3%E5%8F%8A%E5%85%B6%E5%AE%9E%E8%B7%B5)出现了问题;

  * 为了证明推测,添加断点进行调试.经过调试后发现程序的命令策略确实出现了问题

    * 按道理应该走`SpringImplicitNamingStrategy`实现类,但是程序实际走的是`PhysicalNamingStrategy`默认实现.

    ```java
    org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl
    -->
    org.hibernate.boot.MetadataSources#getMetadataBuilder(org.hibernate.boot.registry.StandardServiceRegistry)
    --> 
    org.hibernate.boot.internal.MetadataBuilderImpl
    --> 
    org.hibernate.boot.internal.MetadataBuilderImpl.MetadataBuildingOptionsImpl
    -->
    org.hibernate.boot.registry.selector.spi.StrategySelector#resolveDefaultableStrategy(java.lang.Class<T>, java.lang.Object, java.util.concurrent.Callable<T>)
    ```

  * debug发现可以通过配置`hibernate.implicit_naming_strategy`和`hibernate.physical_naming_strategy`进行修改

  * 修改后的`application.yml`文件内容

    ```yaml
    server:
      port: 8092
    
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/test_data?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=false
        username: root
        password: holelin..
        driver-class-name: com.mysql.cj.jdbc.Driver
      jpa:
        open-in-view: false
        hibernate:
          ddl-auto: update
        # 补充下面的配置项    
        properties:
          hibernate.implicit_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
          hibernate.physical_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
        show-sql: true
    
    ```

* 补充两个配置后,重新启动程序并重新调用创建数据接口,在控制台中出现插入语句日志

  ![img](https://www.holelin.cn/img/questions/%E6%95%B4%E5%90%88%E6%95%B0%E6%8D%AE%E6%BA%90%E9%97%AE%E9%A2%98%E7%B4%A0%E6%9D%901.png)

* 总结与建议

  * 在使用`JPA`建议在`@Table`注解中将数据表的名称书写规范,虽然Spring提供的命名策略来做处理,但是保不齐会出现上诉问题,以及在`Entity`中字段建议都加上`@Column`并在`name `中指明在数据表中实际的值来避免意料之外的问题;
  
### 配置MyBaits Plus多数据源

* 配置多数据源比较简单,可以复制第一份MyBaits配置进行修改

  ```java
  package com.holelin.mysql.mybatis.second.config;
  
  import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
  import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
  import org.apache.ibatis.plugin.Interceptor;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.mybatis.spring.SqlSessionTemplate;
  import org.mybatis.spring.annotation.MapperScan;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.boot.jdbc.DataSourceBuilder;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Primary;
  import org.springframework.core.io.Resource;
  import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
  import org.springframework.jdbc.datasource.DataSourceTransactionManager;
  
  import javax.sql.DataSource;
  
  /**
   * @Description: MyBaits 配置类
   * @Author: HoleLin
   * @CreateDate: 2022/1/29 2:22 PM
   * @UpdateUser: HoleLin
   * @UpdateDate: 2022/1/29 2:22 PM
   * @UpdateRemark: 修改内容
   * @Version: 1.0
   */
  @Configuration
  @MapperScan(basePackages = {"com.holelin.mysql.mybatis.second.mapper"},
          sqlSessionFactoryRef = "secondSqlSessionFactory")
  public class MyBatisPlusSecondConfig {
  
      @Value("${third-party.datasource.url}")
      private String url;
      @Value("${third-party.datasource.username}")
      private String username;
      @Value("${third-party.datasource.password}")
      private String password;
      @Value("${third-party.datasource.driver-class-name}")
      private String driverClassName;
  
      @Primary
      @Bean(name = "secondDataSource")
      public DataSource secondDataSource() {
          final DataSource dataSource = DataSourceBuilder.create().
                  url(url).
                  username(username).
                  password(password).
                  driverClassName(driverClassName).
                  build();
          return dataSource;
      }
  
      /**
       * 配置  SqlSessionFactory TransactionManager SqlSessionTemplate
       */
      @Primary
      @Bean(name = "secondSqlSessionFactory")
      public SqlSessionFactory secondSqlSessionFactory(@Qualifier("secondDataSource") DataSource dataSource) throws Exception {
          final MybatisSqlSessionFactoryBean secondSqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
          secondSqlSessionFactoryBean.setDataSource(dataSource);
          // 指定XML配置路径
          // 注 此处需要使用 getResources()
          // getResources() 获取所有类路径下的指定文件
          // getResource()  表示从当前类的根路径去查找资源，能获取到同一个包下的文件
          final Resource[] resources = new PathMatchingResourcePatternResolver().getResources("classpath*:/mapping/second/*.xml");
          secondSqlSessionFactoryBean.setMapperLocations(resources);
  
          // 配置分页插件
          Interceptor[] plugins = new Interceptor[]{secondPlusInterceptor()};
          secondSqlSessionFactoryBean.setPlugins(plugins);
  
          return secondSqlSessionFactoryBean.getObject();
      }
  
      @Primary
      @Bean(name = "secondTransactionManager")
      public DataSourceTransactionManager secondTransactionManager(@Qualifier("secondDataSource") DataSource dataSource) {
          return new DataSourceTransactionManager(dataSource);
      }
  
      @Primary
      @Bean(name = "secondSqlSessionTemplate")
      public SqlSessionTemplate secondSqlSessionTemplate(@Qualifier("secondSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
          return new SqlSessionTemplate(sqlSessionFactory);
      }
  
      @Bean("secondPlusInterceptor")
      public MybatisPlusInterceptor secondPlusInterceptor() {
          return new MybatisPlusInterceptor();
      }
  
  }
  
  ```

* 若出现下面异常,则需要在JpaConfg中补充

  ```
  ***************************
  APPLICATION FAILED TO START
  ***************************
  
  Description:
  
  Parameter 0 of method entityManagerFactory in com.holelin.mysql.jpa.config.JpaConfig required a bean of type 'org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder' that could not be found.
  
  
  Action:
  
  Consider defining a bean of type 'org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder' in your configuration.
  ```

  ```java
      @Bean
      public EntityManagerFactoryBuilder entityManagerFactoryBuilder() {
          return new EntityManagerFactoryBuilder(new HibernateJpaVendorAdapter(), new HashMap<>(), null);
      }
  ```

* 启动类排除`DataSourceAutoConfiguration`以及`DataSourceTransactionManagerAutoConfiguration`

  ```java
  package com.holelin.mysql;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
  import org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration;
  import org.springframework.scheduling.annotation.EnableAsync;
  import org.springframework.transaction.annotation.EnableTransactionManagement;
  
  @EnableAsync
  @EnableTransactionManagement
  @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class})
  //@SpringBootApplication
  public class MysqlApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(MysqlApplication.class, args);
      }
  
  }
  
  ```

* 特别注意点: **在配置多数据源的时候`@MapperScan`中的`sqlSessionFactoryRef`一定要写并进行区分,不然会出现无法绑定*Mapping.xml中自己写的方法;**

  

​    
