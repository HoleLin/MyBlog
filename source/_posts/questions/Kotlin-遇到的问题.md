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

* [Kotlin Collection VS Kotlin Sequence VS Java Stream](https://www.jianshu.com/p/c5cda7314fe1)

### Kotlin项目中使用SpringBoot Aop切面报错`Caused by: java.lang.IllegalArgumentException: Cannot subclass final class com.holelin.controller.XXXController`

* 环境依赖

  * Gradle

  * Kotlin

  * SpringBoot 2.2.2.RELEASE

    ```groovy
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter
        implementation "org.springframework.boot:spring-boot-starter:$springboot_version"
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web
        implementation "org.springframework.boot:spring-boot-starter-web:$springboot_version"
        // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop
        implementation "org.springframework.boot:spring-boot-starter-aop:$springboot_version"
    ```

  * Spring Kafka 2.4.3.RELEASE

    ```groovy
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

#### 错误信息

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

#### AOP切面不正常的原因

* 需要AOP拦截的类是否是final的，final类不可使用CGLIB来代理。

* 是否在给BEAN配AOP的时候强制使用CGLIB，如果是则可指定proxyTargetClass属性以让spring强制代理目标类。

* 类是否被多次代理了，如果类被多次代理过，则第二次进行代理的时候拿到的是第一次代理后的对象，这个对象是个final形式的，所以会出现这个错误。

#### 当前错误原因

* `KafkaController`为`final`类


#### 解决方法

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

  ```groovy
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

### Kotlin List转Map

```kotlin
    val colors: List<Color> = listOf(
        Color("SILVER", "#C0C0C0"),
        Color("GOLD", "#FFD700"),
        Color("OLIVE", "#808000")
    ) 
    // 1. associate() function
    // Add mapping from name to hex of Color object
    val map: Map<String, String> = colors.associate { Pair(it.name, it.hex) }	
    // or use
    // val map: Map<String, String> = colors.associate { it.name to it.hex }
	  // 2. associateBy() function
    val map: Map<String, String> = colors.associateBy({it.name}, {it.hex})
    // 3. map() function
 		val map: Map<String, String> = colors.map { it.name to it.hex }.toMap()
    // 4. Custom Routine
    val map: MutableMap<String, String> = HashMap()
    for (color in colors) {
        map[color.name] = color.hex
    }
```

### `Kotlin Collection` VS `Kotlin Sequence` VS `Java Stream`

#### 集合中函数式API

* Kolin 的集合分为可变集合(mutable collection)和不可变集合(immutable collection)。不可变集合是 List、Set、Map，它们是只读类型，不能对集合进行修改。可变集合是 MutableList、MutableSet、MutableMap，它们是支持读写的类型，能够对集合进行修改的操作。

##### `filter`

```kotlin
    listOf(5, 12, 8, 33)   // 创建list集合
            .filter { it > 10 }
            .forEach(::println)
```

##### `map`

```kotlin
    listOf("java","kotlin","scala","groovy")
            .map { it.toUpperCase() }
            .forEach(::println)
```

##### `flatMap`

* 遍历所有的元素，为每一个创建一个集合，最后把所有的集合放在一个集合中。

```kotlin
    val newList = listOf(5, 12, 8, 33)
            .flatMap {
                listOf(it, it + 1)
            }

    println(newList)
```

#### Sequence

* 序列(Sequence)是 Kotlin 标准库提供的另一种容器类型。序列与集合有相同的函数 API，却采用不同的实现方式。
* **Kotlin 的 Sequence 更类似于 Java 8 的 Stream，二者都是延迟执行。Kotlin 的集合转换成 Sequence 只需使用`asSequence()`方法。**

```kotlin
  listOf(5, 12, 8, 33)
            .asSequence()
            .filter { it > 10 }
            .forEach(::println)
// <==>            
	sequenceOf(5, 12, 8, 33) // 创建sequence
            .filter { it>10 }
            .forEach (::println)
```

* 使用 Sequence 有助于避免不必要的临时分配开销，并且可以显着提高复杂处理 PipeLines 的性能。

#### Sequence VS Stream

* Sequence 和 Stream 都使用的是惰性求值。

  > 在编程语言理论中，惰性求值（英语：Lazy Evaluation），又译为惰性计算、懒惰求值，也称为传需求调用（call-by-need），是一个计算机编程中的一个概念，目的是要最小化计算机要做的工作。它有两个相关而又有区别的含意，可以表示为“延迟求值”和“最小化求值”。除可以得到性能的提升外，惰性计算的最重要的好处是它可以构造一个无限的数据类型。



| 特性对比      | **Sequence**                                       | Stream                                                       |
| ------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| `autoboxing`  | 会发生自动装箱                                     | 对于原始类型可以避免自动装箱                                 |
| `parallelism` | 不支持                                             | 支持                                                         |
| 跨平台        | 支持 Kotlin/JVM、Kotlin/JS、Kotlin/Native 等多平台 | 只能在 Kotlin/JVM 平台使用，并且 jvm 版本需要>=8             |
| 易用性        | 更简洁、支持更多的功能                             | 使用 Collectors 进行终端操作会使 Stream 更加冗长。           |
| 性能          | 大多数终端操作符是 inline 函数                     | 对于值可能不存在的情况，Sequence 支持可为空的类型，而 Stream 会创建 Optional包装器。因此会多一步的对象创建 |

