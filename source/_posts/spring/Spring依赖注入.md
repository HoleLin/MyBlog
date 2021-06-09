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
| setter方法   | <proeprty name="user" ref="userBean"/>            |
| 构造器       | <construcot-arg name="user" ref="userBean"/>      |
| 字段         | @Autowired<br />private User user                 |
| 方法         | @Autowired<br /> public void user(User user){...} |
| 接口回调     | class MyBean implements BeanFactoryAware{....}    |

### 自动绑定(Autowiring)模式

| 模式        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 默认值,未激活Autowiring需要手动绑定指定依赖对象              |
| byName      | 根据被注入属性的名称作为Bean名称进行依赖查找,并将对象设置到该属性 |
| byType      | 根据被注入属性的类型作为依赖进行查找,并将对象设置到该属性    |
| constructor | 特殊byType,用于构造器                                        |

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

* 注入类型
  * 构造器注入: 低依赖(强制依赖)
  * Setter注入: 多依赖
  * 字段注入: 便利性
  * 方法注入: 方法注入

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
    * Set(SortedSet,NavigableSet,EnumSet)
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

#### @Autowired注入过程

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
