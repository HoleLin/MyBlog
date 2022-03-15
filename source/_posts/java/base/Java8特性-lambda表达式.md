---
title: Java8特性-lambda表达式
date: 2022-03-14 13:06:03
cover: /img/cover/Java.jpg
tags:
- lambda
categories:
- Java
- Java 8特性
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

* Java攻略:Java常见问题的简单解法

### lambda表达式

* **函数式接口**:是一种包含单一抽象方法(single abstract method)的接口.类通过为接口中的所有方法提供实现来实现任何接口,这可以通过顶级类(top-level class),内部类甚至匿名内部类完成.

* lambda表达式必须匹配接口中**单一方法签名**的**参数类型**和**返回类型**,这被称为与方法签名兼容.因此lambda表达式属于接口方法的实现,可以将其赋值给该接口类型的引用.
  * Java库中不存在名为Lambda的类,lambda表达式只能被赋值给函数式接口引用.

* lambda表达式在任何情况下都不能脱离上下文存在,上下文指定了将表达式给哪个函数式接口.
  * lambda表达式既可以是方法的参数,也可以是方法的返回类型,还可以被赋给引用.
  * 无论哪种情况,赋值类型必须为函数式接口.

#### 方法引用

* 使用**双冒号表示法(::)**将示例引用或类名与方法分开.

* 语法:
  * `object::instanceMethod`: 引用特定对象的实例方法.如`System.out::println`
  * `Class::staticMethod`: 引用静态方法,如`Math::max`
  * `Class::instanceMethod`: 调用特定类型的任意对象的实例方法,如`String::length`

#### 构造函数引用

* 在方法中使用`new`关键字

```java
List<String> names = Arrays.asList("A","B","C");
List<Person> people = names.stream().map(name->new Person(name)).collect(Collectors.toList());
// ==>
List<Person> people = names.stream().map(Person::new).collect(Collectors.toList());
```

* `Person::new`的作用是引用Person类的构造函数.与所有lambda表达式类似,有上下文决定执行哪个构造函数.由于上下文提供一个字符串,则使用单参数的String构造函数.

##### 复制构造函数

* 复制构造函数(copy constructor)传入一个Person参数,并返回一个具有相同特性的新Person.

  ```java
  public Person(Person p){
  	this.name = p.name;
  }
  ```

  ```java
  final List<Person> people = Stream.of(before).map(Person::new).collect(Collectors.toList());
  ```

  * 若需要将流代码从原始实例中分离出来,复制构造函数将很有用.

##### 可变参数构造函数

```java
public Person(String... names){
	this.name = Arrays.stream(names).collect(Collectors.joining(""));
}
```

```java
names.stream()
	.map(name->name.split(""))
	.map(Person::new)
	.collect(Collectors.toList());
```

##### 数组

* 构造函数引用也可以和数组一起使用.若希望采用实例的数组(Person[])而非列表,可以使用Stream接口定义的toArray方法

  ```java
  <A> A[] toArray(IntFunction<A[]> generator)
  ```

  * toArray方法采用A表示返回数组的泛型类型(generic type).数组包含流的元素,由所提供的generator函数创建.

  ```java
  Person[] people = names.stream()
  									.map(Person::new)
  									.toArray(Person::new);
  ```

#### 函数式接口

* 创建只包含单一抽象方法的接口,并为其添加`@FunctionalInterface`注解.

### `java.util.function`包

* `java.util.function`包中接口分为四类
  * `Consumer`消费型接口
  * `Supplier`供给型(生产者)接口
  * `Predicate`谓词型接口
  * `Function`功能型接口

#### `java.util.function.Consumer`接口

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents an operation that accepts a single input argument and returns no
 * result. Unlike most other functional interfaces, {@code Consumer} is expected
 * to operate via side-effects.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #accept(Object)}.
 *
 * @param <T> the type of the input to the operation
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

#### `java.util.function.Supplier`接口

```java
package java.util.function;

/**
 * Represents a supplier of results.
 *
 * <p>There is no requirement that a new or distinct result be returned each
 * time the supplier is invoked.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #get()}.
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

#### `java.util.function.Predicate`接口

* `Predicate`接口主要用于流的筛选.

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a predicate (boolean-valued function) of one argument.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #test(Object)}.
 *
 * @param <T> the type of the input to the predicate
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * AND of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code false}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ANDed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * AND of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * Returns a predicate that represents the logical negation of this
     * predicate.
     *
     * @return a predicate that represents the logical negation of this
     * predicate
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * OR of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code true}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ORed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * OR of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * Returns a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}.
     *
     * @param <T> the type of arguments to the predicate
     * @param targetRef the object reference with which to compare for equality,
     *               which may be {@code null}
     * @return a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }

    /**
     * Returns a predicate that is the negation of the supplied predicate.
     * This is accomplished by returning result of the calling
     * {@code target.negate()}.
     *
     * @param <T>     the type of arguments to the specified predicate
     * @param target  predicate to negate
     *
     * @return a predicate that negates the results of the supplied
     *         predicate
     *
     * @throws NullPointerException if target is null
     *
     * @since 11
     */
    @SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```

#### `java.util.function.Function`接口

* `Function`接口包含的单一抽象方法为`apply`,它可以将T类型的泛型输入参数转换为R类型的泛型输出值.

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a function that accepts one argument and produces a result.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    /**
     * Returns a composed function that first applies the {@code before}
     * function to its input, and then applies this function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of input to the {@code before} function, and to the
     *           composed function
     * @param before the function to apply before this function is applied
     * @return a composed function that first applies the {@code before}
     * function and then applies this function
     * @throws NullPointerException if before is null
     *
     * @see #andThen(Function)
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     *
     * @see #compose(Function)
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * Returns a function that always returns its input argument.
     *
     * @param <T> the type of the input and output objects to the function
     * @return a function that always returns its input argument
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```



