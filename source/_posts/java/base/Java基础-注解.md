---
title: Java基础-注解
mermaid: true
date: 2021-07-01 16:55:35
cover: /img/cover/Java.jpg
tags:
- 注解
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

* [【对线面试官】今天来聊聊Java注解](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247483821&idx=1&sn=e9003410a8d3c8a092de0c4d2002bedd&scene=21#wechat_redirect)

#### 注解概念

> 注解是代码的特殊标记,可以在编译(`RetentionPolicy.SOURCE`),类加载(`RetentionPolicy.CLASS`),运行时被读取(`RetentionPolicy.`RUNTIME)

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

