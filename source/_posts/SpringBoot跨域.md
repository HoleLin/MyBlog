---
title: SpringBoot中的跨域问题
date: 2021-05-26 22:14:33
tags: 
- SpringBoot
- 跨域
- Java
categories: 
- SpringBoot
- Spring
---

### SpringBoot中的跨域问题

#### 参考文献

* [Spring Boot 中实现跨域的 5 种方式，你一定要知道！](https://mp.weixin.qq.com/s/myy8NqvNeTXjTRFwD_xGcw)
* [浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
* [详解跨域(最全的解决方案)](https://www.imooc.com/article/21976)
* [CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)

#### 产生跨域问题的原因

> 主要原因: 浏览器的**同源策略(Sameorigin Policy)**
>
> * 同源策略的作用: 同源策略是一个重要的安全策略，它用于限制一个origin的文档或者它加载的脚本如何能与另一个源的资源进行交互。它能帮助阻隔恶意文档，减少可能被攻击的媒介。
> * 同源: 若两个URL的**协议(protocol)**[TCP.HTTP等],**端口(port)**和**主机(host)**[域名/IP地址等]都相同,则表示这两个URL是同源的.这个方案也被称为**“协议/主机/端口元组”**，或者直接是 “元组”。
> * **非同源限制**:
>   * 无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB;
>   * 无法接触非同源网页的 DOM;
>   * 无法向非同源地址发送 AJAX 请求;

### 跨域示例

* **跨域**: 协议(protocol),端口(port),主机(host)三者之中任意一个不同,则会导致跨域;

* 下表给出了与 URL `http://store.company.com/dir/page.html` 的源进行对比的示例:

  | URL                                               | 结果 | 原因                               |
  | :------------------------------------------------ | :--- | :--------------------------------- |
  | `http://store.company.com/dir2/other.html`        | 同源 | 只有路径不同                       |
  | `http://store.company.com/dir/inner/another.html` | 同源 | 只有路径不同                       |
  | `https://store.company.com/secure.html`           | 失败 | 协议不同                           |
  | `http://store.company.com:81/dir/etc.html`        | 失败 | 端口不同 ( `http://` 默认端口是80) |
  | `http://news.company.com/dir/other.html`          | 失败 | 主机不同                           |
###  跨域问题解决方法

#### 前端处理方法

* **JSONP跨域**

  > JSONP跨域其实也是JavaScript设计模式中的一种代理模式。在html页面中通过相应的标签从不同域名下加载静态资源文件是被浏览器允许的，所以我们可以通过这个“犯罪漏洞”来进行跨域。一般，我们可以动态的创建script标签，再去请求一个带参网址来实现跨域通信.

  ```javascript
  //原生的实现方式
  let script = document.createElement('script');
  script.src = 'http://www.nealyang.cn/login?username=Nealyang&callback=callback';
  document.body.appendChild(script);
  function callback(res) {
    console.log(res);
  }
  
  // jquery也支持jsonp的实现方式
  $.ajax({
      url:'http://www.nealyang.cn/login',
      type:'GET',
      dataType:'jsonp',//请求方式为jsonp
      jsonpCallback:'callback',
      data:{
          "username":"Nealyang"
      }
  })
  虽然这种方式非常好用，但是一个最大的缺陷是，只能够实现get请求
  ```

* **document.domain + iframe 跨域**

* **window.name + iframe 跨域**

* **location.hash + iframe 跨域**

* **postMessage跨域**

* **WebSocket协议跨域**

* **Node代理跨域**

* **Nginx代理跨域**

  * **Nginx配置解决iconfont跨域**

    ```yaml
    浏览器跨域访问js、css、img等常规静态资源被同源策略许可，但iconfont字体文件(eot|otf|ttf|woff|svg)例外，此时可在nginx的静态资源服务器中加入以下配置。
    location / {
      add_header Access-Control-Allow-Origin *;
    }
    ```

  * **Nginx反向代理接口跨域**

    > 跨域原理： 同源策略是浏览器的安全策略，不是HTTP协议的一部分。服务器端调用HTTP接口只是使用HTTP协议，不会执行JS脚本，不需要同源策略，也就不存在跨越问题。
    >
    > 实现思路：通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。

    ```
    #proxy服务器
    server {
        listen       81;
        server_name  www.domain1.com;
    
        location / {
            proxy_pass   http://www.domain2.com:8080;  #反向代理
            proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
            index  index.html index.htm;
    
            # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
            add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
            add_header Access-Control-Allow-Credentials true;
        }
    }
    ```

    

* **跨域资源共享 CORS**

  > **CORS** （Cross-Origin Resource Sharing，跨域资源共享）是一个系统，它由一系列传输的[HTTP头](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP_header)组成，这些HTTP头决定浏览器是否阻止前端 JavaScript 代码获取跨域请求的响应。
  >
  > 同源安全策略默认阻止“跨域”获取资源。但是 CORS 给了web服务器这样的权限，即服务器可以选择，允许跨域请求访问到它们的资源。
  >
  > * CORS 头
  >   * [`Access-Control-Allow-Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)
  >
  >     指示请求的资源能共享给哪些域。
  >
  >     ```
  >     Access-Control-Allow-Origin: *
  >     Access-Control-Allow-Origin: <origin>
  >     ```
  >
  >   * [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)
  >
  >     指示当请求的凭证标记为 true 时，是否响应该请求。
  >
  >     ```
  >     Access-Control-Allow-Credentials: true
  >     ```
  >
  >   * [`Access-Control-Allow-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Headers)
  >
  >     用在对预请求的响应中，指示实际的请求中可以使用哪些 HTTP 头。
  >
  >     ```
  >     Access-Control-Allow-Headers: <header-name>[, <header-name>]*
  >     Access-Control-Allow-Headers: *
  >     ```
  >
  >   * [`Access-Control-Allow-Methods`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Methods)
  >
  >     指定对预请求的响应中，哪些 HTTP 方法允许访问请求的资源。
  >
  >     ```
  >     Access-Control-Allow-Methods: <method>, <method>, ...
  >     Access-Control-Allow-Methods: POST, GET, OPTIONS
  >     ```
  >
  >   * [`Access-Control-Expose-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Expose-Headers)
  >
  >     指示哪些 HTTP 头的名称能在响应中列出。
  >
  >     ```
  >     Access-Control-Expose-Headers: <header-name>, <header-name>, ...
  >     Access-Control-Expose-Headers: Content-Length, X-Kuma-Revision
  >     ```
  >
  >     默认情况下，只有七种 [simple response headers](https://developer.mozilla.org/zh-CN/docs/Glossary/Simple_response_header) （简单响应首部）可以暴露给外部：
  >
  >     - [`Cache-Control`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
  >     - [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)
  >     - [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length)
  >     - [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)
  >     - [`Expires`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Expires)
  >     - [`Last-Modified`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified)
  >     - [`Pragma`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Pragma)
  >
  >   * [`Access-Control-Max-Age`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Max-Age)
  >
  >     指示预请求的结果能被缓存多久。
  >
  >     ```
  >     Access-Control-Max-Age: <delta-seconds>
  >     Access-Control-Max-Age: 600 
  >     ```
  >
  >   * [`Access-Control-Request-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Request-Headers)
  >
  >     用于发起一个预请求，告知服务器正式请求会使用那些 HTTP 头。
  >
  >     ```
  >     Access-Control-Request-Headers: <header-name>, <header-name>, ...
  >     Access-Control-Request-Headers: X-PINGOTHER, Content-Type
  >     ```
  >
  >   * [`Access-Control-Request-Method`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Request-Method)
  >
  >     用于发起一个预请求，告知服务器正式请求会使用哪一种 [HTTP 请求方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)。
  >
  >     ```
  >     Access-Control-Request-Method: <method>
  >     Access-Control-Request-Method: POST
  >     ```
  >
  >   * [`Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin)
  >
  >     指示获取资源的请求是从什么域发起的。
  >
  >     ```
  >     Origin: ""
  >     Origin: <scheme> "://" <host> [ ":" <port> ]
  >     Origin: https://developer.mozilla.org
  >     ```

#### Java后端处理方法

* **注意**:

  > - CorsFilter / WebMvConfigurer / @CrossOrigin 需要 SpringMVC 4.2以上版本才支持，对应springBoot 1.3版本以上
  > - 上面前两种方式属于全局 CORS 配置，后两种属于局部 CORS配置。如果使用了局部跨域是会覆盖全局跨域的规则，所以可以通过 @CrossOrigin 注解来进行细粒度更高的跨域资源控制。
  > - 其实无论哪种方案，最终目的都是修改响应头，向响应头中添加浏览器所要求的数据，进而实现跨域

* **返回新的CorsFilter(全局跨域)**

  ```java
  在任意配置类，返回一个 新的 CorsFilter Bean ，并添加映射路径和具体的CORS配置路径
  @Configuration
  public class GlobalCorsConfig {
      @Bean
      public CorsFilter corsFilter() {
          //1. 添加 CORS配置信息
          CorsConfiguration config = new CorsConfiguration();
          //放行哪些原始域
          config.addAllowedOrigin("*");
          //是否发送 Cookie
          config.setAllowCredentials(true);
          //放行哪些请求方式
          config.addAllowedMethod("*");
          //放行哪些原始请求头部信息
          config.addAllowedHeader("*");
          //暴露哪些头部信息
          config.addExposedHeader("*");
          //2. 添加映射路径
          UrlBasedCorsConfigurationSource corsConfigurationSource = new UrlBasedCorsConfigurationSource();
          corsConfigurationSource.registerCorsConfiguration("/**",config);
          //3. 返回新的CorsFilter
          return new CorsFilter(corsConfigurationSource);
      }
  }
  ```

* **重写 WebMvcConfigurer(全局跨域)**

  ```java
  @Configuration
  public class CorsConfig implements WebMvcConfigurer {
      @Override
      public void addCorsMappings(CorsRegistry registry) {
          registry.addMapping("/**")
                  //是否发送Cookie
                  .allowCredentials(true)
                  //放行哪些原始域
                  .allowedOrigins("*")
                  .allowedMethods(new String[]{"GET", "POST", "PUT", "DELETE"})
                  .allowedHeaders("*")
                  .exposedHeaders("*");
      }
  }
  ```

* **使用注解 @CrossOrigin(局部跨域)**

  ```java
  在控制器(类上)上使用注解 @CrossOrigin:，表示该类的所有方法允许跨域。
  @RestController
  @CrossOrigin(origins = "*")
  public class HelloController {
      @RequestMapping("/hello")
      public String hello() {
          return "hello world";
      }
  }
  在方法上使用注解 @CrossOrigin
  @RequestMapping("/hello")
  @CrossOrigin(origins = "*")
  //@CrossOrigin(value = "http://localhost:8081") //指定具体ip允许跨域
  public String hello() {
     return "hello world";
  }
  ```

* **手动设置响应头 (HttpServletResponse)(局部跨域)**

  ```java
  使用 HttpServletResponse 对象添加响应头(Access-Control-Allow-Origin)来授权原始域，这里 Origin的值也可以设置为 “*”,表示全部放行。
  @RequestMapping("/index")
  public String index(HttpServletResponse response) {
      response.addHeader("Access-Allow-Control-Origin","*");
      return "index";
  }
  ```

* **自定web filter 实现跨域**

  ```java
  import java.io.IOException;
  import javax.servlet.Filter;
  import javax.servlet.FilterChain;
  import javax.servlet.FilterConfig;
  import javax.servlet.ServletException;
  import javax.servlet.ServletRequest;
  import javax.servlet.ServletResponse;
  import javax.servlet.http.HttpServletResponse;
  import org.springframework.stereotype.Component;
  @Component
  public class MyCorsFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
      HttpServletResponse response = (HttpServletResponse) res;
      response.setHeader("Access-Control-Allow-Origin", "*");
      response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
      response.setHeader("Access-Control-Max-Age", "3600");
      response.setHeader("Access-Control-Allow-Headers", "x-requested-with,content-type");
      chain.doFilter(req, res);
    }
    public void init(FilterConfig filterConfig) {}
    public void destroy() {}
  }
  在web.xml中配置这个过滤器，使其生效
  
  <!-- 跨域访问 START-->
  <filter>
   <filter-name>CorsFilter</filter-name>
   <filter-class>com.mesnac.aop.MyCorsFilter</filter-class>
  </filter>
  <filter-mapping>
   <filter-name>CorsFilter</filter-name>
   <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!-- 跨域访问 END  -->
  ```

  