---
title: Spring源码分析(Spring5.1.x)
date: 2021-05-27 20:55:33
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags: 
- Spring
- Ioc
- 源码
categories: 
- Spring
---

### Spring源码分析(Spring5.1.x)

#### 参考文献

* [《轻松读懂spring》之 IOC的主干流程（上）](https://mp.weixin.qq.com/s/si531-_MeTNajM-q1NFqgQ)

#### 入口

> Spring容器的顶层接口是：`BeanFactory`，但我们使用更多的是它的子接口：`ApplicationContext`。
>
> 通常情况下，如果我们想要手动初始化通过`xml文件`配置的spring容器时，代码是这样的：
>
> ```java
> ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
> User user = (User)applicationContext.getBean("name");
> ```
>
> 如果想要手动初始化通过`配置类`配置的spring容器时，代码是这样的：
>
> ```java
> AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);
> User user = (User)applicationContext.getBean("name");
> ```
>
> 这两个类应该是最常见的入口了，它们却殊途同归，最终都会调用`refresh`方法，该方法才是spring容器初始化的真正入口。

#### refresh方法

> `refresh`方法是`spring ioc`的真正入口，它负责初始化spring容器。
>
> 既然这个方法的作用是初始化spring容器，那方法名为啥不叫`init`？答案很简单，因为它不只被调用一次。
>
> * 在`springboot`的`SpringAppication`类中的`run`方法会调用`refreshContext`方法，该方法会调用一次`refresh`方法。
>
> * 在`springcloud`的`BootstrapApplicationListener`类中的`onApplicationEvent`方法会调用`SpringAppication`类中的`run`方法。也会调用一次`refresh`方法。
>
> > 这是springboot项目中如果引入了springcloud，则refresh方法会被调用两次的原因。
>
> * 在`springmvc`的`FrameworkServlet`类中的`initWebApplicationContext`方法会调用`configureAndRefreshWebApplicationContext`方法，该方法会调用一次`refresh`方法，不过会提前判断容器是否激活。
>
> 所以这里的`refresh`表示重新构建的意思。

#### refresh源码

```java
// 路径: org.springframework.context.support.AbstractApplicationContext
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// 准备刷新的上下文环境,例如对系统属性或者环境变量进行准备及验证
			// Prepare this context for refreshing.
			prepareRefresh();

			// 初始化BeanFactory并进行XML文件读取
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 对BeanFactory进行各种功能填充
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// 子类覆盖方法做额外的处理
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				/**
				 * 激活各种BeanFactory处理器
				 * 对于BeanFactoryPostProcessor的处理要区分两种情况:
				 * 一种方式是通过硬编码方式的处理,另一种是通过配置文件的方式
				 * BeanFaPostProcessor不但要实现注册功能,而且还要实现后处理器的激活操作
 				 */
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// 注册拦截Bean创建的Bean处理器,这里只是注册,真正的调用是getBean的时候
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// 为上下文初始化Message源,即不同语言的消息体,用于国际化处理
				// Initialize message source for this context.
				initMessageSource();

				// 初始化应用消息广播器,并放入'applicationEventMulticaster'bean中
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// 留给子类来初始化其他的Bean
				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// 在所有注册的bean中查找Listener bean,注册到消息广播器中
				// Check for listener beans and register them.
				registerListeners();

				// 初始化剩下的单实例(非惰性的)
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// 完成刷新过程,通知生命周期处理lifecycleProcessor刷新过程,同时发出ContextRefreshEvent通知别人
				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}
		}
	}
```

#### refresh方法中主要流程及操作

##### **prepareRefresh()**

> 准备刷新上下文环境,初始化placeholder属性;

```java
// 路径: org.springframework.context.support.AbstractApplicationContext#prepareRefresh
	protected void prepareRefresh() {
		this.startupDate = System.currentTimeMillis();

		synchronized (this.activeMonitor) {
			this.active = true;
		}

		if (logger.isInfoEnabled()) {
			logger.info("Refreshing " + this);
		}

		// 留给子类覆盖
		// Initialize any placeholder property sources in the context environment
		initPropertySources();

		// 验证需要的属性文件是否都已经放入环境中
		// Validate that all properties marked as required are resolvable
		// see ConfigurablePropertyResolver#setRequiredProperties
		getEnvironment().validateRequiredProperties();
	}
```

##### **obtainFreshBeanFactory()**

> 初始化BeanFactory并进行XML文件读取,生成注册BeanDefinition;

```java
// 路径: org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		// 初始化BeanFactory,并进行XML文件读取,并将得到的BeanFactory记录在当前实体的属性中
		refreshBeanFactory();
		// 返回当前实体的BeanFactory属性
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
```

```java
// 路径: org.springframework.context.support.AbstractRefreshableApplicationContext#refreshBeanFactory
  	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			// 创建DefaultListableBeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			// 为了序列化指定id,如果需要的话,让整个BeanFactory从id反序列化到BeanFactory对象
			beanFactory.setSerializationId(getId());
			// 定制BeanFactory,设置相关属性,包括是否允许覆盖同名称的不同定义的对象,循环依赖,CandidateResolver
			customizeBeanFactory(beanFactory);
			// 初始化DocumentReader,并进行XML文件读取及解析
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

```java
// 路径: org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions
	try {
			int validationMode = getValidationModeForResource(resource);
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
```

```java
// 路径: org.springframework.beans.factory.xml.XmlBeanDefinitionReader#registerBeanDefinitions
	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		// 使用DefaultBeanDefinitionDocument实例化BeanDefinitionDocumentReader
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		// 将环境变量设置其中
		documentReader.setEnvironment(getEnvironment());
		// 在实例化BeanDefinitionReader时候会将BeanDefinitionRegistry传入,默认使用继承自DefaultListableBeanFactory
		// 记录统计前BeanDefinition的加载个数
		int countBefore = getRegistry().getBeanDefinitionCount();
		// 加载及注册bean
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		// 记录本次加载的BeanDefinition个数
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

```java
// 路径: org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions
	protected void doRegisterBeanDefinitions(Element root) {
		// 处理profile属性
		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
		if (StringUtils.hasText(profileSpec)) {
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
					profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			if (!getEnvironment().acceptsProfiles(specifiedProfiles)) {
				return;
			}
		}

		// Any nested <beans> elements will cause recursion in this method. In
		// order to propagate and preserve <beans> default-* attributes correctly,
		// keep track of the current (parent) delegate, which may be null. Create
		// the new (child) delegate with a reference to the parent for fallback purposes,
		// then ultimately reset this.delegate back to its original (parent) reference.
		// this behavior emulates a stack of delegates without actually necessitating one.
		// 专门处理解析
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(this.readerContext, root, parent);

		// 解析前处理,留给子类实现
		preProcessXml(root);
		// 入口
		parseBeanDefinitions(root, this.delegate);
		// 解析后处理,留给子类实现
		postProcessXml(root);

		this.delegate = parent;
	}
```

```java
// 路径: org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseBeanDefinitions
	/**
	 * Parse the elements at the root level in the document:
	 * "import", "alias", "bean".
	 * @param root the DOM root element of the document
	 */
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						// 解析默认标签 `<import/> <alias/> <bean/><beans/>
						parseDefaultElement(ele, delegate);
					}
					else {
						// 解析自定义标签<aop/>、<context/>、<mvc/>
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			// 解析自定义标签
			delegate.parseCustomElement(root);
		}
	}
```

```java
// 路径: org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseDefaultElement
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		// 对import标签进行处理
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		// 对alias标签进行处理
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		// 对bean标签进行处理
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		// 对beans标签进行处理
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// 需要进行递归
            // recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```

###### processBeanDefinition

```java
// 路径: org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processBeanDefinition
	/**
	 * Process the given bean element, parsing the bean definition
	 * and registering it with the registry.
	 */
	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		// 对bean的解析分为两种类型,一种是默认类型的解析,另一种是自定义类型的解析
        // 解析元素生成BeanDefinition
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
                // 注册BeanDefinition
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
            // 发送注册事件
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}

```

###### parseBeanDefinitionElement

> 生成BeanDefinition

```java
// 路径: org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element)
	/**
	 * Parses the supplied {@code &lt;bean&gt;} element. May return {@code null}
	 * if there were errors during parse. Errors are reported to the
	 * {@link org.springframework.beans.factory.parsing.ProblemReporter}.
	 */
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
	}
// 路径: org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinition)
	/**
	 * Parses the supplied {@code &lt;bean&gt;} element. May return {@code null}
	 * if there were errors during parse. Errors are reported to the
	 * {@link org.springframework.beans.factory.parsing.ProblemReporter}.
	 */
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
		// 解析id属性
		String id = ele.getAttribute(ID_ATTRIBUTE);
		// 解析name属性
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
		// 分割name属性
		List<String> aliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}
		// beanName赋值id
		String beanName = id;
        // 判断beanName是否为空
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
            // 将第一个别名赋值给beanName
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
            // 检查名称的唯一性
			checkNameUniqueness(beanName, aliases, ele);
		}
		// 解析元素,生成BeanDefinition
		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
            // 对beanName为空,进行处理
			if (!StringUtils.hasText(beanName)) {
				try {
					// 如果不存在beanName,则更具Spring中提供的命名规则为当前bean生成对应的beanName
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
            // 返回BeanDefinitionHolder
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}
```

```java
// 路径: org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, java.lang.String, org.springframework.beans.factory.config.BeanDefinition)
	/**
	 * Parse the bean definition itself, without regard to name or aliases. May return
	 * {@code null} if problems occurred during the parsing of the bean definition.
	 */
	public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, BeanDefinition containingBean) {

		this.parseState.push(new BeanEntry(beanName));

		String className = null;
		// 解析class属性
		if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
			className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
		}

		try {
			String parent = null;
			// 解析parent属性
			if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
				parent = ele.getAttribute(PARENT_ATTRIBUTE);
			}
			// 创建用于承载属性的AbstractBeanDefinition类型的GenericBeanDefinition
			AbstractBeanDefinition bd = createBeanDefinition(className, parent);
			// 硬编码解析默认的bean的各种属性
            // 解析常见属性,如scope,singleton,abstract,lazy-init(懒加载标签),Autowire
			parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
			// 提取description
			bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
			// 解析MetaElement元数据,填充BeanDefinition
			parseMetaElements(ele, bd);
			// 解析lookup-method属性
			// 获取器注入是一种特殊的方法注入,它是把一个方法声明为返回某种类型的bean,但是实际要返回的bean是配置文件里面配置的.此方法可用在设计有些可插拔的功能上,解除程序依赖.
			parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
			// 解析replaced-method属性
			// 主要是对bean中replaced-method子元素的提取
			// 方法替换:可以在运行是用新的方法替换现有的方法,与之前的look-up不同的是,replaced-method不但可以动态地替换返回实体bean,而且还能动态地更改原有方法的逻辑.
			parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

			// 解析构造函数参数
			parseConstructorArgElements(ele, bd);
			// 解析property子元素
			parsePropertyElements(ele, bd);
			// 解析qualifier子元素
			parseQualifierElements(ele, bd);

			bd.setResource(this.readerContext.getResource());
			bd.setSource(extractSource(ele));

			return bd;
		}
		catch (ClassNotFoundException ex) {
			error("Bean class [" + className + "] not found", ele, ex);
		}
		catch (NoClassDefFoundError err) {
			error("Class that bean class [" + className + "] depends on not found", ele, err);
		}
		catch (Throwable ex) {
			error("Unexpected failure during bean definition parsing", ele, ex);
		}
		finally {
			this.parseState.pop();
		}

		return null;
	}
```

###### registerBeanDefinition

> 注册BeanDefinition

```java
// 路径: org.springframework.beans.factory.support.BeanDefinitionReaderUtils#registerBeanDefinition
	/**
	 * Register the given bean definition with the given bean factory.
	 * @param definitionHolder the bean definition including name and aliases
	 * @param registry the bean factory to register with
	 * @throws BeanDefinitionStoreException if registration failed
	 */
	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

        // 使用beanName做唯一标识注册
		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    	// 注册所有别名
		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
```

```java
// 路径: org.springframework.beans.factory.support.DefaultListableBeanFactory#registerBeanDefinition
	//---------------------------------------------------------------------
	// Implementation of BeanDefinitionRegistry interface
	//---------------------------------------------------------------------

	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				/**
				 * 注册前的最后一次校验,这里的校验不同于之前的XML文件校验
				 * 主要是对于AbstractBeanDefinition属性中的methodOverrides校验
				 * 校验methodOverrides是否与工厂方法并存或者methodOverrides对应的方法根本不存在
				 */
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition oldBeanDefinition;

		// 因为beanDefinitionMap是全局变量,这里定会存在并发访问的情况
		synchronized (this.beanDefinitionMap) {
			oldBeanDefinition = this.beanDefinitionMap.get(beanName);
			// 处理注册已经注册的beanName情况
			if (oldBeanDefinition != null) {
				// 如果对应的BeanName已经注册且在配置中配置了bean不允许被覆盖,则抛出异常
				if (!this.allowBeanDefinitionOverriding) {
					throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
							"Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
							"': There is already [" + oldBeanDefinition + "] bound.");
				}
				else {
					if (this.logger.isInfoEnabled()) {
						this.logger.info("Overriding bean definition for bean '" + beanName +
								"': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");
					}
				}
			}
			else {
				// 记录beanName
				this.beanDefinitionNames.add(beanName);
				this.frozenBeanDefinitionNames = null;
			}
			// 注册beanDefinition
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}

		if (oldBeanDefinition != null || containsSingleton(beanName)) {
			// 重置所有beanName对应的缓存
			resetBeanDefinition(beanName);
		}
	}
```

```java
// 路径: org.springframework.core.SimpleAliasRegistry#registerAlias
	/**
	 * 1) alias与beanName相同情况处理,若alias与beanName名称相同则不需处理并删除原有alias
	 * 2) alias覆盖处理 若aliasName已经使用并已经指向了另一个beanName则需要使用用户的设置进行处理
	 * 3) alias循环检查 当A->B存在时,若再次出现A->C->B时候则会抛出异常
	 * 4) 注册alias
	 * @param name the canonical name
	 * @param alias the alias to be registered
	 */
	public void registerAlias(String name, String alias) {
		Assert.hasText(name, "'name' must not be empty");
		Assert.hasText(alias, "'alias' must not be empty");
		// 如果beanName与alias相同的话不记录并删除对应的alias
		if (alias.equals(name)) {
			this.aliasMap.remove(alias);
		}
		else {
			// 如果alias不允许被覆盖则抛出异常 默认开启
			if (!allowAliasOverriding()) {
				String registeredName = this.aliasMap.get(alias);
				if (registeredName != null && !registeredName.equals(name)) {
					throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" +
							name + "': It is already registered for name '" + registeredName + "'.");
				}
			}
			// 注册前检查是否出现了循环注册，如注册 (a, b) 时已经注册了 (b, a)
			/**
			 * beanName		alias
			 * 1.  a			b
			 * 2.  b			c
			 * 3.  b			a
			 * aliasMap
			 * 1. b->a
			 * 2. c->b
			 * 3. "a".equals(canonicalName("b")) canonicalName("b")-->"a"
			 */
			checkForAliasCircle(name, alias);
			this.aliasMap.put(alias, name);
		}
	}
```

##### **prepareBeanFactory(beanFactory)**

> 对BeanFactory进行各种功能填充,准备BeanFactory容器的参数;

```java
// 路径: org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
	/**
	 * Configure the factory's standard context characteristics,
	 * such as the context's ClassLoader and post-processors.
	 * @param beanFactory the BeanFactory to configure
	 */
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// 设置BeanFactory的classLoader为当前context的classLoader
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());

		// 设置BeanFactory的表达语言(SPEL)处理器,Spring3增加了表达式语言的支持
		// 默认可以使用#{bean.xxx}的形式来调用相关属性值
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver());

		// 为BeanFactory增加一个默认的propertyEditor(属性编辑器),这个主要是对bean的属性等设置管理的一个工具
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// 添加BeanPostProcessor 对一些内置类,比如EnvironmentAware,MessageSourceAware的信息注入
		// Configure the bean factory with context callbacks.
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));

		// 设置了几个忽略自动装配的接口
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// 设置了几个自动装配的特殊规则
		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// 增加对AspectJ的支持
		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// 将相关环境变量以及属性注册以单例模式进行注册
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
// 路径: org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#registerSingleton
	public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
		Assert.notNull(beanName, "'beanName' must not be null");
		synchronized (this.singletonObjects) {
			Object oldObject = this.singletonObjects.get(beanName);
			if (oldObject != null) {
				throw new IllegalStateException("Could not register object [" + singletonObject +
						"] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
			}
			addSingleton(beanName, singletonObject);
		}
	}
```

```java
// 路径: org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingleton
	/**
	 * Add the given singleton object to the singleton cache of this factory.
	 * <p>To be called for eager registration of singletons.
	 * @param beanName the name of the bean
	 * @param singletonObject the singleton object
	 */
	protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
	}
```

##### **postProcessBeanFactory(beanFactory)**

> 钩子方法,添加BeanFactoryPostProcessor,子类覆盖方法可做额外的处理;

##### **invokeBeanFactoryPostProcessors(beanFactory)**

> 激活各种BeanFactoryPostProcessor; 
>
> 可进行修改注册的BeanDefinition对象操作;

```java
// 路径: org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory)
	/**
	 * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
	 * respecting explicit order if given.
	 * <p>Must be called before singleton instantiation.
	 */
	protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		Set<String> processedBeans = new HashSet<String>();
		// 判断BeanFactory是否是BeanDefinitionRegistry类型
		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new LinkedList<BeanFactoryPostProcessor>();
			List<BeanDefinitionRegistryPostProcessor> registryPostProcessors =
					new LinkedList<BeanDefinitionRegistryPostProcessor>();
			// 硬编码注册的后处理器
			for (BeanFactoryPostProcessor postProcessor : getBeanFactoryPostProcessors()) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryPostProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					// 对于BeanDefinitionRegistryPostProcessor类型,在BeanFactoryPostProcessor的基础上还有自己定义的方法,需要先调用
					registryPostProcessor.postProcessBeanDefinitionRegistry(registry);
					registryPostProcessors.add(registryPostProcessor);
				}
				else {
					// 记录常规BeanFactoryPostProcessor
					regularPostProcessors.add(postProcessor);
				}
			}
			// 配置注册的后处理器
			Map<String, BeanDefinitionRegistryPostProcessor> beanMap =
					beanFactory.getBeansOfType(BeanDefinitionRegistryPostProcessor.class, true, false);
			List<BeanDefinitionRegistryPostProcessor> registryPostProcessorBeans =
					new ArrayList<BeanDefinitionRegistryPostProcessor>(beanMap.values());
			OrderComparator.sort(registryPostProcessorBeans);
			for (BeanDefinitionRegistryPostProcessor postProcessor : registryPostProcessorBeans) {
				// BeanDefinitionRegistryPostProcessor的特殊处理
				postProcessor.postProcessBeanDefinitionRegistry(registry);
			}
			// 激活postProcessBeanFactory方法,之前激活的是postProcessBeanDefinitionRegistry
			// 激活硬编码设置的BeanDefinitionRegistryPostProcessor
			invokeBeanFactoryPostProcessors(registryPostProcessors, beanFactory);
			// 激活配置的BeanDefinitionRegistryPostProcessor
			invokeBeanFactoryPostProcessors(registryPostProcessorBeans, beanFactory);
			// 激活常规BeanFactoryPostProcessor
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
			processedBeans.addAll(beanMap.keySet());
		}
		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(getBeanFactoryPostProcessors(), beanFactory);
		}

		// 对于配置中读取的BeanFactoryPostProcessor的处理
		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		List<String> orderedPostProcessorNames = new ArrayList<String>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
		// 对后处理器进行分类
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// 已经处理过的,无需处理
				// skip - already processed in first phase above
			}
			else if (isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 按照优先级进行排序
		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		OrderComparator.sort(priorityOrderedPostProcessors);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		// 按照order排序
		OrderComparator.sort(orderedPostProcessors);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// 最后处理无序BeanPostProcessor
		// Finally, invoke all other BeanFactoryPostProcessors.
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
	}
```

##### **registerBeanPostProcessors(beanFactory)**

>  注册拦截Bean创建的BeanFactoryPostProcessor,这里只是注册,真正的调用是getBean的时候;

```java
	/**
	 * Instantiate and invoke all registered BeanPostProcessor beans,
	 * respecting explicit order if given.
	 * <p>Must be called before any instantiation of application beans.
	 */
	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

		/**
		 * BeanPostProcessorChecker是一个普通的信息打印,可能会有些情况,当Spring的配置中后处理器还没被注册就已经开始bean的初始化时,
		 * 便会打印出BeanPostProcessorChecker
		 */
		// Register BeanPostProcessorChecker that logs an info message when
		// a bean is created during BeanPostProcessor instantiation, i.e. when
		// a bean is not eligible for getting processed by all BeanPostProcessors.
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

		// 使用PriorityOrdered保证顺序
		// Separate between BeanPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
		// MergedBeanDefinitionPostProcessor
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<BeanPostProcessor>();
		// 使用Ordered保证顺序
		List<String> orderedPostProcessorNames = new ArrayList<String>();
		// 无序BeanPostProcessor
		List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
		for (String ppName : postProcessorNames) {
			if (isTypeMatch(ppName, PriorityOrdered.class)) {
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 第一步,注册所有实现PriorityOrdered的BeanPostProcessor
		// First, register the BeanPostProcessors that implement PriorityOrdered.
		OrderComparator.sort(priorityOrderedPostProcessors);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

		// 第二步,注册所有实现Ordered的BeanPostProcessor
		// Next, register the BeanPostProcessors that implement Ordered.
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		OrderComparator.sort(orderedPostProcessors);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// 第三步,注册所有无序的BeanPostProcessor
		// Now, register all regular BeanPostProcessors.
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// 第四部,注册所有MergedBeanDefinitionPostProcessor类型的BeanPostProcessor,并非重复注册
		// 在BeanFactory.addBeanPostProcessor中先移除已经存在的BeanPostProcessor
		// Finally, re-register all internal BeanPostProcessors.
		OrderComparator.sort(internalPostProcessors);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// 添加ApplicationListener探测器
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector());
	}

```

##### **initMessageSource()**

> 为上下文初始化Message源,即不同语言的消息体,用于国际化处理;

```java
	/**
	 * Initialize the MessageSource.
	 * Use parent's if none defined in this context.
	 */
	protected void initMessageSource() {
		// 主要功能是提取配置中定义的messageSource,并将其记录在Spring容器中,也就是AbstractApplicationContext中
		// 若用户没有设置资源文件,Spring中也提供了默认的配置DelegatingMessageSource
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
			if (logger.isDebugEnabled()) {
				logger.debug("Using MessageSource [" + this.messageSource + "]");
			}
		}
		else {
			// Use empty MessageSource to be able to accept getMessage calls.
			DelegatingMessageSource dms = new DelegatingMessageSource();
			dms.setParentMessageSource(getInternalParentMessageSource());
			this.messageSource = dms;
			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
			if (logger.isDebugEnabled()) {
				logger.debug("Unable to locate MessageSource with name '" + MESSAGE_SOURCE_BEAN_NAME +
						"': using default [" + this.messageSource + "]");
			}
		}
	}
```

##### **initApplicationEventMulticaster()**

> 初始化应用消息广播器,并放入'applicationEventMulticaster'bean中;

```java
	/**
	 * Initialize the ApplicationEventMulticaster.
	 * Uses SimpleApplicationEventMulticaster if none defined in the context.
	 * @see org.springframework.context.event.SimpleApplicationEventMulticaster
	 */
	protected void initApplicationEventMulticaster() {
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		// 如果用户自定义了事件广播器,那使用用户自定义的事件广播器
		// 如果用户没有自定义事件广播器,那么使用默认的ApplicationEventMulticaster
		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
			this.applicationEventMulticaster =
					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
			if (logger.isDebugEnabled()) {
				logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
			}
		}
		else {
			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
			if (logger.isDebugEnabled()) {
				logger.debug("Unable to locate ApplicationEventMulticaster with name '" +
						APPLICATION_EVENT_MULTICASTER_BEAN_NAME +
						"': using default [" + this.applicationEventMulticaster + "]");
			}
		}
	}
```

##### **onRefresh()**

> 留给子类来初始化其他的Bean;

##### **registerListeners()**

> 在所有注册的bean中查找Listener bean,注册到消息广播器中;

```java
	/**
	 * Add beans that implement ApplicationListener as listeners.
	 * Doesn't affect other listeners, which can be added without being beans.
	 */
	protected void registerListeners() {
		// 硬编码方式注册的监听器处理
		// Register statically specified listeners first.
		for (ApplicationListener<?> listener : getApplicationListeners()) {
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// 配置文件注册的监听器处理
		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}
	}
```

##### **finishBeanFactoryInitialization(beanFactory)**

>  初始化剩下的单实例(非惰性的);

```java
	/**
	 * Finish the initialization of this context's bean factory,
	 * initializing all remaining singleton beans.
	 */
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// 冻结所有的bean定义,说明注册的bean将不被修改或任何进一步的处理
		// Allow for caching all bean definition metadata, not expecting further changes.
		beanFactory.freezeConfiguration();

		// 初始化剩下的单实例(非惰性的)
		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();
	}
```

##### **finishRefresh()**

> 完成刷新过程,通知生命周期处理lifecycleProcessor刷新过程,同时发出ContextRefreshEvent通知别人;

```java
	/**
	 * Finish the refresh of this context, invoking the LifecycleProcessor's
	 * onRefresh() method and publishing the
	 * {@link org.springframework.context.event.ContextRefreshedEvent}.
	 */
	protected void finishRefresh() {
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



