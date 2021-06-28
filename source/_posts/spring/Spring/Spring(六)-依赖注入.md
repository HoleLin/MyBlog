---
title: Spring依赖注入
date: 2021-06-07 23:13:46
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 依赖注入
categories:
- Spring
---

### 参考文献

* [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-autowired-exceptions)

#### 依赖注入模式和类型

#### 手动模式

> 配置或编程的方式,提前安排注入规则

* XML资源配置元信息;
* Java注解配置元信息;
* API配置元信息;

#### 自动模式

> 实现方提供依赖自动关联的方式,按照内建的注入规则

* Autowiring(自动绑定)

### 依赖注入的类型

| 依赖注入类型 | 配置原数据举例                                    |
| ------------ | ------------------------------------------------- |
| setter方法   | \<proeprty name="user" ref="userBean"/>           |
| 构造器       | \<construcot-arg name="user" ref="userBean"/>     |
| 字段         | @Autowired<br />private User user                 |
| 方法         | @Autowired<br /> public void user(User user){...} |
| 接口回调     | class MyBean implements BeanFactoryAware{....}    |

### 自动绑定(`Autowiring`)模式

| 模式        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 默认值,未激活Autowiring需要手动绑定指定依赖对象              |
| byName      | 根据被注入属性的名称作为Bean名称进行依赖查找,并将对象设置到该属性 |
| byType      | 根据被注入属性的类型作为依赖进行查找,并将对象设置到该属性    |
| constructor | 特殊byType,用于构造器                                        |

#### 自动绑定(`Autowiring`)限制和不足

> ##### Limitations and Disadvantages of Autowiring
>
> Autowiring works best when it is used consistently across a project. If autowiring is not used in general, it might be confusing to developers to use it to wire only one or two bean definitions.
>
> Consider the limitations and disadvantages of autowiring:
>
> - Explicit dependencies in `property` and `constructor-arg` settings always override autowiring. You cannot autowire simple properties such as primitives, `Strings`, and `Classes` (and arrays of such simple properties). This limitation is by-design.
> - Autowiring is less exact than explicit wiring. Although, as noted in the earlier table, Spring is careful to avoid guessing in case of ambiguity that might have unexpected results. The relationships between your Spring-managed objects are no longer documented explicitly.
> - Wiring information may not be available to tools that may generate documentation from a Spring container.
> - Multiple bean definitions within the container may match the type specified by the setter method or constructor argument to be autowired. For arrays, collections, or `Map` instances, this is not necessarily a problem. However, for dependencies that expect a single value, this ambiguity is not arbitrarily resolved. If no unique bean definition is available, an exception is thrown.
>
> In the latter scenario, you have several options:
>
> - Abandon autowiring in favor of explicit wiring.
> - Avoid autowiring for a bean definition by setting its `autowire-candidate` attributes to `false`, as described in the [next section](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-factory-autowire-candidate).
> - Designate a single bean definition as the primary candidate by setting the `primary` attribute of its `<bean/>` element to `true`.
> - Implement the more fine-grained control available with annotation-based configuration, as described in [Annotation-based Container Configuration](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-annotation-config).

* **优点**
  * 自动装配可以大大地减少属性和构造器参数的指派。
  * 自动装配也可以在解析对象时更新配置。
* **缺点**
  * 在`property`和`constructor-arg`设置中的依赖总是重载自动装配，我们无法对原始类型（如int，long，boolean等就是首字母小写的那些类型），还有String，Classes做自动装配。这是受限于设计。
    自动装配跟直接装配（explicit wiring）相比较，在准确性方便还是差那么点，虽然没有明确地说明，但是Spring还是尽量避免这种模棱两可的情况，导致出现没预料到的结果。
    Spring容器生成文档的工具可能会不能使用装配的信息。
    容器中多个bean的定义可能要对setter和构造器参数做类型匹配才能做依赖注入，虽然对于array，collection和map来说不是啥问题，但是对于只有单一值的依赖来讲，这就有点讲不清楚了，所以如果没有唯一的bean定义，那只能抛出异常。

#### Setter方法注入

* 手动模式
  * XML资源配置元信息
  * Java注解配置元信息
  * API配置元信息
* 自动模式
  * byName
  * byType

#### 构造器注入

* 手动模式

  * XML资源配置元信息
  * Java注解配置元信息
  * API配置元信息
* 自动模式

  * constructor

#### 字段注入

* 手动模式

  * Java注解配置元信息

    * @Autowired

      >  **会忽略静态字段(static)**

    * @Resource

    * @Inject(可选)

#### 方法注入

* 手动模式
  * Java注解配置元信息
    * @Autowired
    * @Resource
    * @Inject(可选)
    * @Bean

#### 接口回调注入

##### Aware系列回调接口

* 自动模式

  | 内建接口                       | 说明                                             |
  | ------------------------------ | ------------------------------------------------ |
  | BeanNameAware                  | 获取BeanName                                     |
  | BeanClassLoaderAware           | 获取加载当前Bean Class的ClassLoader              |
  | BeanFactoryAware               | 获取Ioc容器-BeanFactory                          |
  | EnvironmentAware               | 获取Environment对象                              |
  | EmbeddedValueResolverAware     | 获取StringValueResolver对象,用于占位符处理       |
  | ResourceLoaderAware            | 获取资源加载-ResourceLoder                       |
  | ApplicationEventPublisherAware | 获取ApplicationEventPublisher对象,用于Spring事件 |
  | MessageSourceAware             | 获取MessageSource对象,用于Spring国际化           |
  | ApplicationContextAware        | 获取Spring应用上下文-ApplicationContext对象      |

### 依赖注入类型选择

* 注入选型
  * **构造器注入**: 低依赖(强制依赖)
  * **Setter注入**: 多依赖
  * **字段注入**: 便利性
  * **方法注入**: 方法注入

### 基础类型的注入

#### 原生类型(Primitive)

> boolean,byte,char,short,int,floot,long,double

#### 标量类型(Scalar)

> Number,Character,Boolean,Enum,Locale,Charset,Currency,Properties,UUID

#### 常规类型(General)

> Object,String,TimeZone,Calendar,Optional等

#### Spring类型

> Resource,InputSource,Formatter等

#### 集合类型

* 数组类型(Array)

  > 原生类型,标量类型,常规类型,Spring类型

* 集合类型(Collection)

  * Collection
    * List
    * Set(`SortedSet`,`NavigableSet`,`EnumSet`)
  * Map:
    * Properties

### 限定注入

#### 使用注解@Qualifier限定

* 通过Bean名称限定
* 通过分组限定

#### 基于注解@Qualifier扩展限定

* 自定义注解 如Spring Cloud @LoadBlanced

### 延迟注入

#### 使用API `ObjectFactory`延迟注入

* 单一类型
* 集合类型

#### 使用API`ObjectProvider`延迟注入

* 单一类型
* 集合类型

### 依赖处理过程

* 入口:  `DefaultListableBeanFactory#resolveDependency`
* 依赖描述符: `DependencyDescriptor`
* 自动绑定候选对象处理器: `AutowireCandidateResolver`

#### `@Autowired`注入过程

* 元信息解析
* 依赖查找
* 依赖注入(字段,方法)

#### @Inject注入过程

* 若`JSR-330`存在与`ClassPath`,复用`AutowiredAnnotationBeanPostProcessor`实现

  ```java
  
  	/**
  	 * Create a new {@code AutowiredAnnotationBeanPostProcessor} for Spring's
  	 * standard {@link Autowired @Autowired} and {@link Value @Value} annotations.
  	 * <p>Also supports JSR-330's {@link javax.inject.Inject @Inject} annotation,
  	 * if available.
  	 */
  	@SuppressWarnings("unchecked")
  	public AutowiredAnnotationBeanPostProcessor() {
  		this.autowiredAnnotationTypes.add(Autowired.class);
  		this.autowiredAnnotationTypes.add(Value.class);
  		try {
  			this.autowiredAnnotationTypes.add((Class<? extends Annotation>)
  					ClassUtils.forName("javax.inject.Inject", AutowiredAnnotationBeanPostProcessor.class.getClassLoader()));
  			logger.trace("JSR-330 'javax.inject.Inject' annotation found and supported for autowiring");
  		}
  		catch (ClassNotFoundException ex) {
  			// JSR-330 API not available - simply skip.
  		}
  	}
  ```

#### Java通用注解注入原理

* `CommonAnnotationBeanPostProcessor`

  * **注入注解**

    * `javax.xml.ws.WebServiceRef`
    * `javax.ejb.EJB`
    * `javax.annotation.Resource`

    ```java
    	static {
    		try {
    			@SuppressWarnings("unchecked")
    			Class<? extends Annotation> clazz = (Class<? extends Annotation>)
    					ClassUtils.forName("javax.xml.ws.WebServiceRef", CommonAnnotationBeanPostProcessor.class.getClassLoader());
    			webServiceRefClass = clazz;
    		}
    		catch (ClassNotFoundException ex) {
    			webServiceRefClass = null;
    		}
    
    		try {
    			@SuppressWarnings("unchecked")
    			Class<? extends Annotation> clazz = (Class<? extends Annotation>)
    					ClassUtils.forName("javax.ejb.EJB", CommonAnnotationBeanPostProcessor.class.getClassLoader());
    			ejbRefClass = clazz;
    		}
    		catch (ClassNotFoundException ex) {
    			ejbRefClass = null;
    		}
    
    		resourceAnnotationTypes.add(Resource.class);
    		if (webServiceRefClass != null) {
    			resourceAnnotationTypes.add(webServiceRefClass);
    		}
    		if (ejbRefClass != null) {
    			resourceAnnotationTypes.add(ejbRefClass);
    		}
    	}
    ```

  * **生命周期注解**

    *  `javax.annotation.PostConstruct`
    *  `javax.annotation.PreDestroy`

    ```java
    	/**
    	 * Create a new CommonAnnotationBeanPostProcessor,
    	 * with the init and destroy annotation types set to
    	 * {@link javax.annotation.PostConstruct} and {@link javax.annotation.PreDestroy},
    	 * respectively.
    	 */
    	public CommonAnnotationBeanPostProcessor() {
    		setOrder(Ordered.LOWEST_PRECEDENCE - 3);
    		setInitAnnotationType(PostConstruct.class);
    		setDestroyAnnotationType(PreDestroy.class);
    		ignoreResourceType("javax.xml.ws.WebServiceContext");
    	}
    ```

  #### 自定义依赖注入理解

  * 基于`AutowiredAnnotationBeanPostProcessor`实现

  #### 自定义实现

  * 生命周期处理
    * `InstantiationAwareBeanPostProcessor`
    * `MergedBeanDefinitionPostProcessor`
  * 元数据
    * `InjectedElement`
    * `InjectionMetadata`
