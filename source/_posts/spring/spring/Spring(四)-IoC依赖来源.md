---
title: Spring(四) IoC依赖来源
mermaid: true
date: 2021-06-27 20:58:21
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 依赖来源 
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

#### 依赖查找的来源

* 查找来源

  | 来源                    | 配置元信息                        |
  | ----------------------- | --------------------------------- |
  | Spring `BeanDefinition` | \<bean id="user" class="...User"> |
  |                         | @Bean public User user(){..}      |
  |                         | `BeanDefinitonBuilder`            |
  | 单例对象                | API实现                           |

* Spring 内建`BeanDefinition`

  | Bean名称                                                     | Bean实例                               | 使用场景                                       |
  | ------------------------------------------------------------ | -------------------------------------- | ---------------------------------------------- |
  | org.springframework.context.<br />annotation.internalConfigurationAnnotationProcessor | `ConfigurationClassPostProcessor`      | 处理Spring配置                                 |
  | org.springframework.context.annotation.<br />internalAutowiredAnnotationProcessor | `AutowiredAnnotationBeanPostProcessor` | 处理`@Autowired`以及`@Value`                   |
  | org.springframework.context.annotation.<br />internalCommonAnnotationProcessor | `CommonAnnotationBeanPostProcessor`    | (条件激活)处理JSR-250注解,如`@PostConstructor` |
  | org.springframework.context.event.<br />internalEventListenerProcessor | `EventListenerMethodProcessor`         | 处理标注`@EventListener`的Spring事件监听方法   |

  ```java
  	// org.springframework.context.annotation.AnnotationConfigUtils
  	/**
  	 * Register all relevant annotation post processors in the given registry.
  	 * @param registry the registry to operate on
  	 * @param source the configuration source element (already extracted)
  	 * that this registration was triggered from. May be {@code null}.
  	 * @return a Set of BeanDefinitionHolders, containing all bean definitions
  	 * that have actually been registered by this call
  	 */
  	public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
  			BeanDefinitionRegistry registry, @Nullable Object source) {
  
  		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
  		if (beanFactory != null) {
  			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
  				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
  			}
  			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
  				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
  			}
  		}
  
  		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
  		// 处理Spring配置
  		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
  		}
  		// 处理`@Autowired`以及`@Value`
  		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
  		}
  		// (条件激活)处理JSR-250注解,如`@PostConstructor`
  		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
  		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
  		}
  
  		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
  		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition();
  			try {
  				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
  						AnnotationConfigUtils.class.getClassLoader()));
  			}
  			catch (ClassNotFoundException ex) {
  				throw new IllegalStateException(
  						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
  			}
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
  		}
  		// 处理标注`@EventListener`的Spring事件监听方法
  		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
  		}
  
  		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
  			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
  			def.setSource(source);
  			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
  		}
  
  		return beanDefs;
  	}
  ```

* Spring内建单例对象

  | Bean名称                      | Bean实例                                      | 使用场景               |
  | ----------------------------- | --------------------------------------------- | ---------------------- |
  | `environment`                 | `Environment`对象                             | 外部化配置以及Profiles |
  | `systemProperties`            | `Map<String, Object> getSystemProperties();`  | Java系统属性           |
  | `systemEnvironment`           | `Map<String, Object> getSystemEnvironment();` | 操作系统环境变量       |
  | `messageSource`               | `MessageSource`对象                           | 国际化文案             |
  | `applicationEventMulticaster` | `ApplicationEventMulticaster`对象             | Spring事件广播器       |
  | `lifecycleProcessor`          | `LifecycleProcessor`对象                      | `Lifecycle Bean`处理器 |

  ```java
  	/**
  	 * Configure the factory's standard context characteristics,
  	 * such as the context's ClassLoader and post-processors.
  	 * @param beanFactory the BeanFactory to configure
  	 */
  	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  		// ...省略
  
  		// Register default environment beans.
  		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
  			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
  		}
  		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
  			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
  		}
  		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
  			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
  		}
  	}
  
  ```

  ```java
  	/**
  	 * Initialize the MessageSource.
  	 * Use parent's if none defined in this context.
  	 */
  	protected void initMessageSource() {
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
  			this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
  			// Make MessageSource aware of parent MessageSource.
  			if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
  				HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
  				if (hms.getParentMessageSource() == null) {
  					// Only set parent context as parent MessageSource if no parent MessageSource
  					// registered already.
  					hms.setParentMessageSource(getInternalParentMessageSource());
  				}
  			}
  			if (logger.isTraceEnabled()) {
  				logger.trace("Using MessageSource [" + this.messageSource + "]");
  			}
  		}
  		else {
  			// Use empty MessageSource to be able to accept getMessage calls.
  			DelegatingMessageSource dms = new DelegatingMessageSource();
  			dms.setParentMessageSource(getInternalParentMessageSource());
  			this.messageSource = dms;
  			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
  			if (logger.isTraceEnabled()) {
  				logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
  			}
  		}
  	}
  ```

  ```java
  	/**
  	 * Initialize the ApplicationEventMulticaster.
  	 * Uses SimpleApplicationEventMulticaster if none defined in the context.
  	 * @see org.springframework.context.event.SimpleApplicationEventMulticaster
  	 */
  	protected void initApplicationEventMulticaster() {
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
  			this.applicationEventMulticaster =
  					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
  			if (logger.isTraceEnabled()) {
  				logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
  			}
  		}
  		else {
  			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
  			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
  			if (logger.isTraceEnabled()) {
  				logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
  						"[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
  			}
  		}
  	}
  
  ```

  ```java
  	/**
  	 * Initialize the LifecycleProcessor.
  	 * Uses DefaultLifecycleProcessor if none defined in the context.
  	 * @see org.springframework.context.support.DefaultLifecycleProcessor
  	 */
  	protected void initLifecycleProcessor() {
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (beanFactory.containsLocalBean(LIFECYCLE_PROCESSOR_BEAN_NAME)) {
  			this.lifecycleProcessor =
  					beanFactory.getBean(LIFECYCLE_PROCESSOR_BEAN_NAME, LifecycleProcessor.class);
  			if (logger.isTraceEnabled()) {
  				logger.trace("Using LifecycleProcessor [" + this.lifecycleProcessor + "]");
  			}
  		}
  		else {
  			DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
  			defaultProcessor.setBeanFactory(beanFactory);
  			this.lifecycleProcessor = defaultProcessor;
  			beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);
  			if (logger.isTraceEnabled()) {
  				logger.trace("No '" + LIFECYCLE_PROCESSOR_BEAN_NAME + "' bean, using " +
  						"[" + this.lifecycleProcessor.getClass().getSimpleName() + "]");
  			}
  		}
  	}
  ```

#### 依赖注入来源

* 注入来源

  | 来源                    | 配置元数据                        |
  | ----------------------- | --------------------------------- |
  | Spring `BeanDefinition` | \<bean id="user" class="...User"> |
  |                         | @Bean public User user(){..}      |
  |                         | `BeanDefinitonBuilder`            |
  | 单例对象                | API实现                           |
  | 非Spring容器管理对象    |                                   |

#### Spring容器管理和游离对象

* 依赖对象

  | 来源                    | Spring Bean对象 | 生命周期管理 | 配置元信息 | 使用场景          |
  | ----------------------- | --------------- | ------------ | ---------- | ----------------- |
  | Spring `BeanDefinition` | 是              | 是           | 有         | 依赖查找,依赖注入 |
  | 单体对象                | 是              | 否           | 无         | 依赖查找,依赖注入 |
  | `ResolvableDependency`  | 否              | 否           | 无         | 依赖注入          |

#### Spring `BeanDefinition`作为依赖来源

* 要素
  * 元数据: `BeanDefinition`
  * 注册: `BeanDefinitionRegistry#registerBeanDefintion`
  * 类型: 延迟和非延迟
  * 顺序: Bean生命周期顺序按照注册顺序

#### 单例对象作为依赖来源

* 要素:
  * 来源: 外部普通对象Java对象(不一定是`POJO`)
  * 注册: `SingletonBeanRegistry#registerSingleton`
* 限制
  * 无生命周期管理
  * 无法实现延迟初始化Bean

#### 非Spring容器管理对象作为依赖来源

* 要素
  * `ConfigurableListableBeanFactory#registerResolvableDependency  `
* 限制
  * 无生命周期管理
  * 无法实现延迟初始化Bean
  * 无法通过依赖查找

#### 外部配置作为依赖来源

* 要素
  * 类型: 非常规Spring对象依赖来源
* 限制
  * 无生命周期管理
  * 无法实现延迟初始化Bean
  * 无法通过依赖查找
