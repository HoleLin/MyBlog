---
title: Java基础-反射&&动态代理
mermaid: true
date: 2021-07-01 23:24:24
cover: /img/cover/Java.jpg
tags:
- 反射
- 动态代理
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

* [【对线面试官】Java反射 && 动态代理](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247483893&idx=1&sn=af51e626f2c2baec8cae4f4a15425957&scene=21#wechat_redirect)

#### 反射的作用

> 在运行时获取类的信息

#### 动态代理

* 动态代理是代理模式的一种;
* 代理模型有静态代理和动态代理,静态代理需要自己写代理类,实现对应的接口;
* Java中,动态代理常见的又有两种实现方式: **`JDK`动态代理**和`CGLIB`代理
  * `JDK`动态代理其实就是运用了反射机制;
  * `JDK`动态代理会帮我们实现接口的方法,通过`invokeHandler`所需要的方法进行增强;
  * `CGLIB`代理则是用`ASM`框架,通过修改其字节码生成子类来处理
