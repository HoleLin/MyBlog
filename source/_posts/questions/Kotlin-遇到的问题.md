---
title: Kotlin-遇到的问题
date: 2021-08-13 17:14:00
index_img: /img/cover/Kotlin.jpeg
cover: /img/cover/Kotlin.jpeg
tags:
- questions
categories:
- Kotlin
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

#### Kotlin项目中使用SpringBoot Aop切面报错`Caused by: java.lang.IllegalArgumentException: Cannot subclass final class com.holelin.controller.XXXController`

* 环境依赖

  * Gradle

  * Kotlin

  * SpringBoot 2.2.2.RELEASE

    ```
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter
        implementation "org.springframework.boot:spring-boot-starter:$springboot_version"
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web
        implementation "org.springframework.boot:spring-boot-starter-web:$springboot_version"
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop
        implementation "org.springframework.boot:spring-boot-starter-aop:$springboot_version"
    ```

  * Spring Kafka 2.4.3.RELEASE

    ```
      // https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka
        implementation 'org.springframework.kafka:spring-kafka:2.4.3.RELEASE'
    ```

* 注解

  ```kotlin
  @Retention(AnnotationRetention.RUNTIME)
  @Target(AnnotationTarget.FUNCTION,AnnotationTarget.TYPE)
  annotation class LoginRequired{
  }
  ```

* 切面类

  ```kotlin
  import org.aspectj.lang.annotation.Aspect
  import org.aspectj.lang.annotation.Before
  import org.aspectj.lang.annotation.Pointcut
  import org.springframework.stereotype.Component
  import javax.servlet.http.HttpServletRequest
  
  @Aspect
  @Component
  open class LoginRequiredInterceptor {
  
      @Pointcut(value = "@annotation(com.holelin.annotations.LoginRequired))")
      fun pointCut(){
      }
  
      @Before("pointCut()")
      fun before(){
          println("before ..")
      }
  }
  ```

* 注解使用类

  ```kotlin
  import com.holelin.annotations.LoginRequired
  import com.holelin.mq.kafka.KafkaProducer
  import org.springframework.beans.factory.annotation.Autowired
  import org.springframework.web.bind.annotation.PostMapping
  import org.springframework.web.bind.annotation.RequestMapping
  import org.springframework.web.bind.annotation.RestController
  
  @RequestMapping("/kafka")
  @RestController
   class KafkaController {
      @Autowired
      private lateinit var kafkaProducer: KafkaProducer
  
      @LoginRequired
      @PostMapping("/send")
      fun sendMessage() {
          kafkaProducer.sendMessage()
      }
  }
  ```

  * 辅助类

    ```kotlin
    import cn.hutool.core.date.DateTime
    import cn.hutool.json.JSONUtil
    import com.github.javafaker.Faker
    import com.holelin.domain.AccessRecord
    import org.springframework.beans.factory.annotation.Autowired
    import org.springframework.kafka.core.KafkaTemplate
    import org.springframework.stereotype.Component
    
    @Component
    class KafkaProducer {
    
        @Autowired
        private lateinit var kafkaTemplate: KafkaTemplate<String, String>
    
        private val testTopic: String = "UserInfo"
        
        fun sendMessage() {
            var accessRecord = AccessRecord()
            var faker = Faker()
            println(faker.address().toString())
            println(faker.artist().toString())
            accessRecord.apply {
                this.ip = "192.168.0.1"
                this.accessTime = DateTime.now()
                this.param = JSONUtil.parseObj(faker.ancient()).toString()
                this.url = "/kafka/send"
                this.requestMethod = "POST"
                this.username = "holelin"
                this.response = JSONUtil.parseObj(faker.address()).toString()
                this.accessServiceName = "kotlin-demo"
            }
            var send = kafkaTemplate.send(testTopic, JSONUtil.toJsonStr(accessRecord))
            send.addCallback({ result -> println(result) }, { ex -> println(ex) })
        }
    }
    ```

##### 错误信息

```java
Caused by: org.springframework.aop.framework.AopConfigException: Could not generate CGLIB subclass of class com.holelin.controller.KafkaController: Common causes of this problem include using a final class or a non-visible class; nested exception is java.lang.IllegalArgumentException: Cannot subclass final class com.holelin.controller.KafkaController
	at org.springframework.aop.framework.CglibAopProxy.getProxy(CglibAopProxy.java:208) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.ProxyFactory.getProxy(ProxyFactory.java:110) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.createProxy(AbstractAutoProxyCreator.java:471) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.wrapIfNecessary(AbstractAutoProxyCreator.java:350) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.postProcessAfterInitialization(AbstractAutoProxyCreator.java:299) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization(AbstractAutowireCapableBeanFactory.java:431) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1800) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:595) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	... 16 common frames omitted
Caused by: java.lang.IllegalArgumentException: Cannot subclass final class com.holelin.controller.KafkaController
	at org.springframework.cglib.proxy.Enhancer.generateClass(Enhancer.java:657) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.DefaultGeneratorStrategy.generate(DefaultGeneratorStrategy.java:25) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.ClassLoaderAwareGeneratorStrategy.generate(ClassLoaderAwareGeneratorStrategy.java:57) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.java:358) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.proxy.Enhancer.generate(Enhancer.java:582) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData$3.apply(AbstractClassGenerator.java:110) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData$3.apply(AbstractClassGenerator.java:108) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.internal.LoadingCache$2.call(LoadingCache.java:54) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at java.base/java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:264) ~[na:na]
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java) ~[na:na]
	at org.springframework.cglib.core.internal.LoadingCache.createEntry(LoadingCache.java:61) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.internal.LoadingCache.get(LoadingCache.java:34) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData.get(AbstractClassGenerator.java:134) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:319) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.proxy.Enhancer.createHelper(Enhancer.java:569) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.cglib.proxy.Enhancer.createClass(Enhancer.java:416) ~[spring-core-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.ObjenesisCglibAopProxy.createProxyClassAndInstance(ObjenesisCglibAopProxy.java:57) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy.getProxy(CglibAopProxy.java:205) ~[spring-aop-5.2.4.RELEASE.jar:5.2.4.RELEASE]
	... 23 common frames omitted
```

##### AOP切面不正常的原因

* 需要AOP拦截的类是否是final的，final类不可使用CGLIB来代理。

* 是否在给BEAN配AOP的时候强制使用CGLIB，如果是则可指定proxyTargetClass属性以让spring强制代理目标类。

* 类是否被多次代理了，如果类被多次代理过，则第二次进行代理的时候拿到的是第一次代理后的对象，这个对象是个final形式的，所以会出现这个错误。

##### 当前错误原因

* `KafkaController`为`final`类


##### 解决方法

* 给`KafkaController`类前加`open`关键字

  ```kotlin
  @RequestMapping("/kafka")
  @RestController
  open class KafkaController {
      @Autowired
      private lateinit var kafkaProducer: KafkaProducer
  
      @LoginRequired
      @PostMapping("/send")
      fun sendMessage() {
          kafkaProducer.sendMessage()
      }
  }
  ```

* 此时出现第二个问题自动注入的`KafkaProducer`为`null`

  ![img](https://www.holelin.cn/img/kotlin/Kotlin-SpringBoot-AOP.png)

  

* 处理办法使用Kotlin官方插件：[All-open compiler plugin](https://kotlinlang.org/docs/all-open-plugin.html)

  ```
  buildscript {
      ext{
          dcm4che_version = '5.21.0'
          springboot_version= '2.2.2.RELEASE'
          hutool_version="5.7.7"
      }
      dependencies {
          classpath "org.jetbrains.kotlin:kotlin-allopen:1.5.21"
      }
  }
  
  plugins {
      id 'org.jetbrains.kotlin.jvm' version '1.4.32'
      id "org.jetbrains.kotlin.plugin.spring" version "1.5.21"
  
  }
  
  group 'org.example'
  version '1.0-SNAPSHOT'
  apply plugin: "kotlin-spring"
  
  repositories {
      maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
      maven { url 'https://raw.github.com/nroduit/mvn-repo/master/'}
      maven { url 'https://www.dcm4che.org/maven2/'}
      mavenCentral()
  }
  
  dependencies {
      implementation "org.jetbrains.kotlin:kotlin-stdlib"
  
      //SpringBoot
      // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter
      implementation "org.springframework.boot:spring-boot-starter:$springboot_version"
      // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web
      implementation "org.springframework.boot:spring-boot-starter-web:$springboot_version"
      // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-configuration-processor
      implementation "org.springframework.boot:spring-boot-configuration-processor:2.2.2.RELEASE"
      // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator
      implementation "org.springframework.boot:spring-boot-starter-actuator:$springboot_version"
  
      // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop
      implementation "org.springframework.boot:spring-boot-starter-aop:$springboot_version"
  
      // https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka
      implementation 'org.springframework.kafka:spring-kafka:2.4.3.RELEASE'
  }
  
  compileKotlin {
      kotlinOptions.jvmTarget = "11"
  }
  compileTestKotlin {
      kotlinOptions.jvmTarget = "11"
  }
  
  ```

  ![img](https://www.holelin.cn/img/kotlin/Kotlin-SpringBoot-AOP2.png)



