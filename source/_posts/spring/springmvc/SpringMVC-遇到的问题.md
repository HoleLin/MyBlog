---
title: SpringMVC-遇到的问题
date: 2021-07-19 15:59:29
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- SpringMVC
- Tomcat
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

#### `SpringMVC`配置了编码过滤器中文依旧乱码

* 前置条件

  ```xml
  <!--配置解决中文乱码过滤器-->
    <filter>
      <filter-name>characterEncodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <!--初始化参数-->
      <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
      </init-param>
    </filter>
    <filter-mapping>
      <filter-name>characterEncodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
  ```

* 修改Tomcat的`service.xml`对应配置为:

  ```xml
  <Connector port="8080" protocol="HTTP/1.1"  
                 connectionTimeout="20000"  
                 redirectPort="8443"   
                 URIEncoding="UTF-8"  
                 useBodyEncodingForURI="true"  
                 /> 
  ```

  

  
