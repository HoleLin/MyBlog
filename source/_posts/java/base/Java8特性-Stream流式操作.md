---
title: Java8特性-Stream流式操作
date: 2022-03-14 17:16:44
cover: /img/cover/Java.jpg
tags:
- Stream
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

### Stream API

* 只能对实现了`java.util.Collection`接口的类做流操作.
* 流(Stream)是**数据渠道**,用于操作数据源(集合,数组等)所生成的**元素序列**.
* **集合讲的是数据,流讲的是计算**
* Stream自己不会存储元素
* Stream不会改变源对象.相反,它们会返回一个持有结果的新Stream.
* Stream操作是延迟执行的,这意味着它们等到需要的时候才执行.
* Stream支持同步执行,也支持异步执行.

#### 惰性流

* 流是惰性的,在达到终止条件前不会处理元素,达到终止条件后逐个处理每个元素.如果遇到短路操作,那么只要满足所有条件,流处理就会终止. 
* 对于集合而言,必须执行完所有操作才能进行下一步操作.对于流而言,各种中间操作构成一条流水线,但在流达到终止操作前不会处理任何元素,达到终止操作后只处理所需的值.
* 流处理并非任何情况下都有意义:如果进行任何状态操作(如排序或求和),就不得不处理所有值.但是如果无状态操作后跟一个短路终止操作,流处理的优点还是很明显的.

### Stream的操作三个步骤

#### 创建Stream

* 一个数据源(如集合,数组)获取一个流.

##### Java8中`Collection`接口

* Java8中`Collection`接口被扩展,提供了两个获取流的方法:
  * `default Stream<E> stream()`: 返回一个顺序流;
  * `default Stream<E> parallelStream()`: 返回一个并行流;

##### 由数组创建流

* Java8中`Arrays`的静态方法`stream()`可以获取数组流
* `static <T> Stream<T> stream(T[] array)`: 返回一个流

##### 由值创建流

* 可以使用静态方法`Stream.of()`通过显示值创建一个流,它可以接收任意数量的参数.
* `static<T> Stream<T> of(T... values)`

##### 创建无限流

* 由函数创建流,使用静态方法`Stream.iterate()`或`Stream.generate()`创建无限流.

* `static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) `

  ```java
          Stream.iterate(LocalDate.now(), it->it.plusDays(1))
                  .limit(5)
                  .forEach(System.out::println);
  ```

* `static<T> Stream<T> generate(Supplier<? extends T> s)`

  ```java
       Stream.generate(Math::random)
                  .limit(10)
                  .forEach(System.out::println);
  ```

##### `IntStream`,`LongStream()`的`range`和`rangeClosed`方法

* `static IntStream range(int startInclusive, int endExclusive)`
* `static IntStream rangeClosed(int startInclusive, int endInclusive)`
* `static LongStream range(long startInclusive, final long endExclusive)`
* `static LongStream rangeClosed(long startInclusive, final long endInclusive)`

#### 中间操作

* 一个中间操作链,对数据源的数据进行处理.
* 多个中间操作可以连起来形成一个流水线,除非流水线触发终止操作,否则中间操作不会执行任何的处理,而在终止操作是,一次性全部处理称为"惰性求值"

##### 筛选与切片

| 操作                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| `filter(Predicate p)` | 接收lambda,从流中排除/过滤某些元素                           |
| `distinct()`          | 筛选通过流所生成的元素的hashCode()和equals()去除重复元素     |
| `limit(long maxSize)` | 截断流,使其元素不超过给定数量                                |
| `skip(long n)`        | 跳过元素,返回一个扔掉前n个元素的流.若流中元素不足n个,则返回一个空流,与limit(n)互补. |

##### 映射

| 操作                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `map(Function f)`                 | 接收一个函数作为参数,该函数会被应用到**每个元素**上并将其映射成一个新元素. |
| `mapToDouble(ToDoubleFunction f)` | 接收一个函数作为参数,该函数会被应用到**每个元素**上并将其映射成一个新的DoubleStream. |
| `mapToInt(ToIntFunction f)`       | 接收一个函数作为参数,该函数会被应用到**每个元素**上并将其映射成一个新的IntStream. |
| `mapToLong(ToLongFunction f)`     | 接收一个函数作为参数,该函数会被应用到**每个元素**上并将其映射成一个新的LongStream. |
| `flatMap(Function f)`             | 接收一个函数作为参数,**将流中的每个值都换成另一个流,然后把所有流连接成一个流** |

* `map`和`flatMap`的区别
  * 若需要将每个元素转换为一个值,则使用`Stream.map`方法
  * 若需要将每个元素转换为多个值,且需要将生成的流"展平",则使用`Stream.flatMap`方法
    * 方法参数Function产生的一个输出值流
    * 生成的元素被展平为一个新的流.

##### 排序

| 操作                   | 说明                               |
| ---------------------- | ---------------------------------- |
| `sorted()`             | 产生一个新流,其中按自然顺序排序.   |
| `sorted(Comparator c)` | 产生一个新流,其中按比较器顺序排序. |

##### 装箱流

* 使用基本类型流创建集合

* 解决方法

  * 使用`java.util.stream.IntStream`接口定义的`boxed`方法来包装元素.

    ```java
       IntStream.of(1,2,3,4)
            .boxed()
            .collect(Collectors.toList());
    ```

  * 可以使用合适的包装器类来映射值

    ```java
       IntStream.of(1,2,3,4)
            .mapToInt(Integer::valueof)
            .collect(Collectors.toList());
    ```

  * 还可以使用`collect`方法的三参数形式

    ```java
       IntStream.of(1,2,3,4)
            .collect(ArrayList<Integer>::new,
            Arrays::add,ArrayList::addAll);
    ```

    * Supplier是ArrayList\<Integer>的构造函数.累加器为add方法,表示如何添加单个元素.仅在并行操作中使用的组合器(combiner)是addAll方法,它能将两个列表合二为一.

##### 使用`peek`方法调试流

```java
        IntStream.rangeClosed(1,10)
                .peek(n-> System.out.printf("original:%d%n",n))
                .map(n->n*2)
                .peek(n-> System.out.printf("doubled: %d%n",n))
                .filter(n->n%3==0)
                .peek(n-> System.out.printf("filtered: %d%n",n))
                .sum();
```

##### 流的拼接

* `Stream.concat`方法适用于合并两个流.如果需要合并多个流,则需要使用`Stream.flatMap`
* `concat`方法将创建一个惰性的拼接流,其元素是第一个流的所有元素,后跟第二个流的所有元素.

#### 终止操作(终端操作)

* 一个终止操作,执行中间操作链,并产生结果.

* 终止操作会从流的流水线生成结果,其中结果可以是任何不是流的值,如`List`,`Integer`,甚至为`void`

##### 查找与匹配

| 操作                     | 说明                                                      |
| ------------------------ | --------------------------------------------------------- |
| `allMatch(Predicate p)`  | 检查是否匹配所有元素                                      |
| `anyMatch(Predicate p)`  | 检查是否至少匹配一个元素                                  |
| `noneMatch(Predicate p)` | 检查是否没有匹配所有元素                                  |
| `findFirst()`            | 返回第一个元素                                            |
| `findAny()`              | 返回当前流中的任意元素                                    |
| `count()`                | 返回流中元素总数                                          |
| `max(Comparator c)`      | 返回流中最大值                                            |
| `min(Comparator c)`      | 返回流中最小值                                            |
| `forEach(Consumer c)`    | 内部迭代.使用`Collection`接口需要用户做迭代被称为外部迭代 |

##### 归约

* Java的函数式范式经常使用"映射-筛选-归约"(map-filter-reduce)的过程处理数据.
  * 首先map操作将一种类型的流转换为另一种类型接着filter操作产生一个新的流,它仅包含所需的元素,最后通过终止操作从流中生成单个值.

| 操作                                                  | 说明                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `T reduce(T identity, BinaryOperator<T> accumulator)` | identity为累加器的初始值,可以将流中元素反复结合起来,得到一个值,返回T |
| `Optional<T> reduce(BinaryOperator<T> accumulator)`   | 可以将流中元素反复结合起来,得到一个值,返回Optional\<T>       |

* 示例

  ```java
  // 1-10 求和
  IntStream.rangeClosed(1, 10).reduce(Integer::sum).orElse(0)
    
  final int reduce = IntStream.rangeClosed(1, 10).reduce(0, (x, y) -> x +2 * y);
  ```

##### 收集

| 操作                                                      | 说明                |
| --------------------------------------------------------- | ------------------- |
| `<R, A> R collect(Collector<? super T, A, R> collector);` | 将流转换为其他形式. |

* `java.util.stream.Collectors`,使用工具类的`toList`,`toSet`,`toCollection`,`toMap`
* `Function.identity()`

##### 分区和分组

* `Collectors.partitioningBy`方法将元素拆分为满足`Predicate`与不满足`Predicate`的两类
* `Collectors.groupingBy`方法生成一个由类别构成的Map,其中值为每个类别中的元素.

### 比较器和收集器

#### 使用比较器实现排序

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T var1, T var2);

    boolean equals(Object var1);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator)((Serializable)((c1, c2) -> {
            int res = this.compare(c1, c2);
            return res != 0 ? res : other.compare(c1, c2);
        }));
    }
    // ...略
}    
```

* 使用传入`Comparator`的`Stream.sorted`方法,`Comparator`既可以通过lambda表达式实现,也可以使用`Comparator`接口定义的某种`comparing`方法生成.
* `Stream.sorted`方法根据类的自然熟悉怒生成一个新的排序流,自然熟悉怒是通过实现`java.util.Comparable`接口来指定的.
* 从Java1.2引入集合框架开始,工具类`Collections`就已存在.`Collections`类定义的静态方法`sort`传入List作为参数,但返回是void.这种排序是破坏性的,会修改锁提供的集合.即`Collection.sort`方法不符合Java8所倡导的不可变性(immutability)置于首要位置的函数式编程原则.
* Java8采用`Stream.sorted`方法实现相同的排序,但不对原始集合进行修改,而是生成一个新的流.
* 若希望以其他方式排序,可以使用`sorted`方法的重载形式,传入`Comparator`作为参数.

#### 对Map排序

* Map接口始终包含一个称为`Map.Entry`的公共静态内部接口,它表示一个键值对.Map接口定义的entrySet方法返回`Map.Entry`元素的Set.在Java8之前,getKey和getValue是`Map.Entry`接口两种最常用的方法,二者分别返回与某个条目的对应的键和值.

| 方法                                          | 描述                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| `comparingByKey`                              | 返回一个比较器,它根据键的自然顺序比较`Map.Entry`             |
| `comparingByValue`                            | 返回一个比较器,它根据值的自然顺序比较`Map.Entry`             |
| `comparingByKey(Comparator<? super K> cmp) `  | 返回一个比较器,它使用给定的`Comparator`并根据键比较`Map.Entry` |
| `comparingByValue(Comparator<? super V> cmp)` | 返回一个比较器,它使用给定的`Comparator`并根据值比较`Map.Entry` |

### 实现`Collector`接口

```java
package java.util.stream;

import java.util.Collections;
import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;


public interface Collector<T, A, R> {
    /**
     * A function that creates and returns a new mutable result container.
     *
     * @return a function which returns a new, mutable result container
     */
    Supplier<A> supplier();

    /**
     * A function that folds a value into a mutable result container.
     *
     * @return a function which folds a value into a mutable result container
     */
    BiConsumer<A, T> accumulator();

    /**
     * A function that accepts two partial results and merges them.  The
     * combiner function may fold state from one argument into the other and
     * return that, or may return a new result container.
     *
     * @return a function which combines two partial results into a combined
     * result
     */
    BinaryOperator<A> combiner();

    /**
     * Perform the final transformation from the intermediate accumulation type
     * {@code A} to the final result type {@code R}.
     *
     * <p>If the characteristic {@code IDENTITY_FINISH} is
     * set, this function may be presumed to be an identity transform with an
     * unchecked cast from {@code A} to {@code R}.
     *
     * @return a function which transforms the intermediate result to the final
     * result
     */
    Function<A, R> finisher();

    /**
     * Returns a {@code Set} of {@code Collector.Characteristics} indicating
     * the characteristics of this Collector.  This set should be immutable.
     *
     * @return an immutable set of collector characteristics
     */
    Set<Characteristics> characteristics();
	// ...略
}
```

* `Supplier<A> supplier()` : 使用`Supplier<A>`创建累加容器(accumulator container)
  * 用于创建累加器临时结果所用的容器
* `BiConsumer<A, T> accumulator()`: 使用`BiConsumer<A, T>`为累加器容器添加一个新的数据元素
  * 用于将一个元素添加到累加器
* `BinaryOperator<A> combiner()`: 使用`BinaryOperator<A>`合并两个累加容器
  * `BinaryOperator`表示输入类型和输出类型相同,因此可以将两个累加器合二为一.
* `Function<A, R> finisher()`: 使用`Function<A, R>`将累加器容器转换为结果容器.
  * `Function`将累加器转换为所需的结果容器.

```java
    /**
     * Characteristics indicating properties of a {@code Collector}, which can
     * be used to optimize reduction implementations.
     */
    enum Characteristics {
        /**
         * Indicates that this collector is <em>concurrent</em>, meaning that
         * the result container can support the accumulator function being
         * called concurrently with the same result container from multiple
         * threads.
         *
         * <p>If a {@code CONCURRENT} collector is not also {@code UNORDERED},
         * then it should only be evaluated concurrently if applied to an
         * unordered data source.
         */
        CONCURRENT,

        /**
         * Indicates that the collection operation does not commit to preserving
         * the encounter order of input elements.  (This might be true if the
         * result container has no intrinsic order, such as a {@link Set}.)
         */
        UNORDERED,

        /**
         * Indicates that the finisher function is the identity function and
         * can be elided.  If set, it must be the case that an unchecked cast
         * from A to R will succeed.
         */
        IDENTITY_FINISH
    }
```

* `CONCURRENT`: 表示结果容器支持多线程在结果容器上并发地调用累加器;
* `UNORDERED`:  表示集合操作无须保留元素的出现顺序(encounter order);
* `IDENTITY_FINISH`: 表示终止器函数返回其参数而不做任何修改.

```java
ollector.of(TreeSet::new,
                SortedSet<String>::add,
                (left, right) -> {
                    left.addAll(right);
                    return left;
                },
                Collections::unmodifiableNavigableSet);
```

### 闭包复合

* 使用`Function`,`Consumer`与`Predicate`接口中定义的默认的复合方法.

#### `Function`

```java

default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
```

* `compose`方法在原始函数**之前**应用参数
* `andThen`方法在原始函数**之后**应用参数

```java
 				Function<Integer, Integer> add = x -> x + 2;
        Function<Integer, Integer> mult = x -> x * 3;
        final Function<Integer, Integer> multadd = add.compose(mult);
        final Function<Integer, Integer> addmult = add.andThen(mult);
        System.out.println("multadd(1): " + multadd.apply(1));
        System.out.println("addmult(1): " + addmult.apply(1));

// multadd(1): 5
// addmult(1): 9
```

#### `Consumer`

```java
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
```

#### `Predicate`

```java
   default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
```

