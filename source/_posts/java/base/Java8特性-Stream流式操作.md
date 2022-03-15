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

* `java.util.stream.Collectors`

