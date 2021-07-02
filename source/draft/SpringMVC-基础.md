---
title: SpringMVC-基础
mermaid: true
date: 2021-07-02 10:26:49
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags: 
- SpringMVC
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

* [【对线面试官】SpringMVC](https://mp.weixin.qq.com/s?__biz=MzU4NzA3MTc5Mg==&mid=2247484064&idx=1&sn=3a59514a8262ab61036fc89cf0b0a27e&chksm=fdf0eaffca8763e90002ce1daf365f717a4bda3e50878f65943f52d14bee78fc65e837ef32f9&token=664255414&lang=zh_CN&scene=21#wechat_redirect)
* [手码两万余字，SpringMVC 包教包会](https://juejin.cn/post/6844904031807094798#heading-5)

#### SpringMVC中的组件

* **DispatcherServlet：前端控制器**
  * 用户请求到达前端控制器，它就相当于MVC模式中的C，`DispatcherServlet` 是整个流程控制的中心，相当于是 SpringMVC 的大脑，由它调用其它组件处理用户的请求，`DispatcherServlet` 的存在降低了组件之间的耦合性。
* **HandlerMapping：处理器映射器**
  * `HandlerMapping` 负责根据用户请求找到 Handler 即处理器（也就是我们所说的 `Controller`），SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等，在实际开发中，我们常用的方式是注解方式。
* **Handler：处理器**
  * `Handler` 是继 `DispatcherServlet `前端控制器的后端控制器，在`DispatcherServlet` 的控制下 Handler 对具体的用户请求进行处理。由于 `Handler` 涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发 `Handler`。（这里所说的 Handler 就是指我们的 `Controller`）
* **HandlAdapter：处理器适配器**
  * 通过 `HandlerAdapter` 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行
* **ViewResolver：视图解析器**
  * ViewResolver 负责将处理结果生成 View 视图，ViewResolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。 SpringMVC 框架提供了很多的 View 视图类型，包括：jstlView、freemarkerView、pdfView 等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

#### SpringMVC 工作原理

##### 大致流程

* 首先有个统一处理请求的入口;
* 随后根据请求路径找到对应的映射器
* 找到处理请求的适配器
* 拦截器前置处理
* 真实处理请求(也就是调用真正的代码)
* 视图解析器处理
* 拦截器后置处理

##### 流程说明

* 用户发送请求至前端控制器`DispatcherServlet`。
* `DispatcherServlet`收到请求,根据请求路径调用`HandlerMapping`处理器映射器。
* 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给`DispatcherServlet`。
* `DispatcherServlet`调用`HandlerAdapter`处理器适配器。
* `HandlerAdapter`经过适配调用具体的处理器(`Controller`，也叫后端控制器)。
* `Controller`执行完成返回`ModelAndView`。
* `HandlerAdapter`将`Controller`执行结果`ModelAndView`返回给`DispatcherServlet`。
* `DispatcherServlet`将`ModelAndView`传给`ViewReslover`视图解析器。
* `ViewReslover`解析后返回具体`View`。
* `DispatcherServlet`根据`View`进行渲染视图（即将模型数据填充至视图中）。
* `DispatcherServlet`响应用户。

#### DispatcherServlet作用

* DispatcherServlet 是前端控制器设计模式的实现，提供 Spring Web MVC 的集中访问点，而且负责职责的分派，而且与 Spring IoC 容器无缝集成，从而可以获得 Spring 的所有好处。DispatcherServlet 主要用作职责调度工作，本身主要用于控制流程，主要职责如下：
  * 文件上传解析，如果请求类型是 multipart 将通过 MultipartResolver 进行文件上传解析；
  * 通过 HandlerMapping，将请求映射到处理器（返回一个 HandlerExecutionChain，它包括一个处理器、多个 HandlerInterceptor 拦截器）；
  * 通过 HandlerAdapter 支持多种类型的处理器(HandlerExecutionChain 中的处理器)；
  * 通过 ViewResolver 解析逻辑视图名到具体视图实现；
  * 本地化解析；
  * 渲染具体的视图等；
  * 如果执行过程中遇到异常将交给 HandlerExceptionResolver 来解析

#### DispathcherServlet配置详解

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

* **load-on-startup**：表示启动容器时初始化该 Servlet；

* **url-pattern**：表示哪些请求交给 Spring Web MVC 处理， "/" 是用来定义默认 servlet 映射的。也可以如 `*.html` 表示拦截所有以 html 为扩展名的请求

* **contextConfigLocation**：表示 SpringMVC 配置文件的路径

* 其他参数配置

  | 参数                  | 描述                                                         |
  | --------------------- | ------------------------------------------------------------ |
  | contextClass          | 实现WebApplicationContext接口的类，当前的servlet用它来创建上下文。如果这个参数没有指定， 默认使用XmlWebApplicationContext。 |
  | contextConfigLocation | 传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。 |
  | namespace             | WebApplicationContext命名空间。默认值是[server-name]-servlet。 |

  

