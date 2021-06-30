---
title: Spring(十九)-ApplicationContext Lifecycle应用上下文生命周期
mermaid: true
date: 2021-06-27 21:06:04
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- ApplicationContext Lifecycle
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

#### Spring应用上下文准备阶段

* `org.springframework.context.ConfigurableApplicationContext#refresh`

* `org.springframework.context.support.AbstractApplicationContext#prepareRefresh`方法
  * 启动时间: `startupDate`
  * 状态标识: `closed(false)`,`active(true)`
  * 初始化:`PropertySources`  `initPropertySources()`
  * 校验`Environment`中必须属性
  * 初始化事件监听器集合
  * 初始化早期Spring事件集合

#### `BeanFactory` 创建阶段

* `org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory`方法
  * 刷新Spring应用上下文底层`BeanFactory `: `refreshBeanFactory()`
    * 销毁或关闭`BeanFactory`: 如果已存在的话
    * 创建`BeanFactory`: `createBeanFactory()`
    * 设置`BeanFactory` id
    * 设置"是否允许`BeanDefinition` 重复定义": `customizeBeanFactory(DefaultListableBeanFactory beanFactory)`
    * 设置"是否允许循环引用(依赖)": `customizeBeanFactory(DefaultListableBeanFactory beanFactory)`
    * 加载`BeanDefinition`: `loadBeanDefinitions(DefaultListableBeanFactory beanFactory)`方法
    * 关联新建`BeanFactory`到Spring应用上下文
  * 返回Spring应用上下文底层`BeanFactory`: `getBeanFactory()`

```java
	/**
	 * Tell the subclass to refresh the internal bean factory.
	 * @return the fresh BeanFactory instance
	 * @see #refreshBeanFactory()
	 * @see #getBeanFactory()
	 */
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		// 刷新Spring应用上下文底层BeanFactory
		refreshBeanFactory();
		return getBeanFactory();
	}
		/**
	 * This implementation performs an actual refresh of this context's underlying
	 * bean factory, shutting down the previous bean factory (if any) and
	 * initializing a fresh bean factory for the next phase of the context's lifecycle.
	 */
	@Override
	protected final void refreshBeanFactory() throws BeansException {
		// 如果存在的话
		if (hasBeanFactory()) {
			// 销毁或关闭BeanFactory
			destroyBeans();
			closeBeanFactory();
		}
		try {
			// 创建BeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			// 设置BeanFactory Id
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			// 加载BeanDefinition
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				// 关联新建BeanFactory到Spring应用上下文
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
		/**
	 * Customize the internal bean factory used by this context.
	 * Called for each {@link #refresh()} attempt.
	 * <p>The default implementation applies this context's
	 * {@linkplain #setAllowBeanDefinitionOverriding "allowBeanDefinitionOverriding"}
	 * and {@linkplain #setAllowCircularReferences "allowCircularReferences"} settings,
	 * if specified. Can be overridden in subclasses to customize any of
	 * {@link DefaultListableBeanFactory}'s settings.
	 * @param beanFactory the newly created bean factory for this context
	 * @see DefaultListableBeanFactory#setAllowBeanDefinitionOverriding
	 * @see DefaultListableBeanFactory#setAllowCircularReferences
	 * @see DefaultListableBeanFactory#setAllowRawInjectionDespiteWrapping
	 * @see DefaultListableBeanFactory#setAllowEagerClassLoading
	 */
	protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
		// 设置是否允许BeanDefinition重复定义
		if (this.allowBeanDefinitionOverriding != null) {
			beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// 设置是否允许循环引用(依赖)
		if (this.allowCircularReferences != null) {
			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
		}
	}
```

#### `BeanFactory`准备阶段

* `org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory`方法
  * 设置`ClassLoader`
  * 设置Bean表达式处理器
  * 添加`PropertyEditorRegistrar`实现: `ResourceEditorRegistrar`
  * 添加`Aware`回调接口`BeanPostProcessor`: `ApplicationContextAwareProcessor`
  * 忽略`Aware`回调接口作为依赖注入接口
  * 注册`ResolvableDependency`对象: `BeanFactory`,`ResourceLoader`,`ApplicationEventPublisher`,`ApplicationContext`
  * 注册`ApplicationListenerDetector`对象
  * 注册`LoadTimeWeaverAwareProcessor`对象
  * 注册单例对象: `Environment`,`Java System Properties`以及`OS`环境变量

```java
	/**
	 * Configure the factory's standard context characteristics,
	 * such as the context's ClassLoader and post-processors.
	 * @param beanFactory the BeanFactory to configure
	 */
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
        // 设置`ClassLoader`
		beanFactory.setBeanClassLoader(getClassLoader());
        // 设置Bean表达式处理器
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
        // 添加`PropertyEditorRegistrar`实现: `ResourceEditorRegistrar`
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
        // 添加`Aware`回调接口`BeanPostProcessor`: `ApplicationContextAwareProcessor`
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        // 忽略`Aware`回调接口作为依赖注入接口
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
        // 注册`ResolvableDependency`对象: `BeanFactory`,`ResourceLoader`,`ApplicationEventPublisher`,`ApplicationContext`
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
        // 注册`ApplicationListenerDetector`对象
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
        // 注册`LoadTimeWeaverAwareProcessor`对象
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
        // 注册单例对象: `Environment`,`Java System Properties`以及`OS`环境变量
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
####  `BeanFactory`后置处理阶段

* `org.springframework.context.support.AbstractApplicationContext#postProcessBeanFactory`方法
  * 由子类覆盖该方法
* `org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors`方法
  * 调用`BeanFactoryPostProcessor`或`BeanDefinitionRegistry`后置处理方法
  * 注册`LoadTimeWeaverAwareProcessor`对象

```java
	/**
	 * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
	 * respecting explicit order if given.
	 * <p>Must be called before singleton instantiation.
	 */
	protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

		// Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
		// (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
	}
		public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		Set<String> processedBeans = new HashSet<>();

		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
				else {
					regularPostProcessors.add(postProcessor);
				}
			}

			// Do not initialize FactoryBeans here: We need to leave all regular beans
			// uninitialized to let the bean factory post-processors apply to them!
			// Separate between BeanDefinitionRegistryPostProcessors that implement
			// PriorityOrdered, Ordered, and the rest.
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

			// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			// Now, invoke the postProcessBeanFactory callback of all processors handled so far.
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// Finally, invoke all other BeanFactoryPostProcessors.
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}

```

#### `BeanFactory`注册`BeanFactoryPostProcessor`阶段

* `org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors`方法
  * 注册`PriorityOrdered`类型的`BeanPostProcessor Beans`
  * 注册`Ordered`类型的`BeanPostProcessor Beans`
  * 注册普通`BeanPostProcessor Beans`
  * 注册`MergedBeanDefinitionPostProcessor Beans`
  * 注册`ApplicationListenerDetector`对象

```java
	public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

		// Register BeanPostProcessorChecker that logs an info message when
		// a bean is created during BeanPostProcessor instantiation, i.e. when
		// a bean is not eligible for getting processed by all BeanPostProcessors.
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

		// Separate between BeanPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, register the BeanPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

		// Next, register the BeanPostProcessors that implement Ordered.
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// Now, register all regular BeanPostProcessors.
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// Finally, re-register all internal BeanPostProcessors.
		sortPostProcessors(internalPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// Re-register post-processor for detecting inner beans as ApplicationListeners,
		// moving it to the end of the processor chain (for picking up proxies etc).
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
	}
```

#### 初始化内建Bean: `MessageSource`

* `org.springframework.context.support.AbstractApplicationContext#initMessageSource`方法

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

#### 初始化内建Bean: Spring事件广播器

* `org.springframework.context.support.AbstractApplicationContext#initApplicationEventMulticaster`方法

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

#### Spring应用上下文刷新阶段

* `org.springframework.context.support.AbstractApplicationContext#onRefresh`方法
  * 子类覆盖该方法
    * `org.springframework.web.context.support.AbstractRefreshableWebApplicationContext#onRefresh()`
    * `org.springframework.web.context.support.GenericWebApplicationContext#onRefresh()`
    * `org.springframework.boot.web.reactive.context.ReactiveWebServerApplicationContext#onRefresh()`
    * `org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#onRefresh()`
    * `org.springframework.web.context.support.StaticWebApplicationContext#onRefresh()`

#### Spring事件监听器注册阶段

* `org.springframework.context.support.AbstractApplicationContext#registerListeners`方法
  * 添加当前应用上下文所关联的`ApplicationListeners`对象集合
  * 添加`BeanFactory`所注册`ApplicationListener Beans`
  * 广播早期Spring事件

```java
	/**
	 * Add beans that implement ApplicationListener as listeners.
	 * Doesn't affect other listeners, which can be added without being beans.
	 */
	protected void registerListeners() {
		// Register statically specified listeners first.
		for (ApplicationListener<?> listener : getApplicationListeners()) {
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}

		// Publish early application events now that we finally have a multicaster...
		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
		this.earlyApplicationEvents = null;
		if (earlyEventsToProcess != null) {
			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
				getApplicationEventMulticaster().multicastEvent(earlyEvent);
			}
		}
	}
```

#### `BeanFactory`初始化完成阶段

* `org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization`方法

  * `BeanFactory`关联`ConversionService Bean`,如果存在

    ```java
    		// Initialize conversion service for this context.
    		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
    				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
    			beanFactory.setConversionService(
    					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    		}
    ```

  * 添加`StringValueResolver`对象

    ```java
    		// Register a default embedded value resolver if no bean post-processor
    		// (such as a PropertyPlaceholderConfigurer bean) registered any before:
    		// at this point, primarily for resolution in annotation attribute values.
    		if (!beanFactory.hasEmbeddedValueResolver()) {
    			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    		}
    ```

  * 依赖查找`LoadTimeWeaverAware Bean`

    ```java
    		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    		for (String weaverAwareName : weaverAwareNames) {
    			getBean(weaverAwareName);
    		}
    ```

  * `BeanFactory`临时`ClassLoader`置为`null`

    ```java
    		// Stop using the temporary ClassLoader for type matching.
    		beanFactory.setTempClassLoader(null);
    ```

  * `BeanFactory`冻结配置

    ```java
    		// Allow for caching all bean definition metadata, not expecting further changes.
    		beanFactory.freezeConfiguration();
    ```

  * `BeanFactory`初始化非延迟单例`Beans`

    ```java
    		// Instantiate all remaining (non-lazy-init) singletons.
    		beanFactory.preInstantiateSingletons();
    ```

#### Spring应用上下文刷新完成阶段

* `org.springframework.context.support.AbstractApplicationContext#finishRefresh`方法
  * 清除`ResourceLoader`缓存: `org.springframework.core.io.DefaultResourceLoader#clearResourceCaches` `@since 5.0`
  * 初始化`LifecycleProcessor`对象: `org.springframework.context.support.AbstractApplicationContext#initLifecycleProcessor`
  * 调用`org.springframework.context.LifecycleProcessor#onRefresh`方法
  * 发布Spring应用上下文已刷新事件: `ContextRefreshedEvent`
  * 向`MBeanServer`托管`Live Beans`

```java
	/**
	 * Finish the refresh of this context, invoking the LifecycleProcessor's
	 * onRefresh() method and publishing the
	 * {@link org.springframework.context.event.ContextRefreshedEvent}.
	 */
	protected void finishRefresh() {
		// Clear context-level resource caches (such as ASM metadata from scanning).
		clearResourceCaches();

		// Initialize lifecycle processor for this context.
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
		getLifecycleProcessor().onRefresh();

		// Publish the final event.
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		LiveBeansView.registerApplicationContext(this);
	}
```

```java
	/**
	 * Clear all resource caches in this resource loader.
	 * @since 5.0
	 * @see #getResourceCache
	 */
	public void clearResourceCaches() {
		this.resourceCaches.clear();
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

```java
	static void registerApplicationContext(ConfigurableApplicationContext applicationContext) {
		String mbeanDomain = applicationContext.getEnvironment().getProperty(MBEAN_DOMAIN_PROPERTY_NAME);
		if (mbeanDomain != null) {
			synchronized (applicationContexts) {
				if (applicationContexts.isEmpty()) {
					try {
						MBeanServer server = ManagementFactory.getPlatformMBeanServer();
						applicationName = applicationContext.getApplicationName();
						server.registerMBean(new LiveBeansView(),
								new ObjectName(mbeanDomain, MBEAN_APPLICATION_KEY, applicationName));
					}
					catch (Throwable ex) {
						throw new ApplicationContextException("Failed to register LiveBeansView MBean", ex);
					}
				}
				applicationContexts.add(applicationContext);
			}
		}
	}
```

#### Spring应用上下文启动阶段

* `org.springframework.context.support.AbstractApplicationContext#start`方法
  * 启动`LifecycleProcessor`
    * 依赖查找: `Lifecycle Beans`
    * 启动`Lifecycle Beans`
  * 发布Spring应用上下文已启动事件: `ContextStartedEvent`

```java
	@Override
	public void start() {
		getLifecycleProcessor().start();
		publishEvent(new ContextStartedEvent(this));
	}
```

#### Spring应用上下文停止阶段

* `org.springframework.context.support.AbstractApplicationContext#stop`

  * 停止`LifecycleProcessor`
    * 依赖查找: `Lifecycle Beans`
    * 启动`Lifecycle Beans`
  * 发布Spring应用上下文已停止事件: `ContextStoppedEvent`

```java
	@Override
	public void stop() {
		getLifecycleProcessor().stop();
		publishEvent(new ContextStoppedEvent(this));
	}
```

####  Spring应用上下文关闭阶段

* `org.springframework.context.support.AbstractApplicationContext#close`方法

  * 状态标识: `active(false)`,`closed(true)`

  * `Live Beans JMX`撤销托管

    *  `org.springframework.context.support.LiveBeansView#unregisterApplicationContext`

    ```java
    	static void unregisterApplicationContext(ConfigurableApplicationContext applicationContext) {
    		synchronized (applicationContexts) {
    			if (applicationContexts.remove(applicationContext) && applicationContexts.isEmpty()) {
    				try {
    					MBeanServer server = ManagementFactory.getPlatformMBeanServer();
    					String mbeanDomain = applicationContext.getEnvironment().getProperty(MBEAN_DOMAIN_PROPERTY_NAME);
    					if (mbeanDomain != null) {
    						server.unregisterMBean(new ObjectName(mbeanDomain, MBEAN_APPLICATION_KEY, applicationName));
    					}
    				}
    				catch (Throwable ex) {
    					throw new ApplicationContextException("Failed to unregister LiveBeansView MBean", ex);
    				}
    				finally {
    					applicationName = null;
    				}
    			}
    		}
    	}
    ```

  * 发布Spring应用上下文已关闭事件: `ContextClosedEvent`

    ```java
    		try {
    				// Publish shutdown event.
    				publishEvent(new ContextClosedEvent(this));
    			}
    			catch (Throwable ex) {
    				logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
    			}
    ```

  * 关闭`LifecycleProcessor`

    * 依赖查找`Lifecycle Beans`
    * 停止`Lifecycle Beans`

    ```java
    			// Stop all Lifecycle beans, to avoid delays during individual destruction.
    			if (this.lifecycleProcessor != null) {
    				try {
    					this.lifecycleProcessor.onClose();
    				}
    				catch (Throwable ex) {
    					logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
    				}
    			}
    ```

  * 销毁Spring Beans

    ```java
    			// Destroy all cached singletons in the context's BeanFactory.
    			destroyBeans();
    
                /**
                 * Template method for destroying all beans that this context manages.
                 * The default implementation destroy all cached singletons in this context,
                 * invoking {@code DisposableBean.destroy()} and/or the specified
                 * "destroy-method".
                 * <p>Can be overridden to add context-specific bean destruction steps
                 * right before or right after standard singleton destruction,
                 * while the context's BeanFactory is still active.
                 * @see #getBeanFactory()
                 * @see org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons()
                 */
                protected void destroyBeans() {
                    getBeanFactory().destroySingletons();
                }
    ```

  * 关闭`BeanFactory`

    ```java
    			// Close the state of this context itself.
    			closeBeanFactory();
    ```

  * 回调`onClose()`

    ```java
    			// Let subclasses do some final clean-up if they wish...
    			onClose();
    ```

  * 注册`Shutdown Hook`线程(如果曾注册)

    ```java
    			// If we registered a JVM shutdown hook, we don't need it anymore now:
    			// We've already explicitly closed the context.
    			if (this.shutdownHook != null) {
    				try {
    					Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
    				}
    				catch (IllegalStateException ex) {
    					// ignore - VM is already shutting down
    				}
    			}
    ```

```java
	/**
	 * Close this application context, destroying all beans in its bean factory.
	 * <p>Delegates to {@code doClose()} for the actual closing procedure.
	 * Also removes a JVM shutdown hook, if registered, as it's not needed anymore.
	 * @see #doClose()
	 * @see #registerShutdownHook()
	 */
	@Override
	public void close() {
		synchronized (this.startupShutdownMonitor) {
			doClose();
			// If we registered a JVM shutdown hook, we don't need it anymore now:
			// We've already explicitly closed the context.
			if (this.shutdownHook != null) {
				try {
					Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
				}
				catch (IllegalStateException ex) {
					// ignore - VM is already shutting down
				}
			}
		}
	}
```

```java
/**
	 * Actually performs context closing: publishes a ContextClosedEvent and
	 * destroys the singletons in the bean factory of this application context.
	 * <p>Called by both {@code close()} and a JVM shutdown hook, if any.
	 * @see org.springframework.context.event.ContextClosedEvent
	 * @see #destroyBeans()
	 * @see #close()
	 * @see #registerShutdownHook()
	 */
	protected void doClose() {
		// Check whether an actual close attempt is necessary...
		if (this.active.get() && this.closed.compareAndSet(false, true)) {
			if (logger.isDebugEnabled()) {
				logger.debug("Closing " + this);
			}

			LiveBeansView.unregisterApplicationContext(this);

			try {
				// Publish shutdown event.
				publishEvent(new ContextClosedEvent(this));
			}
			catch (Throwable ex) {
				logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
			}

			// Stop all Lifecycle beans, to avoid delays during individual destruction.
			if (this.lifecycleProcessor != null) {
				try {
					this.lifecycleProcessor.onClose();
				}
				catch (Throwable ex) {
					logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
				}
			}

			// Destroy all cached singletons in the context's BeanFactory.
			destroyBeans();

			// Close the state of this context itself.
			closeBeanFactory();

			// Let subclasses do some final clean-up if they wish...
			onClose();

			// Reset local application listeners to pre-refresh state.
			if (this.earlyApplicationListeners != null) {
				this.applicationListeners.clear();
				this.applicationListeners.addAll(this.earlyApplicationListeners);
			}

			// Switch to inactive.
			this.active.set(false);
		}
	}
```

