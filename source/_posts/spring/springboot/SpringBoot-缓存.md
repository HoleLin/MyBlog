---
title: SpringBoot-缓存
date: 2022-03-10 13:57:13
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
categories:
- Spring
- Spring Framework
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

* SpringBoot2 实战之旅

### Spring Cache

* Spring Cache是Spring3.1以后引入的新技术.
* 其核心思想是:当调用一个缓存方法时,会把该方法参数和返回值作为一个键值对存档在缓存中,等到下次利用同样的参数来调用该方法时将不再执行该方法,而是直接从缓存中获取结果进行返回,从实现缓存的功能.
* 需要加上`@EnableCaching`来开启缓存

#### `@Cacheable`

* `@Cacheable`注解用于标记缓存,也就是对`@Cacheable`注解的位置进行缓存.
* `@Cacheable`可以在方法或类上进行标记,当对方法进行标记时,表示此方法支持缓存;当对类进行标记时,表明当前类中所有方法都支持缓存.
* Spring在每次调用方法前都会根据key查询当前Cache中是否存在相同的key的缓存元素,如果存在,就不再执行该方法,而是直接从缓存中获取结果进行返回,否则执行该方法并将返回结果存入指定的缓存中.
* `@Cacheable`通常会搭配3个属性进行使用
  * `value`: 在使用`@Cacheable`注解的时候,value属性是必须的,用于指定Cache的名称.表明当前缓存的返回值用于哪个缓存上.
  * `key`: 和名称一样,用于指定缓存对应的key,key属性不是必须指定的,如果没有指定key,Spring就会使用默认策略生成的key.
    * 其中默认策略规定:如果当前缓存方法没有参数,那么当前key为0;如果当前缓存方法有一个参数,那么以key为参数;如果当前缓存方法有多个参数,那么key为所有参数的hashcode值.
    * 也可以使用EL表达式来指定当前缓存方法的key,通常为`#参数名`
  * `condition`: 主要用于指定当前缓存的触发条件.

```java
@Cacheable(value="users",key="#user.id",condition="#user.id%2==0")
public User findUser(User user){
	return new User();
}
```

#### `@CachePut`

* 从名称上来看,`@CachePut`只是用于将标记该注解的方法的返回值放入缓存中,无论缓存中是否包含当前缓存,只是以键值的形式将执行结果放入缓存中.

#### `@CacheEvict`

* `@CacheEvict`注解用于清除缓存数据



