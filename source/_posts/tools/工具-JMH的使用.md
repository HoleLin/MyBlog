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

#### JMH简介

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
@State(Scope.Benchmark)
@OutputTimeUnit(TimeUnit.SECONDS)
@Threads(Threads.MAX)
public class LinkedListIterationBenchMark {
 private static final int SIZE = 10000;

    private List<String> list = new LinkedList<>();

    @Setup
    public void setUp() {
        for (int i = 0; i < SIZE; i++) {
            list.add(String.valueOf(i));
        }
    }

    @Benchmark
    @BenchmarkMode(Mode.Throughput)
    public void forIndexIterate() {
        for (int i = 0; i < list.size(); i++) {
            list.get(i);
            System.out.print("");
        }
    }

    @Benchmark
    @BenchmarkMode(Mode.Throughput)
    public void forEachIterate() {
        for (String s : list) {
            System.out.print("");
        }
    }
}
```

#### 执行测试

* 运行 JMH 基准测试有两种方式，一个是生产jar文件运行，另一个是直接写main函数或者放在单元测试中执行.

### JMH注解

#### `@BenchmarkMode`

| 类型           | 描述                             |
| -------------- | -------------------------------- |
| Throughput     | 每段时间执行的次数，一般是秒     |
| AverageTime    | 平均时间，每次操作的平均耗时     |
| SampleTime     | 在测试中，随机进行采样执行的时间 |
| SingleShotTime | 在每次执行中计算耗时             |
| All            | 所有模式                         |

#### `@Warmup`

* `@Warmup(iterations = 3)` 表示预热轮数

#### `@Measurement`

* 正式度量计算的轮数。

| 类型       | 描述           |
| ---------- | -------------- |
| iterations | 进行测试的轮次 |
| time       | 每轮进行的时长 |
| timeUnit   | 时长单位       |

#### `@Threads`

* 每个进程中的测试线程

#### `@Fork`

* 进行 fork 的次数。如果 fork 数是3的话，则 JMH 会 fork 出3个进程来进行测试。

#### `@OutputTimeUnit`

* 基准测试结果的时间类型。一般选择秒、毫秒、微秒。

#### `@Benchmark`

* 方法级注解，表示该方法是需要进行 benchmark 的对象，用法和 JUnit 的 `@Test` 类似。

#### `@Param`

* 属性级注解，`@Param` 可以用来指定某项参数的多种情况。特别适合用来测试一个函数在不同的参数输入的情况下的性能。

#### `@Setup`

* 方法级注解，这个注解的作用就是我们需要在测试之前进行一些**准备工作** ，比如对一些数据的初始化之类的。

#### `@TearDown`

* 方法级注解，这个注解的作用就是我们需要在测试之后进行一些**结束工作** ，比如关闭线程池，数据库连接等的，主要用于资源的回收等.

#### @State

* 当使用`@Setup`参数的时候，必须在类上加这个参数，不然会提示无法运行。

* `State` 用于声明某个类是一个“状态”，然后接受一个 Scope 参数用来表示该状态的共享范围。因为很多 benchmark 会需要一些表示状态的类，JMH 允许你把这些类以依赖注入的方式注入到 benchmark 函数里。Scope 主要分为三种。

* 1. Thread: 该状态为每个线程独享。
  2. Group: 该状态为同一个组里面所有线程共享。
  3. Benchmark: 该状态在所有线程间共享。

#### 启动方法

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

