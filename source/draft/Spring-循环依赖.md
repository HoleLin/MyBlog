---
title: Spring-循环依赖
mermaid: true
date: 2021-07-01 11:09:09
cover: /img/cover/Spring.jpg
tags:
- 循环依赖
categories:
- Spring
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

* [spring：我是如何解决循环依赖的？](https://mp.weixin.qq.com/s/VpCt49_Li35caK5IaQTuNg)

#### 循环依赖

> 一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个环形调用;

* 自己依赖自己的直接依赖;

  <img src="http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%961.png" alt="img" style="zoom:80%;" />

* 两个对象之间的直接依赖;

  <img src="http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%962.png" alt="img" style="zoom:80%;" />

* 多个对象之间的间接依赖;

  <img src="http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%963.png" alt="img" style="zoom:80%;" />

#### 循环依赖的N种场景

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E4%B8%BB%E8%A6%81%E5%9C%BA%E6%99%AF.png)

##### 单例的Setter注入

```java
@Service
publicclass TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}
```

```java
@Service
publicclass TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

* 这是一个经典的循环依赖，但是它能正常运行，得益于Spring的内部机制，让我们根本无法感知它有问题，因为Spring默默帮我们解决了。

* spring内部有三级缓存：

  ```java
  	// 一级缓存
  	/** Cache of singleton objects: bean name to bean instance. */
  	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
  	// 三级缓存
  	/** Cache of singleton factories: bean name to ObjectFactory. */
  	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
  	// 二级缓存
  	/** Cache of early singleton objects: bean name to bean instance. */
  	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
  ```

  - **singletonObjects** 一级缓存，用于保存实例化、注入、初始化完成的bean实例
  - **earlySingletonObjects** 二级缓存，用于保存实例化完成的bean实例
  - **singletonFactories** 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%8D%95%E4%BE%8B%E6%B3%A8%E5%85%A5Spring%E5%A4%84%E7%90%86%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)

```java
    protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

            final String beanName = transformedBeanName(name);
            Object bean;

            // Eagerly check singleton cache for manually registered singletons.
            Object sharedInstance = getSingleton(beanName);
       		// ... 省略
    }
	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 从singletonObjects一级缓存中获取实例
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
                // 从earlySingletonObjects二级缓存中获取实例
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
                    // 从singletonFactories三级缓存中获取实例
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
                        // 加入到二级缓存中
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

```java
		protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		      // ...省略
				// Create bean instance.
				if (mbd.isSingleton()) {
                    // 分两步:
                    // 1. createBean-->doCreateBean-->addSingletonFactory加入到三级缓存中
                    // 2. getSingleton-->addSingleton加入到一级缓存中
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							// Explicitly remove instance from singleton cache: It might have been put there
							// eagerly by the creation process, to allow for circular reference resolution.
							// Also remove any beans that received a temporary reference to the bean.
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
            // ...省略
        }

		protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
			 // ...省略
            // Eagerly cache singletons to be able to resolve circular references
            // even when triggered by lifecycle interfaces like BeanFactoryAware.
            boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                    isSingletonCurrentlyInCreation(beanName));
            // 提前暴露
            if (earlySingletonExposure) {
              	// ...省略
                addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
            }
            // ...省略
        }
        /**
         * Add the given singleton factory for building the specified singleton
         * if necessary.
         * <p>To be called for eager registration of singletons, e.g. to be able to
         * resolve circular references.
         * @param beanName the name of the bean
         * @param singletonFactory the factory for the singleton object
         */
        protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
            Assert.notNull(singletonFactory, "Singleton factory must not be null");
            synchronized (this.singletonObjects) {
                // 若一级缓存中没有
                if (!this.singletonObjects.containsKey(beanName)) {
                    // 加入到三级缓存中
                    this.singletonFactories.put(beanName, singletonFactory);
                    this.earlySingletonObjects.remove(beanName);
                    this.registeredSingletons.add(beanName);
                }
            }
        }
```

```java
	/**
	 * Return the (raw) singleton object registered under the given name,
	 * creating and registering a new one if none registered yet.
	 * @param beanName the name of the bean
	 * @param singletonFactory the ObjectFactory to lazily create the singleton
	 * with, if necessary
	 * @return the registered singleton object
	 */
	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				// ...省略
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				// ...省略
				if (newSingleton) {
                    // 实例创建完成后加入到一级缓存中
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
```

##### 多例的Setter注入

* 这种注入方法偶然会有，特别是在多线程的场景下，具体代码如下：

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}
```

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

* 很多人说这种情况spring容器启动会报错，其实是不对的，我非常负责任的告诉你程序能够正常启动。
* 其实在`AbstractApplicationContext`类的`refresh`方法中告诉了我们答案，它会调用`finishBeanFactoryInitialization`方法，该方法的作用是为了spring容器启动的时候提前初始化一些bean。该方法的内部又调用了`preInstantiateSingletons`方法

```java
		// 方法路径: AbstractApplicationContext#refresh-->finishBeanFactoryInitialization-->preInstantiateSingletons
		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            // 非抽象、单例 并且非懒加载的类才能被提前初始bean
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						final FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
											((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}
```

* 而多例即`SCOPE_PROTOTYPE`类型的类，非单例，不会被提前初始化bean，所以程序能够正常启动

* 如何让他提前初始化bean呢？只需要再定义一个单例的类，在它里面注入TestService1

  ```java
  @Service
  publicclass TestService3 {
  
      @Autowired
      private TestService1 testService1;
  }
  ```

* 运行启动后报错:

  ```java
  Requested bean is currently in creation: Is there an unresolvable circular reference?
  ```



**注意：这种循环依赖问题是无法解决的，因为它没有用缓存，每次都会生成一个新对象。**

##### 构造器注入

```
@Service
publicclass TestService1 {

    public TestService1(TestService2 testService2) {
    }
}
```

```
@Service
publicclass TestService2 {

    public TestService2(TestService1 testService1) {
    }
}
```

* 启动报错`Requested bean is currently in creation: Is there an unresolvable circular reference?`

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E6%9E%84%E9%80%A0%E5%99%A8%E6%B3%A8%E5%85%A5Spring%E5%A4%84%E7%90%86%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)

##### 单例的代理对象Setter注入

* 这种注入方式其实也比较常用，比如平时使用：`@Async`注解的场景，会通过`AOP`自动生成代理对象

```java
@Service
publicclass TestService1 {

    @Autowired
    private TestService2 testService2;

    @Async
    public void test1() {
    }
}
```

```java
@Service
publicclass TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

```java
  org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'testService1': Bean with name 'testService1' has been injected into other beans [testService2] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
```

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%8D%95%E4%BE%8B%E7%9A%84%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1Setter%E6%B3%A8%E5%85%A5.png)

* 如果这时候把TestService1改个名字，改成：TestService6，其他的都不变。再重新启动一下程序，神奇般的好了。

  ```java
  @Service
  publicclass TestService6 {
  
      @Autowired
      private TestService2 testService2;
  
      @Async
      public void test1() {
      }
  }
  ```

* 这就要从spring的bean加载顺序说起了，默认情况下，spring是按照文件完整路径递归查找的，按路径+文件名排序，排在前面的先加载。所以TestService1比TestService2先加载，而改了文件名称之后，TestService2比TestService6先加载。

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%8D%95%E4%BE%8B%E7%9A%84%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1Setter%E6%B3%A8%E5%85%A5%E5%BC%82%E5%B8%B8.png)

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/%E5%8D%95%E4%BE%8B%E7%9A%84%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1Setter%E6%B3%A8%E5%85%A52.png)

##### DependsOn循环依赖

* 还有一种有些特殊的场景，比如我们需要在实例化Bean A之前，先实例化Bean B，这个时候就可以使用`@DependsOn`注解

```java
@DependsOn(value = "testService2")
@Service
publicclass TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}
```

```java
@DependsOn(value = "testService1")
@Service
publicclass TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

* 程序启动之后，执行结果：

```java
Circular depends-on relationship between 'testService2' and 'testService1'
```

* 这个个例子中本来如果TestService1和TestService2都没有加`@DependsOn`注解是没问题的，反而加了这个注解会出现循环依赖问题。
* 它会检查dependsOn的实例有没有循环依赖，如果有循环依赖则抛异常。

![img](http://www.chenjunlin.vip/img/spring/circular-dependency/DependsOn%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)

### 循环处理的解决方案

#### 生成代理对象产生的循环依赖

* 这类循环依赖问题解决方法很多，主要有：
  1. 使用`@Lazy`注解，延迟加载
  2. 使用`@DependsOn`注解，指定加载先后关系
  3. 修改文件名称，改变循环依赖类的加载顺序

#### 使用@DependsOn产生的循环依赖

* 这类循环依赖问题要找到`@DependsOn`注解循环依赖的地方，迫使它不循环依赖就可以解决问题。

#### 多例循环依赖

* 这类循环依赖问题可以通过把bean改成单例的解决。

#### 构造器循环依赖

* 这类循环依赖问题可以通过使用`@Lazy`注解解决。

