---
title: SpringBean生命周期
date: 2021-05-30 15:50:33
tags: 
- Bean生命周期
- Java
categories: 
- Spring
---

### SpringBean生命周期

#### 参考文献

* [深究Spring中Bean的生命周期](https://mp.weixin.qq.com/s/GczkZHJ2DdI7cf9g0e6t_w)
* [小马哥讲Spring核心编程思想](https://time.geekbang.org/course/intro/100042601)

#### Bean的完整生命周期

##### Spring Bean元信息配置阶段

> BeanDefinition配置
>
> * 面向资源
>
>   * BeanDefinitionReader 
>
>     ![img](http://www.chenjunlin.vip/img/spring/BeanDefinitionReader类图.png)
>
>     ```java
>     /*
>      * Copyright 2002-2013 the original author or authors.
>      *
>      * Licensed under the Apache License, Version 2.0 (the "License");
>      * you may not use this file except in compliance with the License.
>      * You may obtain a copy of the License at
>      *
>      *      https://www.apache.org/licenses/LICENSE-2.0
>      *
>      * Unless required by applicable law or agreed to in writing, software
>      * distributed under the License is distributed on an "AS IS" BASIS,
>      * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
>      * See the License for the specific language governing permissions and
>      * limitations under the License.
>      */
>         
>     package org.springframework.beans.factory.support;
>         
>     import org.springframework.beans.factory.BeanDefinitionStoreException;
>     import org.springframework.core.io.Resource;
>     import org.springframework.core.io.ResourceLoader;
>         
>     /**
>      * Simple interface for bean definition readers.
>      * Specifies load methods with Resource and String location parameters.
>      *
>      * <p>Concrete bean definition readers can of course add additional
>      * load and register methods for bean definitions, specific to
>      * their bean definition format.
>      *
>      * <p>Note that a bean definition reader does not have to implement
>      * this interface. It only serves as suggestion for bean definition
>      * readers that want to follow standard naming conventions.
>      *
>      * @author Juergen Hoeller
>      * @since 1.1
>      * @see org.springframework.core.io.Resource
>      */
>     public interface BeanDefinitionReader {
>         
>     	/**
>     	 * Return the bean factory to register the bean definitions with.
>     	 * <p>The factory is exposed through the BeanDefinitionRegistry interface,
>     	 * encapsulating the methods that are relevant for bean definition handling.
>     	 */
>     	BeanDefinitionRegistry getRegistry();
>         
>     	/**
>     	 * Return the resource loader to use for resource locations.
>     	 * Can be checked for the <b>ResourcePatternResolver</b> interface and cast
>     	 * accordingly, for loading multiple resources for a given resource pattern.
>     	 * <p>Null suggests that absolute resource loading is not available
>     	 * for this bean definition reader.
>     	 * <p>This is mainly meant to be used for importing further resources
>     	 * from within a bean definition resource, for example via the "import"
>     	 * tag in XML bean definitions. It is recommended, however, to apply
>     	 * such imports relative to the defining resource; only explicit full
>     	 * resource locations will trigger absolute resource loading.
>     	 * <p>There is also a {@code loadBeanDefinitions(String)} method available,
>     	 * for loading bean definitions from a resource location (or location pattern).
>     	 * This is a convenience to avoid explicit ResourceLoader handling.
>     	 * @see #loadBeanDefinitions(String)
>     	 * @see org.springframework.core.io.support.ResourcePatternResolver
>     	 */
>     	ResourceLoader getResourceLoader();
>         
>     	/**
>     	 * Return the class loader to use for bean classes.
>     	 * <p>{@code null} suggests to not load bean classes eagerly
>     	 * but rather to just register bean definitions with class names,
>     	 * with the corresponding Classes to be resolved later (or never).
>     	 */
>     	ClassLoader getBeanClassLoader();
>         
>     	/**
>     	 * Return the BeanNameGenerator to use for anonymous beans
>     	 * (without explicit bean name specified).
>     	 */
>     	BeanNameGenerator getBeanNameGenerator();
>     
>     
>     	/**
>     	 * Load bean definitions from the specified resource.
>     	 * @param resource the resource descriptor
>     	 * @return the number of bean definitions found
>     	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>     	 */
>     	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;
>         
>     	/**
>     	 * Load bean definitions from the specified resources.
>     	 * @param resources the resource descriptors
>     	 * @return the number of bean definitions found
>     	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>     	 */
>     	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;
>         
>     	/**
>     	 * Load bean definitions from the specified resource location.
>     	 * <p>The location can also be a location pattern, provided that the
>     	 * ResourceLoader of this bean definition reader is a ResourcePatternResolver.
>     	 * @param location the resource location, to be loaded with the ResourceLoader
>     	 * (or ResourcePatternResolver) of this bean definition reader
>     	 * @return the number of bean definitions found
>     	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>     	 * @see #getResourceLoader()
>     	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource)
>     	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource[])
>     	 */
>     	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;
>         
>     	/**
>     	 * Load bean definitions from the specified resource locations.
>     	 * @param locations the resource locations, to be loaded with the ResourceLoader
>     	 * (or ResourcePatternResolver) of this bean definition reader
>     	 * @return the number of bean definitions found
>     	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>     	 */
>     	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;
>         
>     }
>     
>     
>     * XML配置: XMLBeanDefinitionReader
>       
>       ```java
>       // org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory
>       // 路径: org.springframework.context.support.AbstractXmlApplicationContext#loadBeanDefinitions(org.springframework.beans.factory.support.DefaultListableBeanFactory)
>       	/**
>       	 * Loads the bean definitions via an XmlBeanDefinitionReader.
>       	 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
>       	 * @see #initBeanDefinitionReader
>       	 * @see #loadBeanDefinitions
>       	 */
>       	@Override
>       	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
>       		//  为指定beanFactory创建XmlBeanDefinitionReader
>       		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
>       		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
>           
>       		// 对BeanDefinitionReader进行环境变量的设置
>       		// Configure the bean definition reader with this context's
>       		// resource loading environment.
>       		beanDefinitionReader.setEnvironment(this.getEnvironment());
>       		beanDefinitionReader.setResourceLoader(this);
>       		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
>           
>       		// 对BeanDefinitionReader进行设置值,可以覆盖
>       		// Allow a subclass to provide custom initialization of the reader,
>       		// then proceed with actually loading the bean definitions.
>       		initBeanDefinitionReader(beanDefinitionReader);
>       		loadBeanDefinitions(beanDefinitionReader);
>       	}
>       	// ==> org.springframework.context.support.AbstractXmlApplicationContext#loadBeanDefinitions(org.springframework.beans.factory.xml.XmlBeanDefinitionReader)
>       	/**
>       	 * Load the bean definitions with the given XmlBeanDefinitionReader.
>       	 * <p>The lifecycle of the bean factory is handled by the {@link #refreshBeanFactory}
>       	 * method; hence this method is just supposed to load and/or register bean definitions.
>       	 * @param reader the XmlBeanDefinitionReader to use
>       	 * @throws BeansException in case of bean registration errors
>       	 * @throws IOException if the required XML document isn't found
>       	 * @see #refreshBeanFactory
>       	 * @see #getConfigLocations
>       	 * @see #getResources
>       	 * @see #getResourcePatternResolver
>       	 */
>       	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
>       		Resource[] configResources = getConfigResources();
>       		if (configResources != null) {
>       			reader.loadBeanDefinitions(configResources);
>       		}
>       		String[] configLocations = getConfigLocations();
>       		if (configLocations != null) {
>       			reader.loadBeanDefinitions(configLocations);
>       		}
>       	}
>       	// ==> org.springframework.beans.factory.support.AbstractBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.Resource...)
>       	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
>       		Assert.notNull(resources, "Resource array must not be null");
>       		int counter = 0;
>       		for (Resource resource : resources) {
>       			counter += loadBeanDefinitions(resource);
>       		}
>       		return counter;
>       	}
>       ```
>       
>     * Properties配置: PropertiesBeanDefinitionReader
>       
>       ```java
>       	/**
>       	 * Load bean definitions from the specified properties file.
>       	 * @param encodedResource the resource descriptor for the properties file,
>       	 * allowing to specify an encoding to use for parsing the file
>       	 * @param prefix a filter within the keys in the map: e.g. 'beans.'
>       	 * (can be empty or {@code null})
>       	 * @return the number of bean definitions found
>       	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>       	 */
>       	public int loadBeanDefinitions(EncodedResource encodedResource, @Nullable String prefix)
>       			throws BeanDefinitionStoreException {
>           
>       		if (logger.isTraceEnabled()) {
>       			logger.trace("Loading properties bean definitions from " + encodedResource);
>       		}
>           
>       		Properties props = new Properties();
>       		try {
>       			try (InputStream is = encodedResource.getResource().getInputStream()) {
>       				if (encodedResource.getEncoding() != null) {
>       					getPropertiesPersister().load(props, new InputStreamReader(is, encodedResource.getEncoding()));
>       				}
>       				else {
>       					getPropertiesPersister().load(props, is);
>       				}
>       			}
>           
>       			int count = registerBeanDefinitions(props, prefix, encodedResource.getResource().getDescription());
>       			if (logger.isDebugEnabled()) {
>       				logger.debug("Loaded " + count + " bean definitions from " + encodedResource);
>       			}
>       			return count;
>       		}
>       		catch (IOException ex) {
>       			throw new BeanDefinitionStoreException("Could not parse properties from " + encodedResource.getResource(), ex);
>       		}
>       	}
>       ```
>   
> * 面向注解
>
>   * @Configuration
>   * @Bean
>   * @Componet
>
> * 面向API: BeanDefinitionBuilder

##### Spring Bean元信息解析阶段

> * 面向资源BeanDefinition解析
>
>   ```java
>   // 路径: org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.Resource)
>   	/**
>   	 * Load bean definitions from the specified XML file.
>   	 * @param resource the resource descriptor for the XML file
>   	 * @return the number of bean definitions found
>   	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>   	 */
>   	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
>   		return loadBeanDefinitions(new EncodedResource(resource));
>   	}
>   // ==> org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.support.EncodedResource)
>   	/**
>   	 * Load bean definitions from the specified XML file.
>   	 * @param encodedResource the resource descriptor for the XML file,
>   	 * allowing to specify an encoding to use for parsing the file
>   	 * @return the number of bean definitions found
>   	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>   	 */
>   	public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
>   		Assert.notNull(encodedResource, "EncodedResource must not be null");
>   		if (logger.isInfoEnabled()) {
>   			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
>   		}
>
>   		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
>   		if (currentResources == null) {
>   			currentResources = new HashSet<EncodedResource>(4);
>   			this.resourcesCurrentlyBeingLoaded.set(currentResources);
>   		}
>   		if (!currentResources.add(encodedResource)) {
>   			throw new BeanDefinitionStoreException(
>   					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
>   		}
>   		try {
>   			InputStream inputStream = encodedResource.getResource().getInputStream();
>   			try {
>   				InputSource inputSource = new InputSource(inputStream);
>   				if (encodedResource.getEncoding() != null) {
>   					inputSource.setEncoding(encodedResource.getEncoding());
>   				}
>                   // 核心方法
>   				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
>   			}
>   			finally {
>   				inputStream.close();
>   			}
>   		}
>   		catch (IOException ex) {
>   			throw new BeanDefinitionStoreException(
>   					"IOException parsing XML document from " + encodedResource.getResource(), ex);
>   		}
>   		finally {
>   			currentResources.remove(encodedResource);
>   			if (currentResources.isEmpty()) {
>   				this.resourcesCurrentlyBeingLoaded.remove();
>   			}
>   		}
>   	}
>   // ==> org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions
>   	/**
>   	 * Actually load bean definitions from the specified XML file.
>   	 * @param inputSource the SAX InputSource to read from
>   	 * @param resource the resource descriptor for the XML file
>   	 * @return the number of bean definitions found
>   	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
>   	 */
>   	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
>   			throws BeanDefinitionStoreException {
>   		try {
>   			int validationMode = getValidationModeForResource(resource);
>   			Document doc = this.documentLoader.loadDocument(
>   					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
>               // 入口
>   			return registerBeanDefinitions(doc, resource);
>   		}
>   		catch (BeanDefinitionStoreException ex) {
>   			throw ex;
>   		}
>   		catch (SAXParseException ex) {
>   			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
>   					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
>   		}
>   		catch (SAXException ex) {
>   			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
>   					"XML document from " + resource + " is invalid", ex);
>   		}
>   		catch (ParserConfigurationException ex) {
>   			throw new BeanDefinitionStoreException(resource.getDescription(),
>   					"Parser configuration exception parsing XML from " + resource, ex);
>   		}
>   		catch (IOException ex) {
>   			throw new BeanDefinitionStoreException(resource.getDescription(),
>   					"IOException parsing XML document from " + resource, ex);
>   		}
>   		catch (Throwable ex) {
>   			throw new BeanDefinitionStoreException(resource.getDescription(),
>   					"Unexpected exception parsing XML document from " + resource, ex);
>   		}
>   	}
>   // ==> org.springframework.beans.factory.xml.XmlBeanDefinitionReader#registerBeanDefinitions
>   	/**
>   	 * Register the bean definitions contained in the given DOM document.
>   	 * Called by {@code loadBeanDefinitions}.
>   	 * <p>Creates a new instance of the parser class and invokes
>   	 * {@code registerBeanDefinitions} on it.
>   	 * @param doc the DOM document
>   	 * @param resource the resource descriptor (for context information)
>   	 * @return the number of bean definitions found
>   	 * @throws BeanDefinitionStoreException in case of parsing errors
>   	 * @see #loadBeanDefinitions
>   	 * @see #setDocumentReaderClass
>   	 * @see BeanDefinitionDocumentReader#registerBeanDefinitions
>   	 */
>   	@SuppressWarnings("deprecation")
>   	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
>   		// 使用DefaultBeanDefinitionDocument实例化BeanDefinitionDocumentReader
>   		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
>   		// 将环境变量设置其中
>   		documentReader.setEnvironment(getEnvironment());
>   		// 在实例化BeanDefinitionReader时候会将BeanDefinitionRegistry传入,默认使用继承自DefaultListableBeanFactory
>   		// 记录统计前BeanDefinition的加载个数
>   		int countBefore = getRegistry().getBeanDefinitionCount();
>   		// 加载及注册bean
>   		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
>   		// 记录本次加载的BeanDefinition个数
>   		return getRegistry().getBeanDefinitionCount() - countBefore;
>   	}
>   ```
>
>   ```java
>   // ==> org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#registerBeanDefinitions
>   	/**
>   	 * This implementation parses bean definitions according to the "spring-beans" XSD
>   	 * (or DTD, historically).
>   	 * <p>Opens a DOM Document; then initializes the default settings
>   	 * specified at the {@code <beans/>} level; then parses the contained bean definitions.
>   	 */
>   	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
>   		this.readerContext = readerContext;
>   		logger.debug("Loading bean definitions");
>   		Element root = doc.getDocumentElement();
>   		doRegisterBeanDefinitions(root);
>   	}
>   // ==> 
>   	/**
>   	 * Register each bean definition within the given root {@code <beans/>} element.
>   	 */
>   	protected void doRegisterBeanDefinitions(Element root) {
>   		// 处理profile属性
>   		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
>   		if (StringUtils.hasText(profileSpec)) {
>   			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
>   					profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
>   			if (!getEnvironment().acceptsProfiles(specifiedProfiles)) {
>   				return;
>   			}
>   		}
>
>   		// Any nested <beans> elements will cause recursion in this method. In
>   		// order to propagate and preserve <beans> default-* attributes correctly,
>   		// keep track of the current (parent) delegate, which may be null. Create
>   		// the new (child) delegate with a reference to the parent for fallback purposes,
>   		// then ultimately reset this.delegate back to its original (parent) reference.
>   		// this behavior emulates a stack of delegates without actually necessitating one.
>   		// 专门处理解析
>   		BeanDefinitionParserDelegate parent = this.delegate;
>   		this.delegate = createDelegate(this.readerContext, root, parent);
>
>   		// 解析前处理,留给子类实现
>   		preProcessXml(root);
>   		// 入口
>   		parseBeanDefinitions(root, this.delegate);
>   		// 解析后处理,留给子类实现
>   		postProcessXml(root);
>
>   		this.delegate = parent;
>   	}
>   // org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseBeanDefinitions
>   	/**
>   	 * Parse the elements at the root level in the document:
>   	 * "import", "alias", "bean".
>   	 * @param root the DOM root element of the document
>   	 */
>   	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
>   		if (delegate.isDefaultNamespace(root)) {
>   			NodeList nl = root.getChildNodes();
>   			for (int i = 0; i < nl.getLength(); i++) {
>   				Node node = nl.item(i);
>   				if (node instanceof Element) {
>   					Element ele = (Element) node;
>   					if (delegate.isDefaultNamespace(ele)) {
>   						// 解析默认标签
>   						parseDefaultElement(ele, delegate);
>   					}
>   					else {
>   						// 解析自定义标签
>   						delegate.parseCustomElement(ele);
>   					}
>   				}
>   			}
>   		}
>   		else {
>   			// 解析自定义标签
>   			delegate.parseCustomElement(root);
>   		}
>   	}
>   // org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseCustomElement(org.w3c.dom.Element)
>   	public BeanDefinition parseCustomElement(Element ele) {
>   		return parseCustomElement(ele, null);
>   	}
>
>   	/**
>   	 * 1. 创建一个需要扩展的组件
>   	 * 2. 定义一个XSD文件描述组件内容
>   	 * 3. 创建一个文件,实现BeanDefinitionParser接口,用来解析XSD文件中定义和组件定义
>   	 * 4. 创建一个Handler文件,扩展自NamespaceHandlerSupport,目的是将组件注册到Spring容器
>   	 * 5. 编写Spring.handlers和Spring.schemas文件
>   	 * @param containingBd 父类bean,对顶层元素的解析设置为null
>   	 */
>   	public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
>   		// 获取对应命名空间
>   		String namespaceUri = getNamespaceURI(ele);
>   		// 根据命名空间找到对应的NamespaceHandler
>   		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
>   		if (handler == null) {
>   			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
>   			return null;
>   		}
>   		// 调用自定义的NamespaceHandler进行解析
>   		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
>   	}
>   //  org.springframework.beans.factory.xml.NamespaceHandlerSupport#parse
>   	/**
>   	 * Parses the supplied {@link Element} by delegating to the {@link BeanDefinitionParser} that is
>   	 * registered for that {@link Element}.
>   	 */
>   	public BeanDefinition parse(Element element, ParserContext parserContext) {
>   		// 寻找解析器并进行解析操作 BeanDefinitionParser#parse
>   		return findParserForElement(element, parserContext).parse(element, parserContext);
>   	}
>
>   	/**
>   	 * Locates the {@link BeanDefinitionParser} from the register implementations using
>   	 * the local name of the supplied {@link Element}.
>   	 */
>   	private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
>   		// 获取元素名称,也就是<myname:user中的user,若在示例中,此时localName为user
>   		String localName = parserContext.getDelegate().getLocalName(element);
>   		// 根据user找到对应的解析器,也就是在
>   		// registerBeanDefinitionParser("user",new UserBeanDefinitionParser());
>   		// 注册的解析器
>   		BeanDefinitionParser parser = this.parsers.get(localName);
>   		if (parser == null) {
>   			parserContext.getReaderContext().fatal(
>   					"Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
>   		}
>   		return parser;
>   	}
>   ```
>
>   * XML解析器: **BeanDefinitionParser**
>
>     ![img](http://www.chenjunlin.vip/img/spring/BeanDefinitionParser类图.png)
>
>     ```java
>     /**
>      * Interface used by the {@link DefaultBeanDefinitionDocumentReader} to handle custom,
>      * top-level (directly under {@code <beans/>}) tags.
>      *
>      * <p>Implementations are free to turn the metadata in the custom tag into as many
>      * {@link BeanDefinition BeanDefinitions} as required.
>      *
>      * <p>The parser locates a {@link BeanDefinitionParser} from the associated
>      * {@link NamespaceHandler} for the namespace in which the custom tag resides.
>      *
>      * @author Rob Harrop
>      * @since 2.0
>      * @see NamespaceHandler
>      * @see AbstractBeanDefinitionParser
>      */
>     public interface BeanDefinitionParser {
>     
>     	/**
>     	 * Parse the specified {@link Element} and register the resulting
>     	 * {@link BeanDefinition BeanDefinition(s)} with the
>     	 * {@link org.springframework.beans.factory.xml.ParserContext#getRegistry() BeanDefinitionRegistry}
>     	 * embedded in the supplied {@link ParserContext}.
>     	 * <p>Implementations must return the primary {@link BeanDefinition} that results
>     	 * from the parse if they will ever be used in a nested fashion (for example as
>     	 * an inner tag in a {@code <property/>} tag). Implementations may return
>     	 * {@code null} if they will <strong>not</strong> be used in a nested fashion.
>     	 * @param element the element that is to be parsed into one or more {@link BeanDefinition BeanDefinitions}
>     	 * @param parserContext the object encapsulating the current state of the parsing process;
>     	 * provides access to a {@link org.springframework.beans.factory.support.BeanDefinitionRegistry}
>     	 * @return the primary {@link BeanDefinition}
>     	 */
>     	@Nullable
>     	BeanDefinition parse(Element element, ParserContext parserContext);
>     
>     }
>     ==> 实现类
>     public abstract class AbstractBeanDefinitionParser implements BeanDefinitionParser {
>     	...
>     	@Override
>     	@Nullable
>     	public final BeanDefinition parse(Element element, ParserContext parserContext) {
>     		AbstractBeanDefinition definition = parseInternal(element, parserContext);
>     		if (definition != null && !parserContext.isNested()) {
>     			try {
>     				String id = resolveId(element, definition, parserContext);
>     				if (!StringUtils.hasText(id)) {
>     					parserContext.getReaderContext().error(
>     							"Id is required for element '" + parserContext.getDelegate().getLocalName(element)
>     									+ "' when used as a top-level tag", element);
>     				}
>     				String[] aliases = null;
>     				if (shouldParseNameAsAliases()) {
>     					String name = element.getAttribute(NAME_ATTRIBUTE);
>     					if (StringUtils.hasLength(name)) {
>     						aliases = StringUtils.trimArrayElements(StringUtils.commaDelimitedListToStringArray(name));
>     					}
>     				}
>     				// 将AbstractBeanDefinition转换为BeanDefinitionHolder并注册
>     				BeanDefinitionHolder holder = new BeanDefinitionHolder(definition, id, aliases);
>     				registerBeanDefinition(holder, parserContext.getRegistry());
>     				if (shouldFireEvents()) {
>     					// 需要通知监听器进行处理
>     					BeanComponentDefinition componentDefinition = new BeanComponentDefinition(holder);
>     					postProcessComponentDefinition(componentDefinition);
>     					parserContext.registerComponent(componentDefinition);
>     				}
>     			}
>     			catch (BeanDefinitionStoreException ex) {
>     				String msg = ex.getMessage();
>     				parserContext.getReaderContext().error((msg != null ? msg : ex.toString()), element);
>     				return null;
>     			}
>     		}
>     		return definition;
>     	}
>         ...
>     }
>     ```
>
> * 面向注解BeanDefinition解析: **AnnotatedBeanDefinitionReader**
>
>   ```java
>   package com.holelin.readsource;
>     
>   import org.springframework.beans.factory.support.DefaultListableBeanFactory;
>   import org.springframework.context.annotation.AnnotatedBeanDefinitionReader;
>     
>   public class AnnotatedBeanDefinitionReaderDemo {
>   	public static void main(String[] args) {
>   		DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
>   		AnnotatedBeanDefinitionReader beanDefinitionReader = new AnnotatedBeanDefinitionReader(beanFactory);
>   		int beanDefinitionCountBefore = beanFactory.getBeanDefinitionCount();
>   		beanDefinitionReader.register(AnnotatedBeanDefinitionReaderDemo.class);
>   		int beanDefinitionCountAfter = beanFactory.getBeanDefinitionCount();
>   		System.out.println("已加载 BeanDefinition 数量: " + (beanDefinitionCountAfter - beanDefinitionCountBefore));
>   		AnnotatedBeanDefinitionReaderDemo demo = beanFactory.getBean("annotatedBeanDefinitionReaderDemo", AnnotatedBeanDefinitionReaderDemo.class);
>   		System.out.println(demo);
>   	}
>   }
>   ```

##### Spring Bean注册阶段

> BeanDefinition注册接口
>
> * **BeanDefinitionRegistry**
>
>   ```java
>   /*
>    * Copyright 2002-2018 the original author or authors.
>    *
>    * Licensed under the Apache License, Version 2.0 (the "License");
>    * you may not use this file except in compliance with the License.
>    * You may obtain a copy of the License at
>    *
>    *      https://www.apache.org/licenses/LICENSE-2.0
>    *
>    * Unless required by applicable law or agreed to in writing, software
>    * distributed under the License is distributed on an "AS IS" BASIS,
>    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
>    * See the License for the specific language governing permissions and
>    * limitations under the License.
>    */
>     
>   package org.springframework.beans.factory.support;
>     
>   import org.springframework.beans.factory.BeanDefinitionStoreException;
>   import org.springframework.beans.factory.NoSuchBeanDefinitionException;
>   import org.springframework.beans.factory.config.BeanDefinition;
>   import org.springframework.core.AliasRegistry;
>     
>   /**
>    * Interface for registries that hold bean definitions, for example RootBeanDefinition
>    * and ChildBeanDefinition instances. Typically implemented by BeanFactories that
>    * internally work with the AbstractBeanDefinition hierarchy.
>    *
>    * <p>This is the only interface in Spring's bean factory packages that encapsulates
>    * <i>registration</i> of bean definitions. The standard BeanFactory interfaces
>    * only cover access to a <i>fully configured factory instance</i>.
>    *
>    * <p>Spring's bean definition readers expect to work on an implementation of this
>    * interface. Known implementors within the Spring core are DefaultListableBeanFactory
>    * and GenericApplicationContext.
>    *
>    * @author Juergen Hoeller
>    * @since 26.11.2003
>    * @see org.springframework.beans.factory.config.BeanDefinition
>    * @see AbstractBeanDefinition
>    * @see RootBeanDefinition
>    * @see ChildBeanDefinition
>    * @see DefaultListableBeanFactory
>    * @see org.springframework.context.support.GenericApplicationContext
>    * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
>    * @see PropertiesBeanDefinitionReader
>    */
>   public interface BeanDefinitionRegistry extends AliasRegistry {
>     
>   	/**
>   	 * Register a new bean definition with this registry.
>   	 * Must support RootBeanDefinition and ChildBeanDefinition.
>   	 * @param beanName the name of the bean instance to register
>   	 * @param beanDefinition definition of the bean instance to register
>   	 * @throws BeanDefinitionStoreException if the BeanDefinition is invalid
>   	 * @throws BeanDefinitionOverrideException if there is already a BeanDefinition
>   	 * for the specified bean name and we are not allowed to override it
>   	 * @see GenericBeanDefinition
>   	 * @see RootBeanDefinition
>   	 * @see ChildBeanDefinition
>   	 */
>   	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
>   			throws BeanDefinitionStoreException;
>     
>   	/**
>   	 * Remove the BeanDefinition for the given name.
>   	 * @param beanName the name of the bean instance to register
>   	 * @throws NoSuchBeanDefinitionException if there is no such bean definition
>   	 */
>   	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
>     
>   	/**
>   	 * Return the BeanDefinition for the given bean name.
>   	 * @param beanName name of the bean to find a definition for
>   	 * @return the BeanDefinition for the given name (never {@code null})
>   	 * @throws NoSuchBeanDefinitionException if there is no such bean definition
>   	 */
>   	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
>     
>   	/**
>   	 * Check if this registry contains a bean definition with the given name.
>   	 * @param beanName the name of the bean to look for
>   	 * @return if this registry contains a bean definition with the given name
>   	 */
>   	boolean containsBeanDefinition(String beanName);
>     
>   	/**
>   	 * Return the names of all beans defined in this registry.
>   	 * @return the names of all beans defined in this registry,
>   	 * or an empty array if none defined
>   	 */
>   	String[] getBeanDefinitionNames();
>     
>   	/**
>   	 * Return the number of beans defined in the registry.
>   	 * @return the number of beans defined in the registry
>   	 */
>   	int getBeanDefinitionCount();
>     
>   	/**
>   	 * Determine whether the given bean name is already in use within this registry,
>   	 * i.e. whether there is a local bean or alias registered under this name.
>   	 * @param beanName the name to check
>   	 * @return whether the given bean name is already in use
>   	 */
>   	boolean isBeanNameInUse(String beanName);
>     
>   }
>   ```
>   
>   ```java
>   	//---------------------------------------------------------------------
>   	// Implementation of BeanDefinitionRegistry interface
>   	//---------------------------------------------------------------------
>     
>   	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
>   			throws BeanDefinitionStoreException {
>     
>   		Assert.hasText(beanName, "Bean name must not be empty");
>   		Assert.notNull(beanDefinition, "BeanDefinition must not be null");
>     
>   		if (beanDefinition instanceof AbstractBeanDefinition) {
>   			try {
>   				/**
>   				 * 注册前的最后一次校验,这里的校验不同于之前的XML文件校验
>   				 * 主要是对于AbstractBeanDefinition属性中的methodOverrides校验
>   				 * 校验methodOverrides是否与工厂方法并存或者methodOverrides对应的方法根本不存在
>   				 */
>   				((AbstractBeanDefinition) beanDefinition).validate();
>   			}
>   			catch (BeanDefinitionValidationException ex) {
>   				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
>   						"Validation of bean definition failed", ex);
>   			}
>   		}
>     
>   		BeanDefinition oldBeanDefinition;
>     
>   		// 因为beanDefinitionMap是全局变量,这里定会存在并发访问的情况
>   		synchronized (this.beanDefinitionMap) {
>   			oldBeanDefinition = this.beanDefinitionMap.get(beanName);
>   			// 处理注册已经注册的beanName情况
>   			if (oldBeanDefinition != null) {
>   				// 如果对应的BeanName已经注册且在配置中配置了bean不允许被覆盖,则抛出异常
>   				if (!this.allowBeanDefinitionOverriding) {
>   					throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
>   							"Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
>   							"': There is already [" + oldBeanDefinition + "] bound.");
>   				}
>   				else {
>   					if (this.logger.isInfoEnabled()) {
>   						this.logger.info("Overriding bean definition for bean '" + beanName +
>   								"': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");
>   					}
>   				}
>   			}
>   			else {
>   				// 记录beanName 用List保证注册顺序
>                  	/** List of bean definition names, in registration order */ 
>   				// private final List<String> beanDefinitionNames = new ArrayList<String>(64);
>   				this.beanDefinitionNames.add(beanName);
>   				this.frozenBeanDefinitionNames = null;
>   			}
>   			// 注册beanDefinition
>   			this.beanDefinitionMap.put(beanName, beanDefinition);
>   		}
>     
>   		if (oldBeanDefinition != null || containsSingleton(beanName)) {
>   			// 重置所有beanName对应的缓存
>   			resetBeanDefinition(beanName);
>   		}
>   	}
>   ```

##### Spring BeanDefinition合并阶段

> BeanDefinition合并
>
> * 父子BeanDefinition合并: Root BeanDefinition 不需要合并,没有parent; 普通 BeanDefinition GenericBeanDefinition
>   * 当前BeanFactory查找
>   * 层次性BeanFactory查找

```java
// org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
	/**
	 * Return an instance, which may be shared or independent, of the specified bean.
	 * @param name the name of the bean to retrieve
	 * @param requiredType the required type of the bean to retrieve
	 * @param args arguments to use when creating a bean instance using explicit arguments
	 * (only applied when creating a new instance as opposed to retrieving an existing one)
	 * @param typeCheckOnly whether the instance is obtained for a type check,
	 * not for actual use
	 * @return an instance of the bean
	 * @throws BeansException if the bean could not be created
	 */
	@SuppressWarnings("unchecked")
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		// 提取对应beanName
		/**
		 * 此处传入的参数可能是别名,也可能是FactoryBean,所以需要解析
		 * * 去除FactoryBean的修饰符,也就是如果name="&aa",那么会首先去除&而使name="aa"
		 * * 取指定alias所表示的最终beanName,例如别名A指向名称为B的bean则返回B;
		 * 若别名A指向别名B,别名B又指向名称C的bean则返回C
		 */
		final String beanName = transformedBeanName(name);
		Object bean;
		/**
		 * 检查缓存中或者实例工厂中是否有对应的实例
		 * 为什么首先使用这段代码?
		 * 因为在创建单例bean的时候会存在依赖注入的情况,
		 * 而在创建依赖的时候为了避免循环依赖,Spring创建bean的原则是不等bean创建完成
		 * 在Spring中创建bean的原则是不等bean创建完成就将创建的bean的ObjectFactory提早曝光加入到缓存中,一旦下一个bean创建时候需要依赖上一个bean则直接使用ObjectFactory
		 * 网上形象比喻:
		 * `创建bean的时候，就是打包快递发货，主管为了知道你今天要派发多少个包裹，为了节省大家时间以及以免统计漏掉的情况。
		 * 你可以先拿出一个包裹箱子，上面写上要寄收件人、收货地址、联系方式等等，但是这时候还没有往里面打包真正的快递。
		 * 这里曝光的bean就相当于这个快递箱子。`
		 *
		 */
		// 直接尝试从缓存获取或者singletonFactories中的ObjectFactory中获取
		// Eagerly check singleton cache for manually registered singletons.
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			if (logger.isTraceEnabled()) {
				if (isSingletonCurrentlyInCreation(beanName)) {
					logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
							"' that is not fully initialized yet - a consequence of a circular reference");
				}
				else {
					logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
				}
			}
			// 返回对应的实例,有时候存在诸如BeanFactory的情况并不是直接返回实例本身而是返回指定方法返回的实例
			// 缓存中记录的知识最原始的bean状态,并不是我们最终想要的bean
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			/**
			 * 只有在单例情况下才会尝试解决循环依赖,原型模式情况下,
			 * 如果存在A中有B的属性,B中有A的属性,那么当依赖注入的时候,就会产生当A还未创建完成的时候
			 * 因为对于B的创建再次返回创建A,造成循环依赖,依旧是下面的情况
			 * isPrototypeCurrentlyInCreation(beanName)为true
			 */
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}
			// 如果beanDefinitionMap中也就是在所有已加载的类中不包括beanName("!containsBeanDefinition(beanName)"),
			// 则尝试从parentBeanFactory中检测
			// Check if bean definition exists in this factory.
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent.
				String nameToLookup = originalBeanName(name);
				if (parentBeanFactory instanceof AbstractBeanFactory) {
					return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
							nameToLookup, requiredType, args, typeCheckOnly);
				}
				else if (args != null) {
					// 递归到BeanFactory中寻找
					// Delegation to parent with explicit args.
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else if (requiredType != null) {
					// No args -> delegate to standard getBean method.
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
				else {
					return (T) parentBeanFactory.getBean(nameToLookup);
				}
			}
			// 如果不是仅仅做类型检查则是创建bean,这里要进行记录

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}
			// 将存储XML配置文件的GenericBeanDefinition转换为RootBeanDefinition,
			// 如果指定BeanName是子Bean的话同时会合并父类的相关属性
			try {
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// Guarantee initialization of beans that the current bean depends on.
				String[] dependsOn = mbd.getDependsOn();
				// 若存在依赖则需要递归实列化依赖的bean

				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						// 缓存依赖调用
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}
				// 实例化依赖的bean后便可以实例化mbd本身了
				// singleton模式的创建
				// Create bean instance.
				if (mbd.isSingleton()) {
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

				else if (mbd.isPrototype()) {
					// prototype模式的创建(new)
					// It's a prototype -> create a new instance.
					Object prototypeInstance = null;
					try {
						beforePrototypeCreation(beanName);
						prototypeInstance = createBean(beanName, mbd, args);
					}
					finally {
						afterPrototypeCreation(beanName);
					}
					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
				}

				else {
					// 指定的scope上实例化bean
					String scopeName = mbd.getScope();
					final Scope scope = this.scopes.get(scopeName);
					if (scope == null) {
						throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
					}
					try {
						Object scopedInstance = scope.get(beanName, () -> {
							beforePrototypeCreation(beanName);
							try {
								return createBean(beanName, mbd, args);
							}
							finally {
								afterPrototypeCreation(beanName);
							}
						});
						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
						throw new BeanCreationException(beanName,
								"Scope '" + scopeName + "' is not active for the current thread; consider " +
								"defining a scoped proxy for this bean if you intend to refer to it from a singleton",
								ex);
					}
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}

		// 检查需要的类型是否符合bean的实际类型
		// Check if required type matches the type of the actual bean instance.
		if (requiredType != null && !requiredType.isInstance(bean)) {
			try {
				// 类型转换
				T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
				if (convertedBean == null) {
					throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
				}
				return convertedBean;
			}
			catch (TypeMismatchException ex) {
				if (logger.isTraceEnabled()) {
					logger.trace("Failed to convert bean '" + name + "' to required type '" +
							ClassUtils.getQualifiedName(requiredType) + "'", ex);
				}
				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
			}
		}
		return (T) bean;
	}
```

```java
// 路径 org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition(java.lang.String)
	/**
	 * Return a 'merged' BeanDefinition for the given bean name,
	 * merging a child bean definition with its parent if necessary.
	 * <p>This {@code getMergedBeanDefinition} considers bean definition
	 * in ancestors as well.
	 * @param name the name of the bean to retrieve the merged definition for
	 * (may be an alias)
	 * @return a (potentially merged) RootBeanDefinition for the given bean
	 * @throws NoSuchBeanDefinitionException if there is no bean with the given name
	 * @throws BeanDefinitionStoreException in case of an invalid bean definition
	 */
	public BeanDefinition getMergedBeanDefinition(String name) throws BeansException {
		String beanName = transformedBeanName(name);

		// Efficiently check whether bean definition exists in this factory.
		if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
			return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(beanName);
		}
		// Resolve merged bean definition locally.
		return getMergedLocalBeanDefinition(beanName);
	}
```

```java
//  路径: org.springframework.beans.factory.support.AbstractBeanFactory#getMergedLocalBeanDefinition
	/**
	 * Return a merged RootBeanDefinition, traversing the parent bean definition
	 * if the specified bean corresponds to a child bean definition.
	 * @param beanName the name of the bean to retrieve the merged definition for
	 * @return a (potentially merged) RootBeanDefinition for the given bean
	 * @throws NoSuchBeanDefinitionException if there is no bean with the given name
	 * @throws BeanDefinitionStoreException in case of an invalid bean definition
	 */
	protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException {
		// Quick check on the concurrent map first, with minimal locking.
		RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);
		if (mbd != null) {
			return mbd;
		}
		return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));
	}
```

```java
// 路径: org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition(java.lang.String, org.springframework.beans.factory.config.BeanDefinition, org.springframework.beans.factory.config.BeanDefinition)
	/**
	 * Return a RootBeanDefinition for the given bean, by merging with the
	 * parent if the given bean's definition is a child bean definition.
	 * @param beanName the name of the bean definition
	 * @param bd the original bean definition (Root/ChildBeanDefinition)
	 * @param containingBd (嵌套Bean) the containing bean definition in case of inner bean,
	 * or {@code null} in case of a top-level bean
	 * @return a (potentially merged) RootBeanDefinition for the given bean
	 * @throws BeanDefinitionStoreException in case of an invalid bean definition
	 */
	protected RootBeanDefinition getMergedBeanDefinition(
			String beanName, BeanDefinition bd, BeanDefinition containingBd)
			throws BeanDefinitionStoreException {

		synchronized (this.mergedBeanDefinitions) {
			RootBeanDefinition mbd = null;

			// Check with full lock now in order to enforce the same merged instance.
			if (containingBd == null) {
				mbd = this.mergedBeanDefinitions.get(beanName);
			}

			if (mbd == null) {
				if (bd.getParentName() == null) {
					// Use copy of given root bean definition.
					if (bd instanceof RootBeanDefinition) {
						mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
					}
					else {
						mbd = new RootBeanDefinition(bd);
					}
				}
				else {
					// Child bean definition: needs to be merged with parent.
					BeanDefinition pbd;
					try {
						String parentBeanName = transformedBeanName(bd.getParentName());
						if (!beanName.equals(parentBeanName)) {
							pbd = getMergedBeanDefinition(parentBeanName);
						}
						else {
							if (getParentBeanFactory() instanceof ConfigurableBeanFactory) {
								pbd = ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(parentBeanName);
							}
							else {
								throw new NoSuchBeanDefinitionException(bd.getParentName(),
										"Parent name '" + bd.getParentName() + "' is equal to bean name '" + beanName +
										"': cannot be resolved without an AbstractBeanFactory parent");
							}
						}
					}
					catch (NoSuchBeanDefinitionException ex) {
						throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName,
								"Could not resolve parent bean definition '" + bd.getParentName() + "'", ex);
					}
					// Deep copy with overridden values.
					mbd = new RootBeanDefinition(pbd);
					mbd.overrideFrom(bd);
				}

				// Set default singleton scope, if not configured before.
				if (!StringUtils.hasLength(mbd.getScope())) {
					mbd.setScope(RootBeanDefinition.SCOPE_SINGLETON);
				}

				// A bean contained in a non-singleton bean cannot be a singleton itself.
				// Let's correct this on the fly here, since this might be the result of
				// parent-child merging for the outer bean, in which case the original inner bean
				// definition will not have inherited the merged outer bean's singleton status.
				if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {
					mbd.setScope(containingBd.getScope());
				}

				// Only cache the merged bean definition if we're already about to create an
				// instance of the bean, or at least have already created an instance before.
				if (containingBd == null && isCacheBeanMetadata() && isBeanEligibleForMetadataCaching(beanName)) {
					this.mergedBeanDefinitions.put(beanName, mbd);
				}
			}

			return mbd;
		}
	}
```

##### Spring Bean Class 加载阶段

> ClassLoader 类加载
>
> ```java
> // org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
> 	/**
> 	 * Central method of this class: creates a bean instance,
> 	 * populates the bean instance, applies post-processors, etc.
> 	 * @see #doCreateBean
> 	 */
> 	@Override
> 	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
> 			throws BeanCreationException {
> 
> 		if (logger.isTraceEnabled()) {
> 			logger.trace("Creating instance of bean '" + beanName + "'");
> 		}
> 		RootBeanDefinition mbdToUse = mbd;
> 
> 		// 锁定class,根据设置的class属性或者根据className来解析Class
> 		// Make sure bean class is actually resolved at this point, and
> 		// clone the bean definition in case of a dynamically resolved Class
> 		// which cannot be stored in the shared merged bean definition.
> 		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
> 		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
> 			mbdToUse = new RootBeanDefinition(mbd);
> 			mbdToUse.setBeanClass(resolvedClass);
> 		}
> 		// 验证以及准备覆盖的方法
> 		// Prepare method overrides.
> 		try {
> 			// 对override属性进行标记以及验证,针对lookup-method和replace-method
> 			mbdToUse.prepareMethodOverrides();
> 		}
> 		catch (BeanDefinitionValidationException ex) {
> 			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
> 					beanName, "Validation of method overrides failed", ex);
> 		}
> 		// ... 省略
> 	}
> 
> // ==> org.springframework.beans.factory.support.AbstractBeanFactory#resolveBeanClass
> 	/**
> 	 * Resolve the bean class for the specified bean definition,
> 	 * resolving a bean class name into a Class reference (if necessary)
> 	 * and storing the resolved Class in the bean definition for further use.
> 	 * @param mbd the merged bean definition to determine the class for
> 	 * @param beanName the name of the bean (for error handling purposes)
> 	 * @param typesToMatch the types to match in case of internal type matching purposes
> 	 * (also signals that the returned {@code Class} will never be exposed to application code)
> 	 * @return the resolved bean class (or {@code null} if none)
> 	 * @throws CannotLoadBeanClassException if we failed to load the class
> 	 */
> 	protected Class<?> resolveBeanClass(final RootBeanDefinition mbd, String beanName, final Class<?>... typesToMatch)
> 			throws CannotLoadBeanClassException {
> 		try {
> 			if (mbd.hasBeanClass()) {
> 				return mbd.getBeanClass();
> 			}
> 			if (System.getSecurityManager() != null) {
> 				return AccessController.doPrivileged(new PrivilegedExceptionAction<Class<?>>() {
> 					public Class<?> run() throws Exception {
> 						return doResolveBeanClass(mbd, typesToMatch);
> 					}
> 				}, getAccessControlContext());
> 			}
> 			else {
> 				return doResolveBeanClass(mbd, typesToMatch);
> 			}
> 		}
> 		catch (PrivilegedActionException pae) {
> 			ClassNotFoundException ex = (ClassNotFoundException) pae.getException();
> 			throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), ex);
> 		}
> 		catch (ClassNotFoundException ex) {
> 			throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), ex);
> 		}
> 		catch (LinkageError err) {
> 			throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), err);
> 		}
> 	}
> ```
>
> Java Security安全控制
>
> ```java
>     @CallerSensitive
>     public static ClassLoader getSystemClassLoader() {
>         initSystemClassLoader();
>         if (scl == null) {
>             return null;
>         }
>         SecurityManager sm = System.getSecurityManager();
>         if (sm != null) {
>             checkClassLoaderPermission(scl, Reflection.getCallerClass());
>         }
>         return scl;
>     }
> 
> ```
>
> ConfigurableBeanFactory 临时ClassLoader

##### Spring Bean 实例化阶段

###### 前阶段: 非主流生命周期

> org.springframework.beans.factory.config.**InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation**

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
	/**
	 * Central method of this class: creates a bean instance,
	 * populates the bean instance, applies post-processors, etc.
	 * @see #doCreateBean
	 */
	@Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
		// ...省略
		try {
			// 给BeanPostProcessors一个机会来返回代理来替代真正的实例
			// 应用初始化前的后处理器,解析指定bean是否存在初始化前的短路操作
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}
		// ...省略
	}
// 路径: org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation
	/**
	 * Apply before-instantiation post-processors, resolving whether there is a
	 * before-instantiation shortcut for the specified bean.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @return the shortcut-determined bean instance, or {@code null} if none
	 */
	@Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
// 路径: org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInstantiation
	/**
	 * Apply InstantiationAwareBeanPostProcessors to the specified bean definition
	 * (by class and name), invoking their {@code postProcessBeforeInstantiation} methods.
	 * <p>Any returned object will be used as the bean instead of actually instantiating
	 * the target bean. A {@code null} return value from the post-processor will
	 * result in the target bean being instantiated.
	 * @param beanClass the class of the bean to be instantiated
	 * @param beanName the name of the bean
	 * @return the bean object to use instead of a default instance of the target bean, or {@code null}
	 * @see InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
	 */
	@Nullable
	protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof InstantiationAwareBeanPostProcessor) {
				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
				Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
				if (result != null) {
					return result;
				}
			}
		}
		return null;
	}

```

###### 中阶段

> org.springframework.beans.factory.support.**AbstractAutowireCapableBeanFactory#doCreateBean**
>
> 实例化方式 
>
> * 传统实例化方式
>   * 实例化策略: org.springframework.beans.factory.support.**InstantiationStrategy**
>   
>   ![img](http://www.chenjunlin.vip/img/spring/InstantiationStrategy类图.png)
>   
> * 构造器依赖注入

```java
	/**
	 * Actually create the specified bean. Pre-creation processing has already happened
	 * at this point, e.g. checking {@code postProcessBeforeInstantiation} callbacks.
	 * <p>Differentiates between default bean instantiation, use of a
	 * factory method, and autowiring a constructor.
	 * @param beanName the name of the bean
	 * @param mbd the merged bean definition for the bean
	 * @param args explicit arguments to use for constructor or factory method invocation
	 * @return a new instance of the bean
	 * @throws BeanCreationException if the bean could not be created
	 * @see #instantiateBean
	 * @see #instantiateUsingFactoryMethod
	 * @see #autowireConstructor
	 */
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		// 单例则需要首先清除缓存
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			// 根据指定bean使用对应的策略创建新的实例,如工厂方法,构造函数自动注入,简单初始化
			// 如果存在工厂方法则使用工厂方法进行初始化
			// 一个类有多个构造函数,每个构造函数都有不同的参数,所有需要根据参数锁定构造函数并进行初始化.
			// 如果既不存在工厂方法也不存在带有参数的构造函数,则使用默认的构造函数进行bean的实列化
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		// ...省略

	}
```

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance
	/**
	 * Create a new instance for the specified bean, using an appropriate instantiation strategy:
	 * factory method, constructor autowiring, or simple instantiation.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @param args explicit arguments to use for constructor or factory method invocation
	 * @return a BeanWrapper for the new instance
	 * @see #obtainFromSupplier
	 * @see #instantiateUsingFactoryMethod
	 * @see #autowireConstructor
	 * @see #instantiateBean
	 */
	protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}
		// 通过FactoryMethod来创建实例
		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// Shortcut when re-creating the same bean...
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		if (resolved) {
			if (autowireNecessary) {
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
            // 使用构造器注入的方式
			return autowireConstructor(beanName, mbd, ctors, null);
		}
		// 使用无参构造器的方式
		// No special handling: simply use no-arg constructor.
		return instantiateBean(beanName, mbd);
	}

```

* **instantiateBean**

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#instantiateBean
	/**
	 * Instantiate the given bean using its default constructor.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @return a BeanWrapper for the new instance
	 */
	protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
		try {
			Object beanInstance;
			final BeanFactory parent = this;
			if (System.getSecurityManager() != null) {
				beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
						getInstantiationStrategy().instantiate(mbd, beanName, parent),
						getAccessControlContext());
			}
			else {
                // 入口
				beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
			}
			BeanWrapper bw = new BeanWrapperImpl(beanInstance);
			initBeanWrapper(bw);
			return bw;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
		}
	}
// org.springframework.beans.factory.support.SimpleInstantiationStrategy#instantiate(org.springframework.beans.factory.support.RootBeanDefinition, java.lang.String, org.springframework.beans.factory.BeanFactory)
	@Override
	public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
        // 如果有需要覆盖或者动态替换的方法则当然需要使用cglib进行动态代理,因为可以在创建代理同时将动态方法织入类中
		// 但是如果没有需要动态改变的方法,为了方便直接使用反射就可以了
		// Don't override the class with CGLIB if no overrides.
		if (!bd.hasMethodOverrides()) {
			Constructor<?> constructorToUse;
			synchronized (bd.constructorArgumentLock) {
				constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
				if (constructorToUse == null) {
					final Class<?> clazz = bd.getBeanClass();
					if (clazz.isInterface()) {
						throw new BeanInstantiationException(clazz, "Specified class is an interface");
					}
					try {
						if (System.getSecurityManager() != null) {
							constructorToUse = AccessController.doPrivileged(
									(PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
						}
						else {
							constructorToUse = clazz.getDeclaredConstructor();
						}
						bd.resolvedConstructorOrFactoryMethod = constructorToUse;
					}
					catch (Throwable ex) {
						throw new BeanInstantiationException(clazz, "No default constructor found", ex);
					}
				}
			}
			return BeanUtils.instantiateClass(constructorToUse);
		}
		else {
			// Must generate CGLIB subclass.
			return instantiateWithMethodInjection(bd, beanName, owner);
		}
	}
```

*  **autowireConstructor**

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#autowireConstructor
	/**
	 * "autowire constructor" (with constructor arguments by type) behavior.
	 * Also applied if explicit constructor argument values are specified,
	 * matching all remaining arguments with beans from the bean factory.
	 * <p>This corresponds to constructor injection: In this mode, a Spring
	 * bean factory is able to host components that expect constructor-based
	 * dependency resolution.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @param ctors the chosen candidate constructors
	 * @param explicitArgs argument values passed in programmatically via the getBean method,
	 * or {@code null} if none (-> use constructor argument values from bean definition)
	 * @return a BeanWrapper for the new instance
	 */
	protected BeanWrapper autowireConstructor(
			String beanName, RootBeanDefinition mbd, @Nullable Constructor<?>[] ctors, @Nullable Object[] explicitArgs) {

		return new ConstructorResolver(this).autowireConstructor(beanName, mbd, ctors, explicitArgs);
	}
```

```java
	/**
	 * "autowire constructor" (with constructor arguments by type) behavior.
	 * Also applied if explicit constructor argument values are specified,
	 * matching all remaining arguments with beans from the bean factory.
	 * <p>This corresponds to constructor injection: In this mode, a Spring
	 * bean factory is able to host components that expect constructor-based
	 * dependency resolution.
	 * @param beanName the name of the bean
	 * @param mbd the merged bean definition for the bean
	 * @param chosenCtors chosen candidate constructors (or {@code null} if none)
	 * @param explicitArgs argument values passed in programmatically via the getBean method,
	 * or {@code null} if none (-> use constructor argument values from bean definition)
	 * @return a BeanWrapper for the new instance
	 */
	public BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd,
			@Nullable Constructor<?>[] chosenCtors, @Nullable Object[] explicitArgs) {

		BeanWrapperImpl bw = new BeanWrapperImpl();
		this.beanFactory.initBeanWrapper(bw);

		Constructor<?> constructorToUse = null;
		ArgumentsHolder argsHolderToUse = null;
		Object[] argsToUse = null;
		// explicitArgs通过getBean方法传入
		// 如果getBean方法调用的时候指定方法参数那么直接使用
		if (explicitArgs != null) {
			argsToUse = explicitArgs;
		}
		else {
			// 如果在getBean方法时候没有指定则尝试从配置文件中解析
			Object[] argsToResolve = null;
			// 尝试从缓存中获取
			synchronized (mbd.constructorArgumentLock) {
				constructorToUse = (Constructor<?>) mbd.resolvedConstructorOrFactoryMethod;
				if (constructorToUse != null && mbd.constructorArgumentsResolved) {
					// Found a cached constructor...
					// 从缓存中取
					argsToUse = mbd.resolvedConstructorArguments;
					if (argsToUse == null) {
						// 配置的构造函数参数
						argsToResolve = mbd.preparedConstructorArguments;
					}
				}
			}
			// 如果缓存中存在
			if (argsToResolve != null) {
				// 解析参数类型,如给定方法的构造函数A(int,int)则通过此方法后就会把配置中的("1","1")转换为(1,1)
				// 缓存中值可能是原始值也肯定是最终值
				argsToUse = resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve, true);
			}
		}

		if (constructorToUse == null || argsToUse == null) {
			// Take specified constructors, if any.
			Constructor<?>[] candidates = chosenCtors;
			if (candidates == null) {
				Class<?> beanClass = mbd.getBeanClass();
				try {
					candidates = (mbd.isNonPublicAccessAllowed() ?
							beanClass.getDeclaredConstructors() : beanClass.getConstructors());
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Resolution of declared constructors on bean Class [" + beanClass.getName() +
							"] from ClassLoader [" + beanClass.getClassLoader() + "] failed", ex);
				}
			}

			if (candidates.length == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
				Constructor<?> uniqueCandidate = candidates[0];
				if (uniqueCandidate.getParameterCount() == 0) {
					synchronized (mbd.constructorArgumentLock) {
						mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
						mbd.constructorArgumentsResolved = true;
						mbd.resolvedConstructorArguments = EMPTY_ARGS;
					}
					bw.setBeanInstance(instantiate(beanName, mbd, uniqueCandidate, EMPTY_ARGS));
					return bw;
				}
			}

			// Need to resolve the constructor.
			boolean autowiring = (chosenCtors != null ||
					mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
			ConstructorArgumentValues resolvedValues = null;

			int minNrOfArgs;
			if (explicitArgs != null) {
				minNrOfArgs = explicitArgs.length;
			}
			else {
				ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
				resolvedValues = new ConstructorArgumentValues();
				minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
			}
			// 排序给定的构造函数,public构造函数优先参数数量降序,非public构造函数参数数量降序
			AutowireUtils.sortConstructors(candidates);
			int minTypeDiffWeight = Integer.MAX_VALUE;
			Set<Constructor<?>> ambiguousConstructors = null;
			LinkedList<UnsatisfiedDependencyException> causes = null;

			for (Constructor<?> candidate : candidates) {

				int parameterCount = candidate.getParameterCount();

				if (constructorToUse != null && argsToUse != null && argsToUse.length > parameterCount) {
					// 如果已经选用的构造函数或者需要的参数个数小于当前的构造函数个数则终止,因为已经按照参数个数降序排列
					// Already found greedy constructor that can be satisfied ->
					// do not look any further, there are only less greedy constructors left.
					break;
				}
				if (parameterCount < minNrOfArgs) {
					// 参数个数不相等
					continue;
				}

				ArgumentsHolder argsHolder;
				Class<?>[] paramTypes = candidate.getParameterTypes();
				if (resolvedValues != null) {
					// 有参数则根据构造对应参数类型的参数
					try {
						// 注解上获取参数名称
						String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, parameterCount);
						if (paramNames == null) {
							// 获取参数名称探索器
							ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
							if (pnd != null) {
								// 获取指定构造函数的参数名称
								paramNames = pnd.getParameterNames(candidate);
							}
						}
						// 根据名称和数据类型创建参数持有者
						argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
								getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
					}
					catch (UnsatisfiedDependencyException ex) {
						if (logger.isTraceEnabled()) {
							logger.trace("Ignoring constructor [" + candidate + "] of bean '" + beanName + "': " + ex);
						}
						// Swallow and try next constructor.
						if (causes == null) {
							causes = new LinkedList<>();
						}
						causes.add(ex);
						continue;
					}
				}
				else {
					// Explicit arguments given -> arguments length must match exactly.
					if (parameterCount != explicitArgs.length) {
						continue;
					}
					// 构造函数没有参数的情况
					argsHolder = new ArgumentsHolder(explicitArgs);
				}
				// 探测是否不确定的构造函数存在,例如不同构造函数的参数为父子关系
				int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
						argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
				// Choose this constructor if it represents the closest match.
				if (typeDiffWeight < minTypeDiffWeight) {
					constructorToUse = candidate;
					argsHolderToUse = argsHolder;
					argsToUse = argsHolder.arguments;
					minTypeDiffWeight = typeDiffWeight;
					ambiguousConstructors = null;
				}
				else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
					if (ambiguousConstructors == null) {
						ambiguousConstructors = new LinkedHashSet<>();
						ambiguousConstructors.add(constructorToUse);
					}
					ambiguousConstructors.add(candidate);
				}
			}

			if (constructorToUse == null) {
				if (causes != null) {
					UnsatisfiedDependencyException ex = causes.removeLast();
					for (Exception cause : causes) {
						this.beanFactory.onSuppressedException(cause);
					}
					throw ex;
				}
				throw new BeanCreationException(mbd.getResourceDescription(), beanName,
						"Could not resolve matching constructor " +
						"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities)");
			}
			else if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) {
				throw new BeanCreationException(mbd.getResourceDescription(), beanName,
						"Ambiguous constructor matches found in bean '" + beanName + "' " +
						"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " +
						ambiguousConstructors);
			}

			if (explicitArgs == null && argsHolderToUse != null) {
				// 将解析的构造函数加入缓存
				argsHolderToUse.storeCache(mbd, constructorToUse);
			}
		}

		Assert.state(argsToUse != null, "Unresolved constructor arguments");
		bw.setBeanInstance(instantiate(beanName, mbd, constructorToUse, argsToUse));
		return bw;
	}
```

###### 后阶段

> 判断是否要进行populate属性赋值
>
> org.springframework.beans.factory.config.**InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation**
>
> **注:Bean已经实例化,但属性尚未赋值**

```java
	/**
	 * Actually create the specified bean. Pre-creation processing has already happened
	 * at this point, e.g. checking {@code postProcessBeforeInstantiation} callbacks.
	 * <p>Differentiates between default bean instantiation, use of a
	 * factory method, and autowiring a constructor.
	 * @param beanName the name of the bean
	 * @param mbd the merged bean definition for the bean
	 * @param args explicit arguments to use for constructor or factory method invocation
	 * @return a new instance of the bean
	 * @throws BeanCreationException if the bean could not be created
	 * @see #instantiateBean
	 * @see #instantiateUsingFactoryMethod
	 * @see #autowireConstructor
	 */
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		// ...省略
		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			// 对bean进行填充,将各个属性值注入,其中可能存在依赖于其他bean的属性,则会递归初始依赖bean
			populateBean(beanName, mbd, instanceWrapper);
			// 调用初始化方法,比如init-method
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}
		// ...省略
	}

```

```java
	/**
	 * Populate the bean instance in the given BeanWrapper with the property values
	 * from the bean definition.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @param bw the BeanWrapper with bean instance
	 */
	@SuppressWarnings("deprecation")  // for postProcessPropertyValues
	protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		if (bw == null) {
			if (mbd.hasPropertyValues()) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
			}
			else {
				// 没有可填充的属性
				// Skip property population phase for null instance.
				return;
			}
		}

		// 给InstantiationAwareBeanPostProcessors最后一次机会在属性设置前改变bean
		// 如: 可以用来支撑属性注入的类型
		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					// 返回值为是否继续填充bean
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						return;
					}
				}
			}
		}
        
		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// 根据名称自动注入
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// 根据类型自动注入
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}
		// 后处理器已经初始化
		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		// 需要依赖检查
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						// 对所有需要依赖检查的属性进行后处理
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			// 依赖检查,对应depends-on 3.0已经弃用此属性
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
			// 将属性应用到bean中
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

##### Spring Bean 属性赋值前阶段

> Bean属性值元信息
>
> * PropertyValues
>
> Bean属性赋值前回调
>
> * Spring1.2-5.0: org.springframework.beans.factory.config.**InstantiationAwareBeanPostProcessor#postProcessPropertyValues**
> * Spring 5.1: org.springframework.beans.factory.config.**InstantiationAwareBeanPostProcessor#postProcessProperties**

```java
	/**
	 * Populate the bean instance in the given BeanWrapper with the property values
	 * from the bean definition.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @param bw the BeanWrapper with bean instance
	 */
	@SuppressWarnings("deprecation")  // for postProcessPropertyValues
	protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		// ...省略
        
		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// 根据名称自动注入
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// 根据类型自动注入
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}
		// 后处理器已经初始化
		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		// 需要依赖检查
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					// 赋值前操作
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						// 对所有需要依赖检查的属性进行后处理
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			// 依赖检查,对应depends-on 3.0已经弃用此属性
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
			// 将属性应用到bean中
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

##### Spring Bean Aware 接口回调阶段

> Spring Aware接口
>
> * BeanNameAware
> * BeanClassLoaderAware
> * BeanFactoryAware
>
> 属于ApplicationContext中的生命周期中 prepareBeanFactory
>
> *  EnvironmentAware
> * EmbeddedValueResolverAware
> * ResourceLoaderAware
> * ApplicationEventPublisherAware
> * MessageSourceAware
> * ApplicationContextAware
>
> ```java
> 	/**
> 	 * Configure the factory's standard context characteristics,
> 	 * such as the context's ClassLoader and post-processors.
> 	 * @param beanFactory the BeanFactory to configure
> 	 */
> 	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
> 		// Tell the internal bean factory to use the context's class loader etc.
> 		beanFactory.setBeanClassLoader(getClassLoader());
> 		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
> 		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
> 
> 		// Configure the bean factory with context callbacks.
> 		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
> 		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
> 		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
> 		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
> 		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
> 		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
> 		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
> 		// ...省略
> 	}
> ```
>
> ```java
> // org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces
> 	private void invokeAwareInterfaces(Object bean) {
> 		if (bean instanceof EnvironmentAware) {
> 			((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
> 		}
> 		if (bean instanceof EmbeddedValueResolverAware) {
> 			((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
> 		}
> 		if (bean instanceof ResourceLoaderAware) {
> 			((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
> 		}
> 		if (bean instanceof ApplicationEventPublisherAware) {
> 			((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
> 		}
> 		if (bean instanceof MessageSourceAware) {
> 			((MessageSourceAware) bean).setMessageSource(this.applicationContext);
> 		}
> 		if (bean instanceof ApplicationContextAware) {
> 			((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
> 		}
> 	}
> ```

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
	/**
	 * Initialize the given bean instance, applying factory callbacks
	 * as well as init methods and bean post processors.
	 * <p>Called from {@link #createBean} for traditionally defined beans,
	 * and from {@link #initializeBean} for existing bean instances.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @return the initialized bean instance (potentially wrapped)
	 * @see BeanNameAware
	 * @see BeanClassLoaderAware
	 * @see BeanFactoryAware
	 * @see #applyBeanPostProcessorsBeforeInitialization
	 * @see #invokeInitMethods
	 * @see #applyBeanPostProcessorsAfterInitialization
	 */
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            // 接口回调阶段
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            //  初始化前阶段
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
            // 初始化阶段
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
            // 初始化后阶段
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
	private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```

##### Spring Bean 初始化前阶段

> 方法回调: org.springframework.beans.factory.config.**BeanPostProcessor#postProcessBeforeInitialization**

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
```

##### Spring Bean 初始化阶段

> Bean 初始化(Initialization) **invokeInitMethods**
>
> * @PostConstruct标准方法 依赖注解驱动,可添加 CommonAnnotationBeanPostProcessor
>
> ```java
> // org.springframework.context.annotation.CommonAnnotationBeanPostProcessor#CommonAnnotationBeanPostProcessor
> 	/**
> 	 * Create a new CommonAnnotationBeanPostProcessor,
> 	 * with the init and destroy annotation types set to
> 	 * {@link javax.annotation.PostConstruct} and {@link javax.annotation.PreDestroy},
> 	 * respectively.
> 	 */
> 	public CommonAnnotationBeanPostProcessor() {
> 		setOrder(Ordered.LOWEST_PRECEDENCE - 3);
> 		setInitAnnotationType(PostConstruct.class);
> 		setDestroyAnnotationType(PreDestroy.class);
> 		ignoreResourceType("javax.xml.ws.WebServiceContext");
> 	}
> ```
>
> * 实现InitializingBean接口的afterPropertiesSet()方法
>
> ```java
>     /**
>      * Interface to be implemented by beans that need to react once all their properties
>      * have been set by a {@link BeanFactory}: e.g. to perform custom initialization,
>      * or merely to check that all mandatory properties have been set.
>      *
>      * <p>An alternative to implementing {@code InitializingBean} is specifying a custom
>      * init method, for example in an XML bean definition. For a list of all bean
>      * lifecycle methods, see the {@link BeanFactory BeanFactory javadocs}.
>      *
>      * @author Rod Johnson
>      * @author Juergen Hoeller
>      * @see DisposableBean
>      * @see org.springframework.beans.factory.config.BeanDefinition#getPropertyValues()
>      * @see org.springframework.beans.factory.support.AbstractBeanDefinition#getInitMethodName()
>      */
>     public interface InitializingBean {
> 
>         /**
>          * Invoked by the containing {@code BeanFactory} after it has set all bean properties
>          * and satisfied {@link BeanFactoryAware}, {@code ApplicationContextAware} etc.
>          * <p>This method allows the bean instance to perform validation of its overall
>          * configuration and final initialization when all bean properties have been set.
>          * @throws Exception in the event of misconfiguration (such as failure to set an
>          * essential property) or if initialization fails for any other reason
>          */
>         void afterPropertiesSet() throws Exception;
> 
>     }
> ```
>
> * 自定义初始化方法

```java
	/**
	 * Give a bean a chance to react now all its properties are set,
	 * and a chance to know about its owning bean factory (this object).
	 * This means checking whether the bean implements InitializingBean or defines
	 * a custom init method, and invoking the necessary callback(s) if it does.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the merged bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @throws Throwable if thrown by init methods or by the invocation process
	 * @see #invokeCustomInitMethod
	 */
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                         // 回调方法
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
                // 回调方法
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
                // 自定义初始化方法
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
```

##### Spring Bean 初始化后阶段

> 方法回调: org.springframework.beans.factory.config.**BeanPostProcessor#postProcessAfterInitialization**

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
	@Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}

```

##### Spring Bean 初始化完成阶段

> 方法回调: 
>
> * Spring 4.1+ : org.springframework.beans.factory.**SmartInitializingSingleton#afterSingletonsInstantiated**
>   * 通常在Spring ApplicationContext使用 org.springframework.beans.factory.config.**ConfigurableListableBeanFactory#preInstantiateSingletons()**
>
> ```java
> /**
>  * Callback interface triggered at the end of the singleton pre-instantiation phase
>  * during {@link BeanFactory} bootstrap. This interface can be implemented by
>  * singleton beans in order to perform some initialization after the regular
>  * singleton instantiation algorithm, avoiding side effects with accidental early
>  * initialization (e.g. from {@link ListableBeanFactory#getBeansOfType} calls).
>  * In that sense, it is an alternative to {@link InitializingBean} which gets
>  * triggered right at the end of a bean's local construction phase.
>  *
>  * <p>This callback variant is somewhat similar to
>  * {@link org.springframework.context.event.ContextRefreshedEvent} but doesn't
>  * require an implementation of {@link org.springframework.context.ApplicationListener},
>  * with no need to filter context references across a context hierarchy etc.
>  * It also implies a more minimal dependency on just the {@code beans} package
>  * and is being honored by standalone {@link ListableBeanFactory} implementations,
>  * not just in an {@link org.springframework.context.ApplicationContext} environment.
>  *
>  * <p><b>NOTE:</b> If you intend to start/manage asynchronous tasks, preferably
>  * implement {@link org.springframework.context.Lifecycle} instead which offers
>  * a richer model for runtime management and allows for phased startup/shutdown.
>  *
>  * @author Juergen Hoeller
>  * @since 4.1
>  * @see org.springframework.beans.factory.config.ConfigurableListableBeanFactory#preInstantiateSingletons()
>  */
> public interface SmartInitializingSingleton {
> 
> 	/**
> 	 * Invoked right at the end of the singleton pre-instantiation phase,
> 	 * with a guarantee that all regular singleton beans have been created
> 	 * already. {@link ListableBeanFactory#getBeansOfType} calls within
> 	 * this method won't trigger accidental side effects during bootstrap.
> 	 * <p><b>NOTE:</b> This callback won't be triggered for singleton beans
> 	 * lazily initialized on demand after {@link BeanFactory} bootstrap,
> 	 * and not for any other bean scope either. Carefully use it for beans
> 	 * with the intended bootstrap semantics only.
> 	 */
> 	void afterSingletonsInstantiated();
> 
> }
> ```

##### Spring Bean 销毁前阶段

> 方法回调: org.springframework.beans.factory.config.**DestructionAwareBeanPostProcessor#postProcessBeforeDestruction**
>
> 在容器中被销毁

##### Spring Bean 销毁阶段

> Bean销毁(Destory)
>
> * @PerDestory 标准方法
>
> * 实现org.springframework.beans.factory.**DisposableBean#destroy**
>
>   ```java
>   /**
>    * Interface to be implemented by beans that want to release resources on destruction.
>    * A {@link BeanFactory} will invoke the destroy method on individual destruction of a
>    * scoped bean. An {@link org.springframework.context.ApplicationContext} is supposed
>    * to dispose all of its singletons on shutdown, driven by the application lifecycle.
>    *
>    * <p>A Spring-managed bean may also implement Java's {@link AutoCloseable} interface
>    * for the same purpose. An alternative to implementing an interface is specifying a
>    * custom destroy method, for example in an XML bean definition. For a list of all
>    * bean lifecycle methods, see the {@link BeanFactory BeanFactory javadocs}.
>    *
>    * @author Juergen Hoeller
>    * @since 12.08.2003
>    * @see InitializingBean
>    * @see org.springframework.beans.factory.support.RootBeanDefinition#getDestroyMethodName()
>    * @see org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons()
>    * @see org.springframework.context.ConfigurableApplicationContext#close()
>    */
>   public interface DisposableBean {
>   
>   	/**
>   	 * Invoked by the containing {@code BeanFactory} on destruction of a bean.
>   	 * @throws Exception in case of shutdown errors. Exceptions will get logged
>   	 * but not rethrown to allow other beans to release their resources as well.
>   	 */
>   	void destroy() throws Exception;
>   
>   }
>   ```
>
> * 自定义销毁方法

##### Spring Bean 垃圾收集

> Bean垃圾回收
>
> * 关闭Spring容器(应用上下文)
> * 执行GC
> * SpringBean覆盖finalize()方法被回调

#### 补充描述

##### BeanPostProcessor的使用场景有哪些?

> BeanPostProcessor提供Spring Bean初始化前和初始化后的生命周期,分别对应`postProcessBeforeInitialization`以及`postProcessAfterInitialization`方法,允许对关心的Bean进行扩展,甚至替换.
>
> 其中,ApplicationContext相关的Aware回调也是基于BeanPostProcessor实现,即`ApplicationContextAwareProcessor`

##### BeanFactoryPostProcessor与BeanPostProcessor的区别

> BeanFactoryPostProcessor是Spring BeanFactory(实际为`ConfigurableListableBeanFactory`)的后置处理器,用于扩展BeanFactory或通过BeanFactory进行依赖查找和依赖注入,BeanFactoryPostProcessor必须由Spring ApplicationContext执行,BeanFactory无法进行直接交互;
>
> 而BeanFactory与BeanPostProcessor直接关系属于1:N的关系

##### BeanFactory是怎么处理Bean生命周期的

> BeanFactory默认实现为`DefaultListableBeanFactory`其中Bean生命周期与方法映射如下:
>
> * BeanDefinition注册阶段:  `registerBeanDefinition`
> * BeanDefinition合并阶段:  `getMergedLocalBeanDefinition`
> * Bean实例化前阶段:  `resolveBeforeInstantiation`
> * Bean实例化: `createBeanInstance`
> * Bean实例化后阶段: `populateBean`
> * Bean属性赋值前阶段: `populateBean`
> * Bean属性赋值阶段: `populateBean`
> * Bean Aware接口回调阶段:  `initializeBean`
> * Bean 初始化前阶段:  `initializeBean`
> * Bean 初始化阶段:  `initializeBean`
> * Bean 初始化后阶段:  `initializeBean`
> * Bean 初始化完成阶段: `preInstantiateSingletons`
> * Bean 销毁前阶段: `destoryBean`
> * Bean 销毁阶段: `destoryBean`

