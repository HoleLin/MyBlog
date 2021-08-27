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

![img](https://www.holelin.cn/img/java/exception/Throwable.png)

<img src="https://www.holelin.cn/img/java/exception/exception.webp" alt="img" style="zoom:67%;" />

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

![img](https://www.holelin.cn/img/java/exception/%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86.png)

* 当程序在运行的过程中出现异常后,那么会有`JVM`自动根据有一次的类型实例化一个与之类型匹配的异常类对象(用户不用关心`new`,由系统负责处理);
* 产生了异常对象之后会判断当前语句是否存在异常处理,如果现在没有异常处理,就交给`JVM`进行默认处理(输出异常信息,结束程序调用);
* 如果此时存在异常捕获操作,那么会有`try`语句来捕获异常类实例化对象,与`try`语句后的每一个`catch`的异常类对象进行比较,如果现在有符合的捕获类型,则使用当前`catch`的语句进行异常的处理,如果不匹配,则向下匹配其他的`catch`语句;
* 不管最后异常处理是否能够匹配,那么都要向后执行,如果此时程序中存在`finally`语句,则执行`finally`中的代码,但执行完毕后需要根据之前`catch`匹配结果来决定如何执行,如果之前已经成功的捕获了异常,那么就执行`finally`之后的代码,如果之前没有成功的捕获异常,那么就将异常交给`JVM`处理(输出异常信息,结束程序调用)
* **Tips: 在捕获异常的时候,捕获异常范围大的一定要放在捕获范围小的异常之后,否则程序编译错误;**

#### `return`和`finally`

```java
@Slf4j
public class ExceptionTest {
    public static void main(String[] args) {
        System.out.println(testReturnAndFinally());
    }

    public static int testReturnAndFinally() {
        int x = 1;
        try {
            int temp = x / 0;
            log.info("出现异常");
            return ++x;
        } catch (Exception e) {
            log.info("捕获异常");
        } finally {
            log.info("finally..");
            ++x;
        }
        log.info("返回..");
        return ++x;
    }
}
14:30:50.884 [main] INFO com.holelin.sundry.demo.ExceptionTest - 捕获异常
14:30:50.886 [main] INFO com.holelin.sundry.demo.ExceptionTest - finally..
14:30:50.886 [main] INFO com.holelin.sundry.demo.ExceptionTest - 返回..
3
```

> [The `finally` block *always* executes when the `try` block exits. This ensures that the `finally` block is executed even if an unexpected exception occurs. But `finally` is useful for more than just exception handling — it allows the programmer to avoid having cleanup code accidentally bypassed by a `return`, `continue`, or `break`. Putting cleanup code in a `finally` block is always a good practice, even when no exceptions are anticipated.](https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html)
>
> **Note:** If the `JVM` exits while the `try` or `catch` code is being executed, then the `finally` block may not execute. Likewise, if the thread executing the `try` or `catch` code is interrupted or killed, the `finally` block may not execute even though the application as a whole continues.

> `finally` 块总是在 `try` 块退出时执行。 这确保即使发生意外异常也能执行 finally 块。 但 finally 不仅仅用于异常处理——它允许程序员避免通过 return、continue 或 break 意外绕过清理代码。 将清理代码放在 finally 块中始终是一个好习惯，即使没有预料到异常也是如此。
>
>  注意：如果在执行 try 或 catch 代码时 JVM 退出，则 finally 块可能不会执行。 同样，如果执行 try 或 catch 代码的线程被中断或终止，即使应用程序作为一个整体继续运行，finally 块也可能不会执行。 

##### 以下4中特殊情况下,`finally`块不会被执行:

* 在`finally`语句块第一行发生异常.因为在其他行,`finally`块还是会得到执行.

  ```java
  @Slf4j
  public class ExceptionTest {
      public static void main(String[] args) {
          System.out.println(testReturnAndFinally());
      }
  
      public static int testReturnAndFinally() {
          int x = 1;
          try {
              int temp = x / 0;
              log.info("出现异常");
              return ++x;
          } catch (Exception e) {
              log.info("捕获异常");
          } finally {
              int temp = x / 0;
              log.info("finally..");
              ++x;
          }
          log.info("返回..");
          return ++x;
      }
  }
  14:31:55.063 [main] INFO com.holelin.sundry.demo.ExceptionTest - 捕获异常
  Exception in thread "main" java.lang.ArithmeticException: / by zero
  	at com.holelin.sundry.demo.ExceptionTest.testReturnAndFinally(ExceptionTest.java:20)
  	at com.holelin.sundry.demo.ExceptionTest.main(ExceptionTest.java:8)
  ```

* 在前面的代码中用了`System.exit(int)`退出程序.

  ```
  @Slf4j
  public class ExceptionTest {
      public static void main(String[] args) {
          System.out.println(testReturnAndFinally());
      }
  
      public static int testReturnAndFinally() {
          int x = 1;
          try {
              int temp = x / 0;
              log.info("出现异常");
              return ++x;
          } catch (Exception e) {
              System.exit(0);
              log.info("捕获异常");
          } finally {
              log.info("finally..");
              ++x;
          }
          log.info("返回..");
          return ++x;
      }
  }
  ```

* 程序所在线程死亡.

* 关闭CPU.

