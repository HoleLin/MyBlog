---
title: JVM-类文件结构
date: 2022-01-07 16:13:10
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags:
- 类文件结构
categories:
- Java
- JVM
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

* 深入理解Java虚拟机:JVM高级特性与最佳实践(第3版)

### 类文件结构

| 类型           | 名称                | 数量                  |
| -------------- | ------------------- | --------------------- |
| u4             | magic               | 1                     |
| u2             | minor_version       | 1                     |
| u2             | major_version       | 1                     |
| u2             | constant_pool_count | 1                     |
| cp_info        | constant_pool       | constant_pool_count-1 |
| u2             | access_flags        | 1                     |
| u2             | this_class          | 1                     |
| u2             | super_class         | 1                     |
| u2             | interfaces_count    | 1                     |
| u2             | interfaces          | interfaces_count      |
| u2             | fields_count        | 1                     |
| fields_info    | fields              | fields_count          |
| u2             | methods_count       | 1                     |
| method_info    | methods             | methods_count         |
| u2             | attributes_count    | 1                     |
| attribute_info | attributes          | attributes_count      |

* **无符号数**属于基本的数据类型,以u1,u2,u4,u8来分别代表1字节,2字节,4字节,8字节的无符号数,无符号数可以用来描述数字,索引引用,数量值或者按照UTF-8编码构成字符串值.
* 表是由多个无符号数或者其他表作为数据项构成的复合类型,为了便于区分,所有表的命名都习惯地以"_info"结尾.

#### 魔数和Class文件的版本

* 每个Class文件的头4个字节被称为魔数(Magic Number),它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件.**0xCAFEBABE**
* 紧接着魔数的4个字节存储的是Class文件的版本号:第5和第6个字节是次版本号(Minor Version),第7和第8个字节是主版本号(Major Version).
  * Java的版本号是从45开始的,JDK1.1之后的每个JDK大版本发布主版本号向上加1(JDK1.0~1.1使用了45.0~45.3的版本号),高版本的JDK能向下兼容以前的Class文件,但不能运行以后版本的Class文件.因为<<Java虚拟机规范>>在Class文件校验部分明确要求了即使文件格式并未发生任何变化,虚拟机也必须拒绝执行超过其版本号的Class文件.

| JDK版本     | -target参数                                                  | -source参数      | 版本号 |
| ----------- | ------------------------------------------------------------ | ---------------- | ------ |
| JDK1.1.8    | 不支持target参数                                             | 不支持source参数 | 45.3   |
| JDK1.2.2    | 不带(默认为target1.1)                                        | 1.1~1.2          | 45.3   |
| JDK1.2.2    | -target1.2                                                   | 1.1~1.2          | 46.0   |
| JDK1.3.1_19 | 不带(默认为-target1.2)                                       | 1.1~1.3          | 45.3   |
| JDK1.3.1_19 | -target1.3                                                   | 1.1~1.3          | 47.0   |
| JDK1.4.2_10 | 不带(默认为-target1.2)                                       | 1.1~1.4          | 46.0   |
| JDK1.4.2_10 | -target1.4                                                   | 1.1~1.4          | 48.0   |
| JDK5.0_11   | 不带(默认为-target1.5),后续版本不带target参数默认编译的Class文件均与其JDK版本相同 | 1.1~1.5          | 49.0   |
| JDK5.0_11   | -target1.4-source1.4                                         | 1.1~1.5          | 48.0   |
| JDK6        | 不带(默认为-target6)                                         | 1.1~6            | 50.0   |
| JDK7        | 不带(默认为-target7)                                         | 1.1~7            | 51.0   |
| JDK8        | 不带(默认为-target8)                                         | 1.1~8            | 52.0   |
| JDK9        | 不带(默认为-target9)                                         | 6~9              | 53.0   |
| JDK10       | 不带(默认为-target10)                                        | 6~10             | 54.0   |
| JDK11       | 不带(默认为-target11)                                        | 6~11             | 55.0   |
| JDK12       | 不带(默认为-target12)                                        | 6~12             | 56.0   |
| JDK13       | 不带(默认为-target13)                                        | 6~13             | 57.0   |

#### 常量池

* 紧接着主次版本号之后的是常量池入口,常量池可以比喻为Class文件的资源仓库,他是Class文件结构中与其他项目关联最多的数据,通常也是占用Class空间最大的数据项目之一,另外它还是在Class文件中第一个出现的表类型数据项目.

* 由于常量池中的常量数量是不固定的,所以在常量池的入口需要放置一项u2类型的数据,代表常量池容量计数值(constant_pool_count).**这个容量计数是从1而不是从0开始的.**

  * 在Class文件格式规范制定之时,设计者将第0项常量空出来是有特殊考虑的,这样做的目的在于,如果后面某些指向常量池的索引值的数据在特定情况下需要表达"不引用任何一个常量池项目"的含义,可以把索引值设置为0来表示.
  * Class文件结构中只有常量池的容量计数是从1开始,对于其他集合类型,包括接口索引集合,字段表集合等容量计数都是一般习惯相同,是从0开始的.

* 常量池中主要存放两大类常量:**字面量(Literal)和符号引用(Symbolic Reference)**.

  * 字面量比较接近于Java语言层面的常量概念,如文本字符串,被声明为final的常量值等.
  * 符号引用则属于编译原理的概念,其中主要包括下面几类常量:
    * 被模块导出或者开放的包(Package)
    * 类和接口的全限定名(Fully Qualified Name)
    * 字段的名称和描述符(Descriptor)
    * 方法名称和描述符
    * 方法句柄和方法类型(Method Handle,Method Type,Invoke Dynamic)
    * 动态调用点和动态常量(Dynamically-Computed Call Site,Dynamically-Computed Constant)

* 常量池中每一项常量都是一个表,最初常量表中共有11种结构各种不同的表结构数据,后来为了更好地支持动态语言调用，额外增加了 4 种动态语言相关的常量,所以截至 JDK13，常量表中分别有 17 种不同类型的常量。

  | 类型                             | 标志 | 描述                           |
  | -------------------------------- | ---- | ------------------------------ |
  | CONSTANT_Utf8_info               | 1    | UFT-8编码的字符串              |
  | CONSTANT_Integer_info            | 3    | 整型字面量                     |
  | CONSTANT_Float_info              | 4    | 浮点型字面量                   |
  | CONSTANT_Long_info               | 5    | 长整型字面量                   |
  | CONSTANT_Double_info             | 6    | 双精度浮点型字面量             |
  | CONSTANT_Class_info              | 7    | 类或接口的符号引用             |
  | CONSTANT_String_info             | 8    | 字符串类型字面量               |
  | CONSTANT_Fieldref_info           | 9    | 字段的符号引用                 |
  | CONSTANT_Methodref_info          | 10   | 类中方法的符号引用             |
  | CONSTANT_InterfaceMethodref_info | 11   | 接口中方法的符号引用           |
  | CONSTANT_NameAndType_info        | 12   | 字段或方法的部分符号引用       |
  | CONSTANT_MethodHandle_info       | 15   | 表示方法句柄                   |
  | CONSTANT_MethodType_info         | 16   | 表示方法类型                   |
  | CONSTANT_Dynamic_info            | 17   | 表示一个动态计算常量           |
  | CONSTANT_InvokeDynamic_info      | 18   | 表示一个动态方法调用点         |
  | CONSTANT_Module_info             | 19   | 表示一个模块                   |
  | CONSTANT_Package_info            | 20   | 表示一个模块中开放或者导出的包 |

  ![img](https://www.holelin.cn/img/jvm/%E5%B8%B8%E9%87%8F%E6%B1%A0%E4%B8%AD%E7%9A%84%2017%20%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%BB%93%E6%9E%84%E6%80%BB%E8%A1%A8.png)

#### 访问标志

* 在常量池之后,紧接着的2个字节代表访问标志(access_flag),这个标志用于识别一些类或者接口层次的访问信息.包括:这个Class是类还是接口,是否定义为public类型;是否定义为abstract类型;等等

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final,只有类可设置                               |
| ACC_SUPER      | 0x0020 | 是否允许使用`invokespecial`字节码指令的新语义,`invokespecial`指令的语义在JDK1.0.2发生过改变,为了区别这条指令使用哪种语义,JDK1.0.2之后编译出来的类的这个标志都必须为真. |
| ACC_INTERFACE  | 0x0200 | 标识这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型,对于接口或者抽象来说,此标志值为真,其他类型值为假 |
| ACC_SYNTHETIC  | 0x1000 | 标识这个类并非由用户代码产生的                               |
| ACC_ANNOTATION | 0x2000 | 标识这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标识这是一个枚举                                             |
| ACC_MODULE     | 0x8000 | 标识这是一个模块                                             |

* access_flags 中一共有  16 个标志位可以使用，当前只定义了其中  9 个 ，没有使用到的标志位要求一律为零.

#### 类索引,父类索引与接口索引集合

* 类索引(this_class)和父类索引(super_class)都是一个u2类型的数据,而接口索引集合(interfaces)是一组u2类型的数据的集合,Class文件中由这三项来确定该类型的继承关系.
* 类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名.
  * 由于 Java 语言不允许多重继承，所以父类索引只有一个，除了 `java.lang.Object` 之外，所有的Java 类都有父类，因此除了 `java.lang.Object` 外，所有 Java 类的父类索引都不为 0。
* 接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按 implements 关键字（如果这个 Class 文件表示的是一个接口，则应当是 extends 关键字）后的接口顺序从左到右排列在接口索引集合中

#### 字段表集合

* 字段表（field_info）用于描述接口或者类中声明的变量。Java 语言中的“字段”（Field）包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量.
* 字段可以包括的修饰符有字段的作用域（public、private、 protected 修饰符）、是实例变量还是类变量（static 修饰符）、可变性（final）、并发可见性（volatile 修饰符，是否强制从主内存读写）、可否被序列化（transient 修饰符）、字段数据类型（基本类型、对象、数组）、字段名称。

| 类型            | 名称             | 数量             |
| --------------- | ---------------- | ---------------- |
| u2              | access_flags     | 1                |
| u2              | name_index       | 1                |
| u2              | descriptor_index | 1                |
| u2              | attributes_count | 1                |
| attributes_info | attributes       | attributes_count |

* 字段修饰符放在access_flags中,它与类中的access_flags是非常类似的,都是u2的数据类型,其中可以设置的标志位和含义如下

  | 标志名称      | 标志值 | 含义                     |
  | ------------- | ------ | ------------------------ |
  | ACC_PUBLIC    | 0x0001 | 字段是否为public         |
  | ACC_PRIVATE   | 0x0002 | 字段是否为private        |
  | ACC_PROTECTED | 0x0004 | 字段是否为protected      |
  | ACC_STATIC    | 0x0008 | 字段是否为static         |
  | ACC_FINAL     | 0x0010 | 字段是否为final          |
  | ACC_VOLATILE  | 0x0040 | 字段是否为volatile       |
  | ACC_TRANSIENT | 0x0080 | 字段是否为transient      |
  | ACC_SYNTHETIC | 0x1000 | 字段是否由编译器自动生成 |
  | ACC_ENUM      | 0x4000 | 字段是否为enum           |

  * 由于语法规则的约束，ACC_PUBLIC、ACC_PRIVATE、ACC_PROTECTED 三个标志最多只能选择其一，ACC_FINAL、ACC_VOLATILE 不能同时选择。接口之中的字段必须有 ACC_PUBLIC、ACC_STATIC、ACC_FINAL 标志，这些都是由  Java 本身的语言规则所导致的

* 描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型（byte、char、double、float、int、long、short、boolean）以及代表无返回值的 void 类型都用一个大写字符来表示，而对象类型则用字符 L 加对象的全限定名来表示

  | 标识字符 | 含义                          |
  | -------- | ----------------------------- |
  | B        | 基本类型byte                  |
  | C        | 基本类型char                  |
  | D        | 基本类型double                |
  | F        | 基本类型float                 |
  | I        | 基本类型int                   |
  | J        | 基本类型long                  |
  | S        | 基本类型short                 |
  | Z        | 基本类型boolean               |
  | V        | 特殊类型void                  |
  | L        | 对象类型,如Ljava/lang/Object; |

#### 方法表集合

* Class 文件存储格式中对方法的描述与对字段的描述采用了几乎完全一致的方式.

| 类型            | 名称             | 数量             |
| --------------- | ---------------- | ---------------- |
| u2              | access_flags     | 1                |
| u2              | name_index       | 1                |
| u2              | descriptor_index | 1                |
| u2              | attributes_count | 1                |
| attributes_info | attributes       | attributes_count |

* 因为  volatile 关键字和  transient 关键字不能修饰方法，所以方法表的访问标志中没有了  ACC_VOLATILE 标志和  ACC_TRANSIENT 标志。与之相对，synchronized、native、strictfp 和  abstract 关键字可以修饰方法，方法表的访问标志中也相应地增加了  ACC_SYNCHRONIZED、ACC_NATIVE、ACC_STRICTFP 和  ACC_ABSTRACT 标志

  | 标志名称         | 标志值 | 含义                             |
  | ---------------- | ------ | -------------------------------- |
  | ACC_PUBLIC       | 0x0001 | 方法是否为public                 |
  | ACC_PRIVATE      | 0x0002 | 方法是否为private                |
  | ACC_PROTECTED    | 0x0004 | 方法是否为protected              |
  | ACC_STATIC       | 0x0008 | 方法是否为static                 |
  | ACC_FINAL        | 0x0010 | 方法是否为final                  |
  | ACC_SYNCHRONIZED | 0x0020 | 方法是否为synchronized           |
  | ACC_BRIDGE       | 0x0040 | 方法是否是由编译器产生的桥接方法 |
  | ACC_VARARGS      | 0x0080 | 方法是否接收不定参数             |
  | ACC_NATIVE       | 0x0100 | 方法是否为native                 |
  | ACC_ABSTRACT     | 0x0400 | 方法是否为abstract               |
  | ACC_STRICT       | 0x0800 | 方法是否为strictfp               |
  | ACC_SYNTHETIC    | 0x1000 | 方法是否有编译器自动产生         |

* 在 Java 语言中，要重载（Overload）一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名。 **特征签名是指一个方法中各个参数在常量池中的字段符号引用的集合**，也正是因为返回值不会包含在特征签名之中，所以 Java 语言里面是无法仅仅依靠返回值的不同来对一个已有方法进行重载的。但是在 Class 文件格式之中，特征签名的范围明显要更大一些，只要描述符不是完全一致的两个方法就可以共存。也就是说，如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法共存于同一个 Class 文件中的。

#### 属性表集合

