---
title: Java基础-注解
mermaid: true
date: 2021-07-01 16:55:35
cover: /img/cover/Java.jpg
tags:
- 注解
categories:
- Java
- Base
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

* [【对线面试官】今天来聊聊Java注解](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247483821&idx=1&sn=e9003410a8d3c8a092de0c4d2002bedd&scene=21#wechat_redirect)
* [玩转Java注解：元注解、内置注解、自定义注解的原理和实现](https://mp.weixin.qq.com/s/ulsX3LvTgeYVZFWVsPjgDw)

#### 注解概念

* Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种注释机制。重点：和 Javadoc 不同，Java 标注可以通过反射获取标注内容。
* 在编译器生成类文件时，标注 可以被嵌入到字节码中。Java 虚拟机可以保留标注内容，在运行时可以获取到标注内容 

### 内置注解

#### `@Override`重写

* 概念：检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。

#### `@Deprecated` 过期警告

* 概念：标记过时方法。如果使用该方法，会报编译警告。在开发中，我们经常能遇到这样的情况

#### `@SuppressWarnings` 忽略警告

* 概念：指示编译器去忽略注解中声明的警告。

### 元注解

#### `@Retention`作用域

* 概念：表示在什么级别保存该注解信息。在实际开发中，我们一般都写RUNTIME，除非项目有特殊需求

  > 注解是代码的特殊标记,可以在编译(`RetentionPolicy.SOURCE`),类加载(`RetentionPolicy.CLASS`),运行时被读取(`RetentionPolicy.RUNTIME`)

* `SOURCE`和`CLASS`分别需要基础`AbstractProcessor`,实现`process`方法去处理自定义注解

* `RUNTIME`是日常开发中用的最多的,配合Java反射机制

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

#### `@Documented` 作用文档

* 概念：将此注解包含在 javadoc 中 ，它代表着此注解会被javadoc工具提取成文档。
* 无参的注解，作用域为`RetentionPolicy.RUNTIME`，运行时有用！这个只是用来作为标记，了解即可，在实际运行后会将该注解写入javadoc中，方便查看。

#### `@Target` 目标

* 概念：标记这个注解应该是使用在哪种 Java 成员上面
* 注意这里是数组格式的参数，证明可以传多个值。
  - `@Target(ElementType.TYPE)`——接口、类、枚举、注解
  - `@Target(ElementType.FIELD)`——字段、枚举的常量
  - `@Target(ElementType.METHOD)`——方法
  - `@Target(ElementType.PARAMETER)`——方法参数
  - `@Target(ElementType.CONSTRUCTOR)` ——构造函数
  - `@Target(ElementType.LOCAL_VARIABLE)`——局部变量
  - `@Target(ElementType.ANNOTATION_TYPE)`——注解
  - `@Target(ElementType.PACKAGE)`——包

#### `@Inherited` 继承

* 概念：标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)。

#### 新注解

从 Java 7 开始，额外添加了 3 个注解:

- `@SafeVarargs` - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- `@FunctionalInterface` - Java 8 开始支持，标识一个匿名函数或函数式接口。
- `@Repeatable` - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。
