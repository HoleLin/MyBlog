---
title: 工具-JMH的使用
date: 2022-02-09 17:42:17
index_img: /img/cover/Tools.jpg
cover: /img/cover/Tools.jpg
tags:
- JMH
- 压测
categories:
- 工具
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

* [别再写 main 方法测试了，太 Low！这才是专业 Java 测试方法！](https://mp.weixin.qq.com/s/3k3rrcVmWa-pjy-ho7r0dA)
* https://hg.openjdk.java.net/code-tools/jmh
* [为什么要用JMH？何时应该用？](https://www.zhihu.com/question/276455629/answer/1259967560)

### JMH简介

* JMH，全称 Java Microbenchmark Harness (微基准测试框架），是专门用于Java代码微基准测试的一套测试工具API，是由 OpenJDK/Oracle 官方发布的工具。

#### 基准测试注意点

* 测试前需要预热。
* 防止无用代码进入测试方法中。
* 并发测试。
* 测试结果呈现。

#### 使用场景

* 定量分析某个热点函数的优化效果
* 想定量地知道某个函数需要执行多长时间，以及执行时间和输入变量的相关性
* 对比一个函数的多种实现方式

#### 依赖

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>${jmh.version}</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>${jmh.version}</version>
    <scope>provided</scope>
</dependency>
```

#### 示例

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
@Fork(value = 2, jvmArgs = {"-Xms4G", "-Xmx4G"})
public class DoublingDemo {
    public int doubleIt(int n) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        return n * 2;
    }

    public static void main(String[] args) throws RunnerException {
        final Options opt = new OptionsBuilder()
                .include(DoublingDemo.class.getSimpleName())
                .result("result.json")
                .resultFormat(ResultFormatType.JSON)
                .build();
        new Runner(opt).run();
    }

    @Benchmark
    public int doubleAndSumSequential() {
        return IntStream.of(3, 1, 4, 1, 5, 9)
                .map(this::doubleIt)
                .sum();
    }

    @Benchmark
    public int doubleAndSumParallel() {
        return IntStream.of(3, 1, 4, 1, 5, 9)
                .parallel()
                .map(this::doubleIt)
                .sum();
    }
}
```

#### 执行测试

* 运行 JMH 基准测试有两种方式，一个是生产jar文件运行，另一个是直接写`main`函数或者放在单元测试中执行.

##### `jar`形式

* 使用`maven-shade-plugin`插件

  ```xml
      <build>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-shade-plugin</artifactId>
                  <version>3.2.4</version>
                  <executions>
                      <execution>
                          <phase>package</phase>
                          <goals>
                              <goal>shade</goal>
                          </goals>
                          <configuration>
                              <finalName>jmh-demo</finalName>
                              <transformers>
                                  <transformer
                                          implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                      <mainClass>org.openjdk.jmh.Main</mainClass>
                                  </transformer>
                              </transformers>
                          </configuration>
                      </execution>
                  </executions>
              </plugin>
          </plugins>
      </build>
  ```

  * SpringBoot项目打包报错`Cannot find 'resource' in class org.apache.maven.plugins.shade.resource.ManifestResourceTransformer`

  * 解决方法,参考[maven-shade-plugin - Cannot find 'resource' in class org.apache.maven.plugins.shade.resource.ManifestResourceTransformer](https://stackoverflow.com/questions/49215416/maven-shade-plugin-cannot-find-resource-in-class-org-apache-maven-plugins-sh)\

    ```xml
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-shade-plugin</artifactId>
                    <version>3.2.4</version>
                    <executions>
                        <execution>
                            <phase>package</phase>
                            <goals>
                                <goal>shade</goal>
                            </goals>
                            <configuration>
                                <finalName>jmh-demo</finalName>
                                <transformers>
                                    <transformer
                                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                        <resource>META-INF/spring.handlers</resource>
                                    </transformer>
                                    <transformer
                                            implementation="org.springframework.boot.maven.PropertiesMergingResourceTransformer">
                                        <resource>META-INF/spring.factories</resource>
                                    </transformer>
                                    <transformer
                                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                        <resource>META-INF/spring.schemas</resource>
                                    </transformer>
                                    <transformer
                                            implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                                    <transformer
                                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                        <mainClass>org.openjdk.jmh.Main</mainClass>
                                    </transformer>
                                </transformers>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    ```

* 执行`mvn clean package`后生成`jar`包

* 执行以下命令:

  ```text
  java -jar jmh-demo.jar  -rf json -rff result.json
  ```

  * `-rf json` 是输出 json的格式
  * `-rff /data/result.json` 是输出文件位置和名称

##### `main`形式

```java
/**
 * 仅限于IDE中运行
 * 命令行模式 则是 build 然后 java -jar 启动
 *
 * 1. 这是benchmark 启动的入口
 * 2. 这里同时还完成了JMH测试的一些配置工作
 * 3. 默认场景下，JMH会去找寻标注了@Benchmark的方法，可以通过include和exclude两个方法来完成包含以及排除的语义
 */
public static void main(String[] args) throws RunnerException {
    Options opt = new OptionsBuilder()
            // 包含语义
            // 可以用方法名，也可以用XXX.class.getSimpleName()
            .include("Helloworld")
            // 排除语义
            .exclude("Pref")
            // 预热10轮
            .warmupIterations(10)
            // 代表正式计量测试做10轮，
            // 而每次都是先执行完预热再执行正式计量，
            // 内容都是调用标注了@Benchmark的代码。
            .measurementIterations(10)
            //  forks(3)指的是做3轮测试，
            // 因为一次测试无法有效的代表结果，
            // 所以通过3轮测试较为全面的测试，
            // 而每一轮都是先预热，再正式计量。
            .forks(3)
         .output("E:/Benchmark.log")
            .build();

    new Runner(opt).run();
}
```

### JMH注解

#### `@BenchmarkMode`

* 用来配置Mode选项,可用于类或方法上,这个注解的value是一个数组,可以把集中Mode集合在一起执行,还可以设置`Mode.ALL`,即全部执行一遍.

| 类型             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `Throughput`     | 整体吞吐量,每秒执行了多少次调用,单位为`opt/time`             |
| `AverageTime`    | 所用平均时间，每次操作的平均耗时,单位为`time/op`             |
| `SampleTime`     | 随机取样,最后输出取样结果的分布                              |
| `SingleShotTime` | 只运行一次,往往同时把`Warmup`次设置为0,用于测试冷启动时的性能 |
| `All`            | 所有模式,上面的所有模式都执行一次                            |

#### `@State`

* 当使用`@Setup`参数的时候，必须在类上加这个参数，不然会提示无法运行。
* 通过 State 可以指定一个对象的作用范围，JMH 根据 scope 来进行实例化和共享操作。`@State` 可以被继承使用，如果父类定义了该注解，子类则无需定义。由于 JMH 允许多线程同时执行测试
* Scope 主要分为三种。
  * `Thread`: 默认的State,该状态为每个线程独享,每个测试线程分配一个实例。
  * `Group`: 同一个组里面所有线程共享。
  * `Benchmark`: 所有测试线程间共享一个实例,测试有状态实例在多线程共享下的性能。

#### `@OutputTimeUnit`

* 基准测试结果的时间单位。一般选择秒、毫秒、微秒。

#### `@Warmup`

* 预热所需要配置的一些基本测试参数，可用于类或者方法上。一般前几次进行程序测试的时候都会比较慢，所以要让程序进行几轮预热，保证测试的准确性。
* 参数含义:
  * `iterations`：预热的次数
  * `time`：每次预热的时间
  * `timeUnit`：时间的单位，默认秒
  * `batchSize`：批处理大小，每次操作调用几次方法

* `@Warmup(iterations = 3)` 表示预热轮数

> **为什么需要预热？**
> 因为 JVM 的 JIT 机制的存在，如果某个函数被调用多次之后，JVM 会尝试将其编译为机器码，从而提高执行速度，所以为了让 benchmark 的结果更加接近真实情况就需要进行预热。

#### `@Measurement`

* 实际调用方法所需要配置的一些基本测试参数，可用于类或者方法上，参数和 `@Warmup` 相同。

#### `@Threads`

* 每个进程中的测试线程

#### `@Fork`

* 进行 fork 的次数。如果 fork 数是3的话，则 JMH 会 fork 出3个进程来进行测试。

#### `@Benchmark`

* 方法级注解，表示该方法是需要进行 benchmark 的对象，用法和 JUnit 的 `@Test` 类似。

#### `@Param`

* 属性级注解，`@Param` 可以用来指定某项参数的多种情况。特别适合用来测试一个函数在不同的参数输入的情况下的性能。

#### `@Setup`

* 方法级注解，这个注解的作用就是我们需要在测试之前进行一些**准备工作** ，比如对一些数据的初始化之类的。

#### `@TearDown`

* 方法级注解，这个注解的作用就是我们需要在测试之后进行一些**结束工作** ，比如关闭线程池，数据库连接等的，主要用于资源的回收等.

### JMH IDEA 插件

![img](https://www.holelin.cn/img/tools/jmh/JMH%20Plugin.png)

### JMH 可视化网站

* http://deepoove.com/jmh-visual-chart/
* https://jmh.morethan.io/

