---
title: RxJava-学习笔记
date: 2021-09-03 11:50:41
tags:
- RxJava
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

* [ReactiveX/RxJava文档中文版](https://www.kancloud.cn/luponu/rxjava_zh)

### 名词术语

* Observable 可观察对象，在Rx中定义为更强大的Iterable，在观察者模式中是被观察的对象，一旦数据产生或发生变化，会通过某种方式通知观察者或订阅者;
  * Single 一个特殊的Observable，只发射单个数据。
  * 在一些ReactiveX实现里，还存在一种被称作*Connectable*的Observable，不管有没有观察者订阅它，这种Observable都不会开始发射数据，除非Connect方法被调用。
* Observer 观察者对象，监听Observable发射的数据并做出响应，Subscriber是它的一个特殊实现;
* emit 直译为发射，发布，发出，含义是Observable在数据产生或变化时发送通知给Observer，调用Observer对应的方法;
* items 直译为项目，条目，在Rx里是指Observable发射的数据项;

### `Observable`

* 在异步模型中流程像这样的：

  1. 定义一个方法，这个方法拿着某个异步调用的返回值做一些有用的事情。这个方法是观察者的一部分。
  2. 将这个异步调用本身定义为一个Observable
  3. 观察者通过订阅(Subscribe)操作关联到那个Observable
  4. 继续你的业务逻辑，等方法返回时，Observable会发射结果，观察者的方法会开始处理结果或结果集

* Subscribe方法用于将观察者连接到Observable，你的观察者需要实现以下方法的一个子集：

  - **onNext(T item)**

    Observable调用这个方法发射数据，方法的参数就是Observable发射的数据，这个方法可能会被调用多次，取决于你的实现。

  - **onError(Exception ex)**

    当Observable遇到错误或者无法返回期望的数据时会调用这个方法，这个调用会终止Observable，后续不会再调用onNext和onCompleted，onError方法的参数是抛出的异常。

  - **onComplete**

    正常终止，如果没有遇到错误，Observable在最后一次调用onNext之后调用此方法。

* 根据Observable协议的定义，**onNext可能会被调用零次或者很多次，最后会有一次onCompleted或onError调用（不会同时），传递数据给onNext通常被称作发射，onCompleted和onError被称作通知。**

* Rx的操作符让你可以用声明式的风格组合异步操作序列，它拥有回调的所有效率优势，同时又避免了典型的异步系统中嵌套回调的缺点。

  * **[创建操作]** Create, Defer, Empty/Never/Throw, From, Interval, Just, Range, Repeat, Start, Timer

    * [**`just( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974459) — 将一个或多个对象转换成发射这个或这些对象的一个Observable
    * [**`from( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974457) — 将一个Iterable, 一个Future, 或者一个数组转换成一个Observable
    * [**`repeat( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974461) — 创建一个重复发射指定数据或数据序列的Observable
    * [**`repeatWhen( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974461) — 创建一个重复发射指定数据或数据序列的Observable，它依赖于另一个Observable发射的数据
    * [**`create( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974454) — 使用一个函数从头创建一个Observable
    * [**`defer( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974455) — 只有当订阅者订阅才创建Observable；为每个订阅创建一个新的Observable
    * [**`range( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974460) — 创建一个发射指定范围的整数序列的Observable
    * [**`interval( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974458) — 创建一个按照给定的时间间隔发射整数序列的Observable
    * [**`timer( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974463) — 创建一个在给定的延时之后发射单个数据的Observable
    * [**`empty( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974456) — 创建一个什么都不做直接通知完成的Observable
    * [**`error( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974456) — 创建一个什么都不做直接通知错误的Observable
    * [**`never( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974456) — 创建一个不发射任何数据的Observable

  * **[变换操作]** Buffer, FlatMap, GroupBy, Map, Scan和Window

    * [**`map( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974468) — 对序列的每一项都应用一个函数来变换Observable发射的数据序列
    * [**`flatMap( )`, `concatMap( )`, and `flatMapIterable( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974466) — 将Observable发射的数据集合变换为Observables集合，然后将这些Observable发射的数据平坦化的放进一个单独的Observable
    * [**`switchMap( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974466) — 将Observable发射的数据集合变换为Observables集合，然后只发射这些Observables最近发射的数据
    * [**`scan( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974469) — 对Observable发射的每一项数据应用一个函数，然后按顺序依次发射每一个值
    * [**`groupBy( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974467) — 将Observable分拆为Observable集合，将原始Observable发射的数据按Key分组，每一个Observable发射一组不同的数据
    * [**`buffer( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974465) — 它定期从Observable收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个
    * [**`window( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974470) — 定期将来自Observable的数据分拆成一些Observable窗口，然后发射这些窗口，而不是每次发射一项
    * [**`cast( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974468) — 在发射之前强制将Observable发射的所有数据转换为指定类型

  * **[过滤操作]** Debounce, Distinct, ElementAt, Filter, First, IgnoreElements, Last, Sample, Skip, SkipLast, Take, TakeLast

    * [**`filter( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974475) — 过滤数据
    * [**`takeLast( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974483) — 只发射最后的N项数据
    * [**`last( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974478) — 只发射最后的一项数据
    * [**`lastOrDefault( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974478) — 只发射最后的一项数据，如果Observable为空就发射默认值
    * [**`takeLastBuffer( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974483) — 将最后的N项数据当做单个数据发射
    * [**`skip( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974480) — 跳过开始的N项数据
    * [**`skipLast( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974481) — 跳过最后的N项数据
    * [**`take( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974482) — 只发射开始的N项数据
    * [**`first( )` and `takeFirst( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974476) — 只发射第一项数据，或者满足某种条件的第一项数据
    * [**`firstOrDefault( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974476) — 只发射第一项数据，如果Observable为空就发射默认值
    * [**`elementAt( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974474) — 发射第N项数据
    * [**`elementAtOrDefault( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974474) — 发射第N项数据，如果Observable数据少于N项就发射默认值
    * [**`sample( )` or `throttleLast( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974479) — 定期发射Observable最近的数据
    * [**`throttleFirst( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974479) — 定期发射Observable发射的第一项数据
    * [**`throttleWithTimeout( )` or `debounce( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974472) — 只有当Observable在指定的时间后还没有发射数据时，才发射一个数据
    * [**`timeout( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974504) — 如果在一个指定的时间段后还没发射数据，就发射一个异常
    * [**`distinct( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974473) — 过滤掉重复数据
    * [**`distinctUntilChanged( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974473) — 过滤掉连续重复的数据
    * [**`ofType( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974475) — 只发射指定类型的数据
    * [**`ignoreElements( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974477) — 丢弃所有的正常数据，只发射错误或完成通知

  * **[组合操作]** And/Then/When, CombineLatest, Join, Merge, StartWith, Switch, Zip

    * [**`startWith( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974489) — 在数据序列的开头增加一项数据
    * [**`merge( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974488) — 将多个Observable合并为一个
    * [**`mergeDelayError( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974488) — 合并多个Observables，让没有错误的Observable都完成后再发射错误通知
    * [**`zip( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974491) — 使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果
    * [**`and( )`, `then( )`, and `when( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974485) — (`rxjava-joins`) 通过模式和计划组合多个Observables发射的数据集合
    * [**`combineLatest( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974486) — 当两个Observables中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果
    * [**`join( )` and `groupJoin( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974487) — 无论何时，如果一个Observable发射了一个数据项，只要在另一个Observable发射的数据项定义的时间窗口内，就将两个Observable发射的数据合并发射
    * [**`switchOnNext( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974490) — 将一个发射Observables的Observable转换成另一个Observable，后者发射这些Observables最近发射的数据

    > (`rxjava-joins`) — 表示这个操作符当前是可选的`rxjava-joins`包的一部分，还没有包含在标准的RxJava操作符集合里

  * **[错误处理]** Catch和Retry

    * 很多操作符可用于对Observable发射的`onError`通知做出响应或者从错误中恢复，例如，你可以：
      1. 吞掉这个错误，切换到一个备用的Observable继续发射数据
      2. 吞掉这个错误然后发射默认值
      3. 吞掉这个错误并立即尝试重启这个Observable
      4. 吞掉这个错误，在一些回退间隔后重启这个Observable
    * 这是操作符列表：
      - [**`onErrorResumeNext( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Catch.md#onErrorResumeNext) — 指示Observable在遇到错误时发射一个数据序列
      - [**`onErrorReturn( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Catch.md#onErrorReturn) — 指示Observable在遇到错误时发射一个特定的数据
      - [**`onExceptionResumeNext( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Catch.md#onExceptionResumeNext) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)指示Observable遇到错误时继续发射数据
      - [**`retry( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Retry.md#retry) — 指示Observable遇到错误时重试
      - [**`retryWhen( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Retry.md#retryWhen) — 指示Observable遇到错误时，将错误传递给另一个Observable来决定是否要重新给订阅这个Observable

  * **[辅助操作]** Delay, Do, Materialize/Dematerialize, ObserveOn, Serialize, Subscribe, SubscribeOn, TimeInterval, Timeout, Timestamp, Using

    * [**`materialize( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974498) — 将Observable转换成一个通知列表convert an Observable into a list of Notifications
    * [**`dematerialize( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974498) — 将上面的结果逆转回一个Observable
    * [**`timestamp( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974505) — 给Observable发射的每个数据项添加一个时间戳
    * [**`serialize( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974500) — 强制Observable按次序发射数据并且要求功能是完好的
    * [**`cache( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974522) — 记住Observable发射的数据序列并发射相同的数据序列给后续的订阅者
    * [**`observeOn( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974499) — 指定观察者观察Observable的调度器
    * [**`subscribeOn( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974502) — 指定Observable执行任务的调度器
    * [**`doOnEach( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，对Observable发射的每个数据项使用
    * [**`doOnCompleted( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，对正常完成的Observable使用
    * [**`doOnError( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，对发生错误的Observable使用
    * [**`doOnTerminate( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，对完成的Observable使用，无论是否发生错误
    * [**`doOnSubscribe( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，在观察者订阅时使用
    * [**`doOnUnsubscribe( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，在观察者取消订阅时使用
    * [**`finallyDo( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974497) — 注册一个动作，在Observable完成时使用
    * [**`delay( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974496) — 延时发射Observable的结果
    * [**`delaySubscription( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974496) — 延时处理订阅请求
    * [**`timeInterval( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974503) — 定期发射数据
    * [**`using( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974506) — 创建一个只在Observable生命周期存在的资源
    * [**`single( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974476) — 强制返回单个数据，否则抛出异常
    * [**`singleOrDefault( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974476) — 如果Observable完成时返回了单个数据，就返回它，否则返回默认数据
    * [**`toFuture( )`**, **`toIterable( )`**, **`toList( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974507) — 将Observable转换为其它对象或数据结构

  * **[条件和布尔操作]** All, Amb, Contains, DefaultIfEmpty, SequenceEqual, SkipUntil, SkipWhile, TakeUntil, TakeWhile

    * 条件操作符

      - [**`amb( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Amb) — 给定多个Observable，只让第一个发射数据的Observable发射全部数据
      - [**`defaultIfEmpty( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#DefaultIfEmpty) — 发射来自原始Observable的数据，如果原始Observable没有发射数据，就发射一个默认数据
      - (`rxjava-computation-expressions`) [**`doWhile( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Repeat) — 发射原始Observable的数据序列，然后重复发射这个序列直到不满足这个条件为止
      - (`rxjava-computation-expressions`) [**`ifThen( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Defer) — 只有当某个条件为真时才发射原始Observable的数据序列，否则发射一个空的或默认的序列
      - [**`skipUntil( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#SkipUntil) — 丢弃原始Observable发射的数据，直到第二个Observable发射了一个数据，然后发射原始Observable的剩余数据
      - [**`skipWhile( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#SkipWhile) — 丢弃原始Observable发射的数据，直到一个特定的条件为假，然后发射原始Observable剩余的数据
      - (`rxjava-computation-expressions`) [**`switchCase( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Defer) — 基于一个计算结果，发射一个指定Observable的数据序列
      - [**`takeUntil( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#TakeUntil) — 发射来自原始Observable的数据，直到第二个Observable发射了一个数据或一个通知
      - [**`takeWhile( )` and `takeWhileWithIndex( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#TakeWhile) — 发射原始Observable的数据，直到一个特定的条件为真，然后跳过剩余的数据
      - (`rxjava-computation-expressions`) [**`whileDo( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Repeat) — 如果条件为`true`，则发射源Observable数据序列，并且只要条件保持为`true`就重复发射此数据序列

      > (`rxjava-computation-expressions`) — 表示这个操作符当前是可选包 `rxjava-computation-expressions` 的一部分，还没有包含在标准RxJava的操作符集合里
      >
      
    * 布尔操作符
    
      - [**`all( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#All) — 判断是否所有的数据项都满足某个条件
      - [**`contains( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Contains) — 判断Observable是否会发射一个指定的值
      - [**`exists( )` and `isEmpty( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Contains) — 判断Observable是否发射了一个值
      - [**`sequenceEqual( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Conditional.md#Sequenceequal) — 判断两个Observables发射的序列是否相等
    
  * **[算术和集合操作]** Average, Concat, Count, Max, Min, Reduce, Sum
  
    * #### `rxjava-math` 模块的操作符
  
      - [**`averageInteger( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Average) — 求序列平均数并发射
      - [**`averageLong( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Average) — 求序列平均数并发射
      - [**`averageFloat( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Average) — 求序列平均数并发射
      - [**`averageDouble( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Average) — 求序列平均数并发射
      - [**`max( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Max) — 求序列最大值并发射
      - [**`maxBy( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Max) — 求最大key对应的值并发射
      - [**`min( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Min) — 求最小值并发射
      - [**`minBy( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Min) — 求最小Key对应的值并发射
      - [**`sumInteger( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Sum) — 求和并发射
      - [**`sumLong( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Sum) — 求和并发射
      - [**`sumFloat( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Sum) — 求和并发射
      - [**`sumDouble( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Sum) — 求和并发射
    - 其它聚合操作符
  
      - [**`concat( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Concat) — 顺序连接多个Observables
      - [**`count( )` and `countLong( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Count) — 计算数据项的个数并发射结果
      - [**`reduce( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Reduce) — 对序列使用reduce()函数并发射最终的结果
      - [**`collect( )`**](https://www.kancloud.cn/luponu/rxjava_zh/Mathematical.md#Reduce) — 将原始Observable发射的数据放到一个单一的可变的数据结构中，然后返回一个发射这个数据结构的Observable
      - [**`toList( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974507) — 收集原始Observable发射的所有数据到一个列表，然后返回这个列表
      - [**`toSortedList( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974507) — 收集原始Observable发射的所有数据到一个有序列表，然后返回这个列表
      - [**`toMap( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974507) — 将序列数据转换为一个Map，Map的key是根据一个函数计算的
      - [**`toMultiMap( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974507) — 将序列数据转换为一个列表，同时也是一个Map，Map的key是根据一个函数计算的
  
  * **[转换操作]** To
  
  * **[连接操作]** Connect, Publish, RefCount, Replay
  
    * [**`ConnectableObservable.connect( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974519) — 指示一个可连接的Observable开始发射数据
    * [**`Observable.publish( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974520) — 将一个Observable转换为一个可连接的Observable
    * [**`Observable.replay( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974522) — 确保所有的订阅者看到相同的数据序列，即使它们在Observable开始发射数据之后才订阅
    * [**`ConnectableObservable.refCount( )`**](https://www.kancloud.cn/luponu/rxjava_zh/974521) — 让一个可连接的Observable表现得像一个普通的Observable
  
  * **[反压操作]** 用于增加特殊的流程控制策略的操作符
  
* 在RxJava中，一个实现了_Observer_接口的对象可以订阅(*subscribe*)一个_Observable_ 类的实例。订阅者(subscriber)对Observable发射(*emit*)的任何数据或数据序列作出响应。这种模式简化了并发操作，因为它不需要阻塞等待Observable发射数据，而是创建了一个处于待命状态的观察者哨兵，哨兵在未来某个时刻响应Observable的通知。

### `Single`

* RxJava（以及它派生出来的RxGroovy和RxScala）中有一个名为**Single**的Observable变种。
* Single类似于Observable，不同的是，它总是只发射一个值，或者一个错误通知，而不是发射一系列的值。
* 因此，不同于Observable需要三个方法onNext, onError, onCompleted，订阅Single只需要两个方法：
  - onSuccess - Single发射单个的值到这个方法
  - onError - 如果无法发射需要的值，Single发射一个Throwable对象到这个方法
- Single只会调用这两个方法中的一个，而且只会调用一次，调用了任何一个方法之后，订阅关系终止。

#### Single的操作符

| 操作符                | 返回值     | 说明                                                         |
| :-------------------- | :--------- | :----------------------------------------------------------- |
| compose               | Single     | 创建一个自定义的操作符                                       |
| concat and concatWith | Observable | 连接多个Single和Observable发射的数据                         |
| create                | Single     | 调用观察者的create方法创建一个Single                         |
| error                 | Single     | 返回一个立即给订阅者发射错误通知的Single                     |
| flatMap               | Single     | 返回一个Single，它发射对原Single的数据执行flatMap操作后的结果 |
| flatMapObservable     | Observable | 返回一个Observable，它发射对原Single的数据执行flatMap操作后的结果 |
| from                  | Single     | 将Future转换成Single                                         |
| just                  | Single     | 返回一个发射一个指定值的Single                               |
| map                   | Single     | 返回一个Single，它发射对原Single的数据执行map操作后的结果    |
| merge                 | Single     | 将一个Single(它发射的数据是另一个Single，假设为B)转换成另一个Single(它发射来自另一个Single(B)的数据) |
| merge and mergeWith   | Observable | 合并发射来自多个Single的数据                                 |
| observeOn             | Single     | 指示Single在指定的调度程序上调用订阅者的方法                 |
| onErrorReturn         | Single     | 将一个发射错误通知的Single转换成一个发射指定数据项的Single   |
| subscribeOn           | Single     | 指示Single在指定的调度程序上执行操作                         |
| timeout               | Single     | 它给原有的Single添加超时控制，如果超时了就发射一个错误通知   |
| toSingle              | Single     | 将一个发射单个值的Observable转换为一个Single                 |
| zip and zipWith       | Single     | 将多个Single转换为一个，后者发射的数据是对前者应用一个函数后的结果 |
