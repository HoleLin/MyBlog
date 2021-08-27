---
title: Spring(七) Bean作用域
mermaid: true
date: 2021-06-27 20:58:40
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 作用域 
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

* [Bean Scopes](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/spring-framework-reference/core.html)

### 作用域

| 来源        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| singleton   | 默认 Spring Bean 作用域，一个 `BeanFactory `有且仅有一个实例 |
| prototype   | 原型作用域，每次依赖查找和依赖注入生成新 Bean 对象           |
| request     | 将 Spring Bean 存储在 `ServletRequest` 上下文中; **Only valid in the context of a web-aware Spring `ApplicationContext`.** |
| session     | 将 Spring Bean 存储在 `HttpSession` 中 ; **Only valid in the context of a web-aware Spring `ApplicationContext`.** |
| application | 将 Spring Bean 存储在 `ServletContext` 中; **Only valid in the context of a web-aware Spring `ApplicationContext`.** |
| websocket   | 将 Spring Bean 存储在 `WebSocket` 中;  **Only valid in the context of a web-aware Spring `ApplicationContext`.** |

#### `Singleton` Bean 作用域

![img](https://www.holelin.cn/img/spring/scope/singleton.png)

#### `prototype ` Bean 作用域

![img](https://www.holelin.cn/img/spring/scope/prototype.png)

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

* **注意事项**
  * Spring 容器没有办法管理 prototype Bean 的完整生命周期，也没有办法记录示例的存
    在。销毁回调方法将不会执行，可以利用 `BeanPostProcessor `进行清扫工作。  

#### `request` Bean作用域

* 配置

  * XML:`<bean id="loginAction" class="com.something.LoginAction" scope="request"/>`

  * Java注解

    * `@RequestScope`
    * `@Scope(WebApplicationContext.SCOPE_REQUEST)  `

#### `session` Bean作用域

* 配置
  * XML: `<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>`
  * Java注解
    * `@SessionScope`
    * `@Scope(WebApplicationContext.SCOPE_SESSION)`

#### `application` Bean作用域

* 配置
  * XML: `<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>`
  * Java注解
    * `@ApplicationScope`
    * `@Scope(WebApplicationContext.SCOPE_APPLICATION)`

#### 自定义Bean作用域

* 实现`Scope`

  * `org.springframework.beans.factory.config.Scope`

* 注册Scope

  * API: `org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope  `

  * XML配置:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/aop
            https://www.springframework.org/schema/aop/spring-aop.xsd">
    
        <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
            <property name="scopes">
                <map>
                    <entry key="thread">
                        <bean class="org.springframework.context.support.SimpleThreadScope"/>
                    </entry>
                </map>
            </property>
        </bean>
    
        <bean id="thing2" class="x.y.Thing2" scope="thread">
            <property name="name" value="Rick"/>
            <aop:scoped-proxy/>
        </bean>
    
        <bean id="thing1" class="x.y.Thing1">
            <property name="thing2" ref="thing2"/>
        </bean>
    
    </beans>
    ```

    

​    

​    
