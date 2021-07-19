---
title: Java-编程技巧
date: 2021-07-19 16:39:53
cover: /img/cover/Java.jpg
tags:
- 编程技巧
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

#### Java获取资源路径

* Java中获取资源时,经常使用`class.getResource()`和`ClassLoader.getResource`获取资源时,获取的是编译之后的`class`文件资源,而不是获取`Java`源码

  ```java
  public class GetResourceTest {
      public static void main(String[] args) {
          // path不以'/'开头,默认是从此类所在的包下面取资源
          System.out.println(GetResourceTest.class.getResource(""));
          // path以'/'开头,则是从classpath根下获取
          System.out.println(GetResourceTest.class.getResourceAsStream("/"));
          // path是从classpath根获取的
          System.out.println(GetResourceTest.class.getClassLoader().getResource(""));
          // path不能以'/'开头
          System.out.println(GetResourceTest.class.getClassLoader().getResource("/"));
      }
  }
  file:/E:/Projects/Github/Java-Notes/sundry/target/classes/com/holelin/sundry/demo/
  java.io.ByteArrayInputStream@20ad9418
  file:/E:/Projects/Github/Java-Notes/sundry/target/classes/
  null
  ```

  
