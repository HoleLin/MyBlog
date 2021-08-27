---
title: Java进阶-对象实例化
date: 2021-07-01 09:33:01
cover: /img/cover/Java.jpg
tags:
- 对象实例化
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

* [基础篇：详解JAVA对象实例化过程](https://juejin.cn/post/6861940021146943502)

#### 对象实例化过程

* 对象实例化过程主要分为两个部分
  * **类的加载初始化**
  * **对象的初始化**
* 要创建类的对象实例需要先加载并初始化该类,`main`方法所在的类需要先加载和初始化;
* 类初始化就是执行`<clinit>`方法,独享实例化是执行`<init>`方法;
* 一个子类要初始化需要先初始化父类;

#### 类的加载过程(5个过程)

* 类的加载机制:如果没有相应类的class，则加载class到方法区。对应着**加载->验证->准备->解析-->初始化阶段**
  * **加载**：载入class对象，不一定是从class文件获取，可以是jar包，或者动态生成的class
  * **验证**：校验class字节流是否符合当前JVM规范
  * **准备**：为**类变量**分配内存并设置变量的初始值(**默认值**)。如果是`final`修饰的对象则是赋值声明值
  * **解析**：将常量池的符号引用替换为直接引用
  * **初始化**：执行类构造器`<client>`(**注意不是对象构造器**)，为**类变量**赋值，执行静态代码块。jvm会保证子类的`<client>`执行之前，父类的<client>先执行完毕

* 其中**验证、准备、解析**3个部分称为 连接

* `<clinit>`方法由**静态变量赋值代码和静态代码块**组成；先执行类静态变量显示赋值代码，再到静态代码块代码

![img](https://www.holelin.cn/img/jvm/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.png)

#### 触发类加载的条件

* 第一次创建类的新对象时，**会触发类的加载初始化和对象的初始化函数<init>执行，这个是实例初始化，其他5个都是类初始化**

* JVM启动时会先加载初始化包含`main`方法的类

* 调用类的静态方法（如执行`invokestatic`指令）

* 对类或接口的静态字段执行读写操作（即执行`getstatic`、`putstatic`指令）；不过`final`修饰的静态字段的除外(已经赋值，`String`和基本类型，不包含包装类型)，它被初始化为一个编译时常量表达式
  * **注意**：操作静态字段时，只有直接定义这个字段的类才会被初始化；如通过其子类来操作父类中定义的静态字段，只会触发父类`<clinit>`的初始化而不是子类的初始化

* 调用Java API中的反射方法时(比调用`java.lang.Class`中的方法(`Class.forName`)，或者`java.lang.reflect`包中其他类的方法)

* 当初始化一个类时，其父类没有初始化，则需先触发父类的初始化(接口例外)

#### 对象的实例化过程

* **对象实例化过程** 其实就是执行类构造函数 对应在字节码文件中的`<init>()`方法(称之为实例构造器)；`<init>()`方法由**非静态变量、非静态代码块以及对应的构造器组成**
  * `<init>()`方法可以重载多个，类有几个构造器就有几个`<init>()`方法
  * `<init>()`方法中的代码执行顺序为：父类变量初始化，父类代码块，父类构造器，子类变量初始化，子类代码块，子类构造器。

* 静态变量，静态代码块，普通变量，普通代码块，构造器的执行顺序

![img](https://www.holelin.cn/img/jvm/%E5%AF%B9%E8%B1%A1%E5%88%9D%E5%A7%8B%E5%8C%96%E9%A1%BA%E5%BA%8F.png)

#### 类加载器和双亲委派模型

* 类加载器
  * 通过一个类的**全限定名**来获取**描述此类的二进制字节流**，实现这个动作的代码模块称为类加载器
  * 任意一个类都需要其加载器和类本身来确定类在JVM的唯一性；每个类加载器都有自己的类名称空间，同一个类class由不同的加载器加载，则被JVM判断为不同的类

* 类加载器分类
  * **启动类加载器(Bootstrap ClassLoader)**
  * **扩展类加载器(Extension ClassLoader)**
  * **应用程序类加载器(Application ClassLoader)**
  * **自定义类加载器(Custom ClassLoader)**
  
  <img src="https://www.holelin.cn/img/jvm/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B.png" alt="img" style="zoom:67%;" />
* 双亲委派模型
  * `Bootstrap ClassLoader`由C++代码实现，是虚拟机的一部分。负责加载`$JAVA_HOME/jre/lib/rt.jar` 里所有的class;
  * `Extention ClassLoader`: 负责加载`$JAVA_HOME/lib/ext`或者由`java.ext.dirs`系统属性指定的目录中的JAR包的类。由Java语言实现，父类加载器为null。
  * `Application ClassLoader`: 负责在JVM启动时加载来自Java命令的`-classpath`选项、`java.class.path`系统属性，或者`CLASSPATH`换将变量所指定的JAR包和类路径。
    * 程序可以通过`ClassLoader`的静态方法`getSystemClassLoader()`来获取系统类加载器。如果没有特别指定，则用户自定义的类加载器都以此类加载器作为父加载器。由Java语言实现，父类加载器为`ExtClassLoader`。
  * 其他的类加载器由java语言实现，独立于JVM，并且继承ClassLoader;
  * 不同的类加载器加载同一个class文件会导致出现两个类。而java给出解决方法是下层的加载器加委托上级的加载器去加载类，如果父类无法加载(在自己负责的目录找不到对应的类)，而交还下层类加载器去加载。
* 双亲委派模型好处
  * **一个可以避免类的重复加载，另外也避免了java的核心API被篡改。**

* 打破双亲委派模型

  - 自定义类加载器，重写loadClass方法；
  - 使用线程上下文类加载器；

  
