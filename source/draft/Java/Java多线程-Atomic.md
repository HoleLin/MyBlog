---
title: Java多线程-Atomic
mermaid: true
date: 2021-06-24 00:45:42
cover: /img/cover/Java.jpg
tags:
- ThreadLocal
- 多线程
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

### 基本类型类

> 用于通过原子的方式更新基本类型

* AtomicBoolean: 原子更新布尔类型
* AtomicInteger: 原子更新整型
* AtomicLong: 原子更新长整型

### 数组

> 通过原子的方式更新数组里的某个元素

* AtomicIntegerArray: 原子更新整型数组里的元素
* AtomicLongArray: 原子更新长整型数组里的元素
* AtomicReferenceArray: 原子更新引用类型数组里的元素

### 引用类型

> 如果要原子的更新多个变量，就需要使用这个原子更新引用类型提供的类

* AtomicReference: 原子更新引用类型
* AtomicReferenceFieldUpdater:原子更新引用类型里的字段
* AtomicMarkableReference: 原子更新带有标记位的引用类型

### 字段类

> 如果我们只需要某个类里的某个字段，那么就需要使用原子更新字段类

* AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器
* AtomicLongFieldUpdater: 原子更新长整型字段的更新器
* AtomicStampedReference: 原子更新带有版本号的引用类型
