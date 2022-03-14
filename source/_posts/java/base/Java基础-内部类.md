---
title: Java基础-内部类
mermaid: true
date: 2021-06-11 22:11:28
cover: /img/cover/Java.jpg
tags:
- 内部类
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

* [Java内部类详解](https://www.cnblogs.com/dearcabbage/p/10609838.html)
* [Java内部类以及使用场景](https://www.cnblogs.com/firefrost/p/5064210.html)

### 内部类的定义

> 将一个类定义在**另一个类里面**或者**方法里面**,这样的类被称为**内部类**

### 内部类的分类

* 成员内部类
* 局部内部类
* 匿名内部类
* 静态内部类

#### 内部类的特点

> * 它体现了一种代码的隐藏机制和访问控制机制，内部类与所在外部类有一定的关系，往往只有该外部类调用此内部类，所以没有必要专门用一个Java文件存放这个类。
>
> * 一般的类是不允许用private修饰符的，但是内部类却可以，该实例中，类Inner只对Outer可见，其他的类无法访问的到Inner类。
> * 注意，这里有个例外，如果内部类的访问修饰符被设定为public，那么它是可以被其他类使用的，但是必须经由外部类来实例化内部类。

#### 成员内部类

> 定义在另一个类中

##### **示例**

```java
// 外部类
class External{
	// 内部类
	class Inner{
	
	}
}
```

##### **特点**

* 成员内部类可以***无条件访问外部类的属性和方法***;

* 外部类想要访问内部类的属性或方法时,必须要创建一个内部类对象,然后通过该对象访问内部类的属性或方法;

  ```java
  // 外部类
  @Slf4j
  class External {
      private int age =20;
      private String name="External";
      public void externalMethod() {
          log.info("外部类的方法externalMethod");
          // 外部类无法直接调用内部类的方法/获取内部类的属性,需要创建对象来访问
          Inner inner = new Inner();
          log.info("内部类的属性age: {}", inner.innerAge);
          log.info("内部类的属性name: {}", inner.innerName);
          inner.innerMethod();
      }
      // 内部类
      class Inner {
          private int innerAge =10;
          private String innerName="Inner";
          public void innerMethod() {
              // 无条件获取外部类的属性和方法
              log.info("内部类的方法innerMethod");
              log.info("外部类的属性age: {}", age);
              log.info("外部类的属性name: {}", name);
          }
          public void innerMethod2() {
              log.info("内部类的方法innerMethod2");
              externalMethod();
          }
      }
  }
  // 输出
  22:44:10.053 [main] INFO com.holelin.sundry.test.common.External - 外部类的方法externalMethod
  22:44:10.056 [main] INFO com.holelin.sundry.test.common.External - 内部类的属性age:10
  22:44:10.057 [main] INFO com.holelin.sundry.test.common.External - 内部类的属性name:Inner
  22:44:10.057 [main] INFO com.holelin.sundry.test.common.External - 内部类的方法innerMethod
  22:44:10.057 [main] INFO com.holelin.sundry.test.common.External - 外部类的属性age:20
  22:44:10.057 [main] INFO com.holelin.sundry.test.common.External - 外部类的属性name:External
  ```

* 如果**成员内部类的属性或者方法与外部类的同名**，将导致外部类的这些属性与方法在内部类被隐藏，也可按照该格式调用:**外部类.this.属性/方法**。

  ```java
  // 外部类
  @Slf4j
  class External {
      private int age = 20;
      private String name = "External";
      private Date time;
  
      public void externalMethod() {
          log.info("外部类的方法externalMethod");
          Inner inner = new Inner();
          log.info("内部类的属性age: {}", inner.innerAge);
          log.info("内部类的属性name: {}", inner.innerName);
          inner.innerMethod();
      }
  
      public void repeatMethod() {
          log.info("外部类的方法repeatMethod");
      }
  
      // 内部类
      class Inner {
          private int innerAge = 10;
          private String innerName = "Inner";
          private Date time;
  
          public void innerMethod() {
              // 无条件获取外部类的属性和方法
              log.info("内部类的方法innerMethod");
              log.info("外部类的属性age: {}", age);
              log.info("外部类的属性name: {}", name);
          }
  
          public void innerMethod2() {
              log.info("内部类的方法innerMethod2");
              externalMethod();
          }
  
          public void repeatMethod() {
              log.info("内部类的方法repeatMethod");
              // 内部类和外部类方法同名 需要按照`外部类.this.同名方法名称`
              External.this.repeatMethod();
              // 内部类和外部类方法同名 需要按照`外部类.this.属性名称`
              Date tempTime = External.this.time;
          }
      }
  }
  ```
  
* 内部类只是一个编译时的概念，一旦编译成功，就会成为完全不同的两个类。对于一个名为Outer的外部类和其内部定义的名为Inner的内部类，编译完后会生成`External.class`和`External$Inner.class`两个类

  ![img](https://www.holelin.cn/img/java/inner-class/%E5%86%85%E9%83%A8%E7%BC%96%E8%AF%91%E5%90%8E.png)

##### **类的创建**

* 成员内部类是寄生于外部类，创建内部类对象就必须先创造外部类对象。之后创建内部类有两种方式

  * `外部类实例.new 内部类()`
  * 通过外部类间接创建内部类


```java
public class InnerTest {
    public static void main(String[] args) {
        External external = new External();
        external.externalMethod();
        external.repeatMethod();
        // 方法一: `外部类实例.new 内部类()`
        External.Inner inner1 = external.new Inner();
        // 方法二: 通过外部类间接创建内部类
        External.Inner inner2 = external.getInner();
    }
}

// 外部类
@Slf4j
class External {
    private int age = 20;
    private String name = "External";
    private Date time;

    public void externalMethod() {
        log.info("外部类的方法externalMethod");
        Inner inner = new Inner();
        log.info("内部类的属性age: {}", inner.innerAge);
        log.info("内部类的属性name: {}", inner.innerName);
        inner.innerMethod();
    }

    public void repeatMethod() {
        log.info("外部类的方法repeatMethod");
    }

    public Inner getInner() {
        return new Inner();
    }

    // 内部类
    class Inner {
        private int innerAge = 10;
        private String innerName = "Inner";
        private Date time;

        public void innerMethod() {
            // 无条件获取外部类的属性和方法
            log.info("内部类的方法innerMethod");
            log.info("外部类的属性age: {}", age);
            log.info("外部类的属性name: {}", name);
        }

        public void innerMethod2() {
            log.info("内部类的方法innerMethod2");
            externalMethod();
        }

        public void repeatMethod() {
            log.info("内部类的方法repeatMethod");
            // 内部类和外部类方法同名 需要按照`外部类.this.同名方法名称`
            External.this.repeatMethod();
            // 内部类和外部类方法同名 需要按照`外部类.this.属性名称`
            Date tempTime = External.this.time;
        }
    }
}
```

##### **访问权限**

```java
// 外部类
class External{
    // 内部类
    private class PrivateInner{}
    // 内部类
    public class PublicInner{}
    // 内部类
    class DefaultInner{}
    // 内部类
    protected class ProtectedInner {}
}
```

  ![img](https://www.holelin.cn/img/java/inner-class/%E5%86%85%E9%83%A8%E7%B1%BB%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90.png)

##### 使用场景

*  不可能有其他类使用该内部类
* 该内部类不能被其他类使用，可能会导致错误。这是内部类使用比较多的一个场景。

#### 局部内部类

> 局部内部类是定义在一个方法或者一个作用域里面的类，
>
> * 它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内。
> * 局部内部类就像方法里面的局部变量一样，是不能有public、protected、private及static修饰符的。

##### 示例

```java
class External {
    public void externalMethod() {
        class Inner {

        }
    }
}
```

##### 特点

* 和成员内部类的区别在于局部内部类的访问权限仅限于方法或作用域内。
* **注意事项**：局部内部类就像局部变量一样，前面不能访问修饰符以及static修饰符。

#### 匿名内部类

> 通过接口来实现
>
> 匿名内部类本质上是一个重写或实现了父类或接口的子类对象。

##### 示例

```java
public class InnerTest {
    public static void main(String[] args) {
        new AnonymousInner(){
            @Override
            public void doSomething() {

            }
        };
        new External(){
            @Override
            public void externalMethod() {
                                
            }
        };
    }
}

interface AnonymousInner {
    void doSomething();
}

class External{
	public void externalMethod() {}
}
```

#### 特点

* 匿名内部类在编译的时候由系统自动起名为`InnerTest$1.class`。
* 匿名内部类用于集成其他类或者实现接口，并不需要增加额外的方法，只是对集成方法的实现或是重写

##### 使用场景

* 当类或接口类型作为参数传递时，可以直接使用匿名内部类方式创建对应的对象(**接口回调**)

#### 静态内部类

>  静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static

##### 示例

```java
class External {
    static class Inner {
    }
}
```

##### 特点

* 静态内部类是不需要属于外部类的对象(它不持有指向外部类对象的引用this),属于外部类,并且它不能使用外部类的非静态成员或方法;
* 因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依附于具体对象。它唯一的作用就是随着类的加载（而不是随着对象的产生）而产生。

```java
class External {
    private static int age = 10;
    private String name = "name";

    public void externalMethod() {
        log.info("外部类的方法externalMethod");
    }
    public static void externalStaticMethod() {
        log.info("外部类的方法externalMethod");
    }
    static class Inner {
        public void innerMethod() {
            log.info("外部类私有静态成员变量: {}", age);
            externalStaticMethod();
            // 报错,静态内部类无法引用非外部类的非静态属性/方法
            log.info("外部类私有成员变量: {}", name);
            externalMethod();

        }
    }
}
```

##### 类的创建

```java
public class InnerTest {
    public static void main(String[] args) {
        External.Inner inner = new External.Inner();
        inner.innerMethod();
    }
}
```





