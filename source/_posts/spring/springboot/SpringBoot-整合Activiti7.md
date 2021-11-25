---
title: SpringBoot-整合Activiti7
date: 2021-11-24 13:29:55
index_img: /img/cover/Activiti.png
cover: /img/cover/Activiti.png
tags:
- Activiti7
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

* [Acitvit6 User Guide](https://www.activiti.org/userguide/)

### 环境配置

* #### 框架版本 

  * SpringBoot 2.5.7
  * SpringCloud 2020.0.3
  * activiti 7.1.0.M6
  * flyway  8.0.5
  * mysql 8.0.27

* #### 依赖信息

  * `build.gradle`配置(**修改后的**)

    ```
    buildscript {
        ext {
            springBootVersion = '2.5.7'
            springCloudVersion = '2020.0.3'
            activitiVersion = "7.1.0.M6"
            hutoolVersion = "5.7.16"
            flywayVersion = "8.0.5"
            mysqlVerssion = "8.0.27"
        }
        dependencies {
            classpath "io.spring.gradle:dependency-management-plugin:1.0.10.RELEASE"
        }
    }
    
    
    plugins {
        id "org.jetbrains.kotlin.jvm" version "1.4.31"
        id "application"
    }
    
    group "com.xxxx.xxxx"
    version "1.0-SNAPSHOT"
    
    configurations {
        ktlint
        runtime.exclude group: "org.slf4j", module: "slf4j-log4j12"
    }
    
    repositories {
        maven { url "https://maven.aliyun.com/nexus/content/groups/public/" }
    
        mavenCentral()
    }
    
    dependencies {
        implementation platform("org.jetbrains.kotlin:kotlin-bom")
        implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
        implementation "com.google.guava:guava:30.1-jre"
        implementation "org.jetbrains.kotlin:kotlin-reflect"
    
        // SpringBoot
        implementation("org.springframework.boot:spring-boot-starter:$springBootVersion")
        implementation("org.springframework.boot:spring-boot-starter-web:$springBootVersion") {
            exclude group: "org.springframework.boot", module: "spring-boot-starter-tomcat"
        }
        implementation("org.springframework.boot:spring-boot-starter-jetty:$springBootVersion")
        implementation("org.springframework.boot:spring-boot-configuration-processor:$springBootVersion")
        implementation("org.springframework.boot:spring-boot-actuator-autoconfigure:$springBootVersion")
        // 数据库相关
        implementation("org.springframework.boot:spring-boot-starter-jdbc:$springBootVersion")
        implementation("org.springframework.boot:spring-boot-starter-data-jpa:$springBootVersion")
        implementation "mysql:mysql-connector-java:$mysqlVerssion"
    
        // 数据库脚本管理框架
        implementation("org.flywaydb:flyway-core:$flywayVersion")
    
        // MQ
        implementation("org.springframework.boot:spring-boot-starter-amqp:$springBootVersion")
    
        // Redis
        implementation("org.springframework.boot:spring-boot-starter-data-redis:$springBootVersion")
    
        // SpringCloud
        compile("org.springframework.cloud:spring-cloud-starter-config")
        compile("org.springframework.cloud:spring-cloud-starter-consul-discovery")
        compile("org.springframework.cloud:spring-cloud-starter-openfeign")
        implementation("org.springframework.cloud:spring-cloud-starter-bootstrap:3.0.4")
    
        // 工作流引擎starter
        implementation("org.activiti:activiti-spring-boot-starter:$activitiVersion")
    
        // 工具类
        implementation("cn.hutool:hutool-core:$hutoolVersion")
    
        implementation("com.fasterxml.jackson.module:jackson-module-kotlin:2.13.0")
        testImplementation "org.jetbrains.kotlin:kotlin-test"
        testImplementation "org.jetbrains.kotlin:kotlin-test-junit"
    }
    
    apply plugin: "io.spring.dependency-management"
    
    dependencyManagement {
        imports {
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
        }
    }
    application {
        mainClass = "com.dianeiai.hemnes.AppKt"
    }
    
    
    compileKotlin {
        kotlinOptions.jvmTarget = "11"
    }
    compileTestKotlin {
        kotlinOptions.jvmTarget = "11"
    }
    ```

  * `maven`配置

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.holelin</groupId>
            <artifactId>business-automation</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </parent>
        <groupId>cn.holelin</groupId>
        <artifactId>activiti</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <name>activiti</name>
        <description>activiti</description>
        <properties>
            <java.version>11</java.version>
            <activiti.version>7.1.0.M6</activiti.version>
            <maven.compiler.source>11</maven.compiler.source>
            <maven.compiler.target>11</maven.compiler.target>
        </properties>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
    
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
            <dependency>
                <groupId>org.activiti</groupId>
                <artifactId>activiti-spring-boot-starter</artifactId>
                <version>${activiti.version}</version>
            </dependency>
    
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-jpa</artifactId>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    
    </project>
    ```

### 遇到的问题

#### Table 'xxx.act_ge_property' doesn't exist

```java
Caused by: org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Table 'hemnes.act_ge_property' doesn't exist
### The error may exist in org/activiti/db/mapping/entity/Property.xml
### The error may involve org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty-Inline
### The error occurred while setting parameters
### SQL: select * from ACT_GE_PROPERTY where NAME_ = ?
### Cause: java.sql.SQLSyntaxErrorException: Table 'xxx.act_ge_property' doesn't exist
	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:150)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:77)
	at org.activiti.engine.impl.db.DbSqlSession.selectById(DbSqlSession.java:461)
```

* 解决方法: 在`spring.datasource.url`补充`nullCatalogMeansCurrent=true`

  ```yaml
  spring:
    datasource:
      url: jdbc:mysql://localhost:3306/hemnes?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
  ```

#### Consider revisiting the entries above or defining a bean of type ‘javax.sql.DataSource’ in your configuration

```tcl
Description:

Parameter 0 of method springProcessEngineConfiguration in org.activiti.spring.boot.ProcessEngineAutoConfiguration required a bean of type 'javax.sql.DataSource' that could not be found.

The following candidates were found but could not be injected:
- Bean method 'dataSource' in 'JndiDataSourceAutoConfiguration' not loaded because @ConditionalOnProperty (spring.datasource.jndi-name) did not find property 'jndi-name'
- Bean method 'dataSource' in 'XADataSourceAutoConfiguration' not loaded because @ConditionalOnClass did not find required class 'javax.transaction.TransactionManager'

Action:

Consider revisiting the entries above or defining a bean of type 'javax.sql.DataSource' in your configuration.
```

* 原因: 缺少默认DataSource数据源

* 解决方法:

  * 添加`JPA`/`MyBatis`等数据库的`starter`

    ```
    implementation("org.springframework.boot:spring-boot-starter-data-jpa:$springBootVersion")
    ```

#### No spring.config.import property has been defined

```
No spring.config.import property has been defined

Action:

Add a spring.config.import=configserver: property to your configuration. If configuration is not required add spring.config.import=optional:configserver: instead. To disable this check, set spring.cloud.config.enabled=false or spring.cloud.config.import-check.enabled=false.
```

* 解决方法

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bootstrap</artifactId>
  </dependency>
  ```

  ```
  implementation("org.springframework.cloud:spring-cloud-starter-bootstrap:3.0.4")
  ```

### Activiti使用注意点

* 在使用`UserTask`的`TaskListener`来对`UserTask`任务进行监听时,若要使用Spring的自动注入功能,需要在配置监听器的时候使用`delegateExpression`来指定,如下所示:

  ```xml
          <bpmn:userTask id="xxxxx" name="xxxx">
              <bpmn:extensionElements>
                  <activiti:taskListener event="create" delegateExpression="${createListener}"/>
                  <activiti:taskListener event="complete"  delegateExpression="${completeListener}"/>
              </bpmn:extensionElements>
              <bpmn:incoming>Flow_1chtlxm</bpmn:incoming>
              <bpmn:outgoing>Flow_0n355t9</bpmn:outgoing>
          </bpmn:userTask>
  ```

  

