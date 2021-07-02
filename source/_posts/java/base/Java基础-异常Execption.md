---
title: Java基础-异常Exception
date: 2021-06-11 22:07:30
cover: /img/cover/Java.jpg
tags:
- 异常Exception
categories:
- Java
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

### **受检查异常 和 不受检查异常**

* **受检查的异常(checked)**：这种在编译时被强制检查的异常称为"受检查的异常"。即在方法的声明中声明的异常。对于这种异常，方法强制处理或者通过 `throws` 子句声明。其中一种情况是 Exception 的子类但不是 `RuntimeException` 的子类。
  * **对于checked异常的处理方式**
    * 当前方法明确知道如何处理异常,程序应该使用`try...catch`块来捕获异常,然后在对应的`catch`中修复该异常;
    * 当前方法不知道如何修复异常时,应该定义该方法声明抛出异常;

* **不受检查的异常**(**unchecked**)：在方法的声明中没有声明，但在方法的运行过程中发生的各种异常被称为"不被检查的异常"。这种异常是错误，会被自动捕获。非受检查是 `RuntimeException` 的子类，在编译阶段不受编译器的检查。

### **Java中所有异常或者错误都继承`Throwable`，分为三类：**

* `Error`:所有都继承自`Error`，表示致命的错误，比如内存不够，字节码不合法等。程序无法处理;
* `Exception`:这个属于应用程序级别的异常，这类异常必须捕捉。
* `RuntimeException`:`RuntimeException`继承了`Exception`，而不是直接继Error,这个表示系统异常，比较严重

![img](http://www.chenjunlin.vip/img/java/exception/Throwable.png)

<img src="http://www.chenjunlin.vip/img/java/exception/exception.webp" alt="img" style="zoom:67%;" />

> 图中红色部分为受检查异常,他们必须被捕获或者在函数中声明为抛出异常;

### Java语言中的异常处理包括四个环节

* **声明异常**
  * `throws`关键字可以方法上声明该方法要抛出的异常,然后在方法内部通过.`throw`抛出异常;
* **抛出异常**
  * `throw`用于抛出异常(在方法体中,会触发异常);
* **捕获异常**
  * `try`是用于检测被宝珠的语句块是否出现异常,如果有异常,则抛出异常,并执行`catch`语句;
* **处理异常**
  * `catch`用于捕获从`try`中抛出的异常并作出处理;



