---
title: 网络基础-HTTP协议
date: 2021-10-20 14:27:40
index_img: /img/cover/Network.jpg
cover: /img/cover/Network.jpg
tags:
categories:
- 网络
- HTTP
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

* 编程必备基础 大话HTTP协议[慕课]
* [趣谈网络协议](https://time.geekbang.org/column/intro/85)
* WireShark数据包分析实战(第三版)
* HTTP/1.1-[RFC2616](https://datatracker.ietf.org/doc/html/rfc2616)
* [TCP-RFC793](https://www.ietf.org/rfc/rfc793.txt)
* TCP/IP详解 卷1: 协议
* 图解网络-小林coding
* [UDP-RFC768](https://www.ietf.org/rfc/rfc0768)

### HTTP

<img src="https://www.holelin.cn/img/cs-base/http/HTTP%E6%A6%82%E8%A7%88.png" alt="img" style="zoom:67%;" />

<img src="https://www.holelin.cn/img/cs-base/http/HTTP%E5%8D%8F%E8%AE%AE.png" alt="img" style="zoom:67%;" />

#### 历史

* 20世纪60年代,美国国防部高等研究计划署(ARPA)建立ARPA网,它被认为是如今互联网的始祖.
* 1989年,任职于欧洲核子研究中心(CERN)的Tim Berners-Lee发表了一篇论文,提出了在互联网上构建超链接文档系统的构想.这篇论文中确立了三项关键技术:
  * `URI`: 即统一资源标识符,作为互联网上资源的唯一身份.
  * `HTML`: 即超文本标记语言,描述超文本文档.
  * `HTTP`: 即超文本传输协议,用来传输超文本.
* HTTP/0.9,结构比较简单，为了便于服务器和客户端处理，它也采用了纯文本格式。
* HTTP/1.0,HTTP/1.0 版本在 1996 年正式发布。多方面增强了0.9版本:
  * 增加了 HEAD、POST 等新方法；
  * 增加了响应状态码，标记可能的错误原因；
  * 引入了协议版本号概念；
  * 引入了 HTTP Header（头部）的概念，让 HTTP 处理请求和响应更加灵活；
  * 传输的数据不再仅限于文本。
* HTTP/1.1,[RFC2616](https://www.rfc-editor.org/rfc/inline-errata/rfc2616.html),
  * HTTP/1.1 主要的变更点有:
    * 增加了 PUT、DELETE 等新的方法；
    * 增加了缓存管理和控制；
    * 明确了连接管理，允许持久连接；
    * 允许响应数据分块（chunked），利于传输大文件；
    * 强制要求 Host 头，让互联网主机托管成为可能。
  * 不过由于 HTTP/1.1 太过庞大和复杂，所以在 2014 年又做了一次修订，原来的一个大文档被拆分成了六份较小的文档，编号为 7230-7235，优化了一些细节，但此外没有任何实质性的改动。
* HTTP/2,HTTP/2 的制定充分考虑了现今互联网的现状：宽带、移动、不安全，在高度兼容 HTTP/1.1 的同时在性能改善方面做了很大努力, 基于 Google 的 SPDY 协议.
  * 主要的特点有:
    * 二进制协议，不再是纯文本；
    * 可发起多个请求，废弃了 1.1 里的管道；
    * 使用专用算法压缩头部，减少数据传输量；
    * 允许服务器主动向客户端推送数据；
    * 增强了安全性，“事实上”要求加密通信。
* HTTP/3,2018 年，互联网标准化组织 IETF 提议将“HTTP over QUIC”更名为“HTTP/3”并获得批准，HTTP/3 正式进入了标准化制订阶段,基于 Google 的 QUIC 协议

#### HTTP协议概述

* 超文本传输协议(HTTP)是一种**通信协议**,它允许将超文本标记语言(HTML)文档从Web服务器传送到客户端的浏览器;

  > 超文本传输协议＝超文本＋传输＋协议，协议即约定，HTTP就是约定超文本怎么传输的。初心就是分享信息，所以，简单、开放、有求有应，只针对文本，后来出现了音频、视频、动画、图片、超链接这些玩意，比纯文本复杂了一些，不过初心不改，所以，原则未变，只是需要调整一下适应这些正当其时的需求而已。

* HTTP是一个属于**应用层的面向对象的协议**,由于其简洁,快速的方式,适用于分布式超媒体信息系统,它于1990年提出,经过几年的使用与发展,得到不断地完善和扩展;

* HTTP是一个在计算机世界里面专门在**两点之间传输**文字,图片,音频,视频等**超文本**数据的**约定和规范**

* 在HTTP协议里,浏览器的角色被称为"User Agent"即"用户代理",意思是作为访问者的"代理"来发我HTTP请求.

* CDN(**Content Delivery Network**): 即内容分发网络,它应用了HTTP协议里的缓存和代理技术,代替源站响应客户端的读请求.

  * 好处: 它可以缓存源站的数据,让浏览器的请求不用"千里迢迢"地到达源站服务器,直接在半路就可以获取响应.
  * CDN是互联网中的一项重要基础设施,除了基本的网络加速外,还提供负载均衡,安全防护,边缘计算,跨运营商网络等功能,能够成倍地放大源站服务器的服务能力.

* Web  Service 是一种由 W3C 定义的应用服务开发规范，使用 client-server 主从架构，通常使用 WSDL 定义服务接口，使用 HTTP 协议传输 XML 或 SOAP 消息，也就是说，它是**一个基于 Web（HTTP）的服务架构技术**，既可以运行在内网，也可以在适当保护后运行在外网。

* **WAF**意思是“网络应用防火墙”。与硬件“防火墙”类似，它是应用层面的“防火墙”，专门检测 HTTP 流量，是防护 Web 应用的安全技术。

  * WAF 通常位于 Web 服务器之前，可以阻止如 SQL 注入、跨站脚本等攻击，目前应用较多的一个开源项目是 ModSecurity，它能够完全集成进 Apache 或 Nginx。

#### [状态码](https://datatracker.ietf.org/doc/html/rfc2616#page-57)

* 状态码是十进制的三位数，分为五类，从 100 到 599；

|      | 具体含义                                                | 常见的状态码                                                 |
| ---- | ------------------------------------------------------- | ------------------------------------------------------------ |
| 1xx  | 提示信息,,表示目前是协议处理的中间状态,还需要后续的操作 | 100[Continue],101[Switching Protocols]                       |
| 2xx  | 成功,报文已经收到并正确处理                             | 200[OK],204[Not Content],206[Partial Conent]                 |
| 3xx  | 重定向,资源位置发生变动,需要客户端重新发送请求          | 301[Moved Permantly],302[Foud],304[Not Modified]             |
| 4xx  | 客户端错误,请求报文有误,服务器无法处理                  | 400[Bad Request],403[Forbidden],404[Not Found]               |
| 5xx  | 服务器错误,服务器在处理请求时内部发生了错误             | 500[Internal Service Error],501[Not Implemented],502[Bad Gateway],503[Service Unavailable] |

* **101 Switching Protocols**:它的意思是客户端使用 Upgrade 头字段，要求在 HTTP 协议的基础上改成其他的协议继续通信，比如 WebSocket。而如果服务器也同意变更协议，就会发送状态码 101，但这之后的数据传输就不会再使用 HTTP 了。
* **204 No Content**: 另一个很常见的成功状态码，它的含义与“200 OK”基本相同，但响应头后没有 body 数据。所以对于 Web 服务器来说，正确地区分 200 和 204 是很必要的。
* **206 Partial Conent**: 是 HTTP 分块下载或断点续传的基础，在客户端发送“范围请求”、要求获取资源的部分数据时出现，它与 200 一样，也是服务器成功处理了请求，但 body 里的数据不是资源的全部，而是其中的一部分。
  * 状态码 206 通常还会伴随着头字段“**Content-Range**”，表示响应报文里 body 数据的具体范围，供客户端确认，例如“Content-Range: bytes 0-99/2000”，意思是此次获取的是总计 2000 个字节的前 100 个字节。

* **301 Moved Permantly**: 表示永久重定向,说明请求的资源已经不存在了,需要改用新的URL再次访问.
* **302 Foud**: 表示临时重定向,说明请求的资源还在,但暂时需要用另一个URI来访问.
  * 301 和 302 都会在响应头里使用字段 **Location** 指明后续要跳转的 URI，最终的效果很相似，浏览器都会重定向到新的 URI。两者的根本区别在于语义，一个是“永久”，一个是“临时”，所以在场景、用法上差距很大。

* **304 Not Modified**: 是一个比较有意思的状态码，它用于 If-Modified-Since 等条件请求，表示资源未修改，用于缓存控制。它不具有通常的跳转含义，但可以理解成“重定向已到缓存的文件”（即“缓存重定向”）。
* **400 Bad Request**: 是一个通用的错误码，表示请求报文有错误，但具体是数据格式错误、缺少请求头还是 URI 超长它没有明确说，只是一个笼统的错误，客户端看到 400 只会是“一头雾水”“不知所措”。所以，在开发 Web 应用时应当尽量避免给客户端返回 400，而是要用其他更有明确含义的状态码。
* **403 Forbidden**: 实际上不是客户端的请求出错，而是表示服务器禁止访问资源。原因可能多种多样，例如信息敏感、法律禁止等，如果服务器友好一点，可以在 body 里详细说明拒绝请求的原因，不过现实中通常都是直接给一个“闭门羹”
* **404 Not Found**: 它的原意是资源在本服务器上未找到，所以无法提供给客户端。但现在已经被“用滥了”，只要服务器“不高兴”就可以给出个 404，而我们也无从得知后面到底是真的未找到，还是有什么别的原因，某种程度上它比 403 还要令人讨厌。
* **405 Method Not Allowed**：不允许使用某些方法操作资源，例如不允许 POST 只能 GET；
* **406 Not Acceptable**：资源无法满足客户端请求的条件，例如请求中文但只有英文；
* **408 Request Timeout**：请求超时，服务器等待了过长的时间；
* **409 Conflict**：多个请求发生了冲突，可以理解为多线程并发时的竞态；
* **413 Request Entity Too Large**：请求报文里的 body 太大；
* **414 Request-URI Too Long**：请求行里的 URI 太大；
* **429 Too Many Requests**：客户端发送了太多的请求，通常是由于服务器的限连策略；
* **431 Request Header Fields Too Large**：请求头某个字段或总体太大；
* **500 Internal Server Error**: 与 400 类似，也是一个通用的错误码，服务器究竟发生了什么错误我们是不知道的。不过对于服务器来说这应该算是好事，通常不应该把服务器内部的详细信息，例如出错的函数调用栈告诉外界。虽然不利于调试，但能够防止黑客的窥探或者分析。
* **501 Not Implemented**: 表示客户端请求的功能还不支持，这个错误码比 500 要“温和”一些，和“即将开业，敬请期待”的意思差不多，不过具体什么时候“开业”就不好说了。
* **502 Bad Gateway**: 通常是服务器作为网关或者代理时返回的错误码，表示服务器自身工作正常，访问后端服务器时发生了错误，但具体的错误原因也是不知道的。
* **503 Service Unavailable**:表示服务器当前很忙，暂时无法响应服务，我们上网时有时候遇到的“网络服务正忙，请稍后重试”的提示信息就是状态码 503。
  * 503 是一个“临时”的状态，很可能过几秒钟后服务器就不那么忙了，可以继续提供服务，所以 503 响应报文里通常还会有一个“**Retry-After**”字段，指示客户端可以在多久以后再次尝试发送请求。


#### 报文结构

##### [请求结构](https://datatracker.ietf.org/doc/html/rfc2616#section-5)

```
        //GET / HTTP/1.1
        Request-Line   = Method SP Request-URI SP HTTP-Version CRLF
 				Request        = Request-Line             ; 
                        *(( general-header        ; 
                         | request-header         ;
                         | entity-header ) CRLF)  ;
                        CRLF
                        [ message-body ]          ; 
```

##### [响应结构](https://datatracker.ietf.org/doc/html/rfc2616#section-6)

```
			 //HTTP/1.1 200 OK
			 Status-Line    = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
			 Response       = Status-Line              ; 
                       *(( general-header        ; 
                        | response-header        ; 
                        | entity-header ) CRLF)  ; 
                       CRLF
                       [ message-body ]          ;
```

* 请求头和响应头的结构是基本一样的，唯一的区别是起始行，所以我把请求头和响应头里的字段放在一起介绍。
* 头部字段是 key-value 的形式，key 和 value 之间用“:”分隔，最后用 CRLF 换行表示字段结束。
* 使用头字段需要注意下面几点:
  * 字段名不区分大小写，例如“Host”也可以写成“host”，但首字母大写的可读性更好；
  * 字段名里不允许出现空格，可以使用连字符“-”，但不能使用下划线“_”。例如，“test-name”是合法的字段名，而“test name”“test_name”是不正确的字段名；
  * 字段名后面必须紧接着“:”，不能有空格，而“:”后的字段值前可以有多个空格；
  * 字段的顺序是没有意义的，可以任意排列不影响语义；
  * 字段原则上不能重复，除非这个字段本身的语义允许，例如 Set-Cookie。

#### 常用头字段

* HTTP 协议规定了非常多的头部字段，实现各种各样的功能，但基本上可以分为四大类：

  * [通用字段](https://datatracker.ietf.org/doc/html/rfc2616#section-4.5)：在请求头和响应头里都可以出现；

    ```
    			 general-header = Cache-Control            ; Section 14.9
                          | Connection               ; Section 14.10
                          | Date                     ; Section 14.18
                          | Pragma                   ; Section 14.32
                          | Trailer                  ; Section 14.40
                          | Transfer-Encoding        ; Section 14.41
                          | Upgrade                  ; Section 14.42
                          | Via                      ; Section 14.45
                          | Warning                  ; Section 14.46
    ```

  * [请求字段](https://datatracker.ietf.org/doc/html/rfc2616#section-5.3)：仅能出现在请求头里，进一步说明请求信息或者额外的附加条件；

    ```
           request-header = Accept                   ; Section 14.1
                          | Accept-Charset           ; Section 14.2
                          | Accept-Encoding          ; Section 14.3
                          | Accept-Language          ; Section 14.4
                          | Authorization            ; Section 14.8
                          | Expect                   ; Section 14.20
                          | From                     ; Section 14.22
                          | Host                     ; Section 14.23
                          | If-Match                 ; Section 14.24
                          | If-Modified-Since        ; Section 14.25
                          | If-None-Match            ; Section 14.26
                          | If-Range                 ; Section 14.27
                          | If-Unmodified-Since      ; Section 14.28
                          | Max-Forwards             ; Section 14.31
                          | Proxy-Authorization      ; Section 14.34
                          | Range                    ; Section 14.35
                          | Referer                  ; Section 14.36
                          | TE                       ; Section 14.39
                          | User-Agent               ; Section 14.43
    ```

  * [响应字段](https://datatracker.ietf.org/doc/html/rfc2616#section-6.2)：仅能出现在响应头里，补充说明响应报文的信息；

    ```
           response-header = Accept-Ranges           ; Section 14.5
                           | Age                     ; Section 14.6
                           | ETag                    ; Section 14.19
                           | Location                ; Section 14.30
                           | Proxy-Authenticate      ; Section 14.33
                           | Retry-After             ; Section 14.37
                           | Server                  ; Section 14.38
                           | Vary                    ; Section 14.44
                           | WWW-Authenticate        ; Section 14.47
    ```

  * [实体字段](https://datatracker.ietf.org/doc/html/rfc2616#section-7.1)：它实际上属于通用字段，但专门描述 body 的额外信息。

    ```
           entity-header  = Allow                    ; Section 14.7
                          | Content-Encoding         ; Section 14.11
                          | Content-Language         ; Section 14.12
                          | Content-Length           ; Section 14.13
                          | Content-Location         ; Section 14.14
                          | Content-MD5              ; Section 14.15
                          | Content-Range            ; Section 14.16
                          | Content-Type             ; Section 14.17
                          | Expires                  ; Section 14.21
                          | Last-Modified            ; Section 14.29
                          | extension-header
    
           extension-header = message-header
    ```

#### 请求方法

* 目前 HTTP/1.1 规定了八种方法，单词**都必须是大写的形式**

  | flag      | 说明                                   |
  | --------- | -------------------------------------- |
  | `GET`     | 获取资源，可以理解为读取或者下载数据； |
  | `HEAD`    | 获取资源的元信息；                     |
  | `POST`    | 向资源提交数据，相当于写入或上传数据； |
  | `PUT`     | 类似 POST；                            |
  | `DELETE`  | 删除资源；                             |
  | `CONNECT` | 建立特殊的连接隧道；                   |
  | `OPTIONS` | 列出可对资源实行的方法；               |
  | `TRACE`   | 追踪请求 - 响应的传输路径。            |

##### 安全和幂等

* **安全**: 是指请求方法不会“破坏”服务器上的资源，即不会对服务器上的资源造成实质的修改。
  * 按照这个定义，只有 GET 和 HEAD 方法是“安全”的，因为它们是“只读”操作，只要服务器不故意曲解请求方法的处理方式，无论 GET 和 HEAD 操作多少次，服务器上的数据都是“安全的”。
* “**幂等**”实际上是一个数学用语，被借用到了 HTTP 协议里，意思是多次执行相同的操作，结果也都是相同的，即多次“幂”后结果“相等”。
  * GET 和 HEAD 既是安全的也是幂等的，DELETE 可以多次删除同一个资源，效果都是“资源不存在”，所以也是幂等的。
  * 按照 RFC 里的语义，POST 是“新增或提交数据”，多次提交数据会创建多个资源，所以不是幂等的；而 PUT 是“替换或更新数据”，多次更新一个资源，资源还是会第一次更新的状态，所以是幂等的。

#### [URI](https://www.ietf.org/rfc/rfc3986.html)

* Uniform Resource Identifiers

```
  		   http_URL = "http:" "//" host [ ":" port ] [ abs_path [ "?" query ]]
         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
          |   _____________________|__
         / \ /                        \
         urn:example:animal:ferret:nose
```

* 在“://”之后，是被称为“**authority**”的部分，表示**资源所在的主机名**，通常的形式是“host:port”，即主机名加端口号。
* URI 的 path 部分必须以“/”开始，也就是必须包含“/”，不要把“/”误认为属于前面 authority。

##### URI 的完整格式

```
   scheme://user:passwd@host:port/path?query#fragment
```

* 第一个多出的部分是协议名之后、主机名之前的**身份信息**“user:passwd@”，表示登录主机时的用户名和密码，但现在已经不推荐使用这种形式了（RFC7230），因为它把敏感信息以明文形式暴露出来，存在严重的安全隐患。
* 第二个多出的部分是查询参数后的**片段标识符**“#fragment”，它是 URI 所定位的资源内部的一个“锚点”或者说是“标签”，浏览器可以在获取资源后直接跳转到它指示的位置。
  * 但片段标识符仅能由浏览器这样的客户端使用，服务器是看不到的。也就是说，浏览器永远不会把带“#fragment”的 URI 发送给服务器，服务器也永远不会用这种方式去处理资源的片段。

##### URI 的编码

* URI 里只能使用 ASCII 码,若要在 URI 里使用英语以外的汉语、日语等其他语言则需要进行转义.

* URI 转义的规则有点“简单粗暴”，直接把非 ASCII 码或特殊字符转换成十六进制字节值，然后前面再加上一个“%”。

  ```
        pct-encoded = "%" HEXDIG HEXDIG
  ```

#### 跨域资源共享

* 跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。
* 在现在前端最常用的 cors 跨域中，浏览器都是用 OPTIONS 方法发预检请求的

#### 实体数据

* 数据类型表示实体数据的内容是什么，使用的是 MIME type，相关的头字段是 Accept 和 Content-Type；
* 数据编码表示实体数据的压缩方式，相关的头字段是 Accept-Encoding 和 Content-Encoding；
* 语言类型表示实体数据的自然语言，相关的头字段是 Accept-Language 和 Content-Language；
* 字符集表示实体数据的编码方式，相关的头字段是 Accept-Charset 和 Content-Type；
* 客户端需要在请求头里使用 Accept 等头字段与服务器进行“内容协商”，要求服务器返回最合适的数据；
* Accept 等头字段可以用“,”顺序列出多个可能的选项，还可以用“;q=”参数来精确指定权重。

####  HTTP数据传输过程

* **数据封装(Data Encapsulation)**是将**协议数据单元**(PDU)封装在一组协议头和尾中过程.	

![img](https://www.holelin.cn/img/cs-base/http/HTTP%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E8%BF%87%E7%A8%8B.png)

* 发送端发送数据时,数据会从上层传输到下层,且每经过一层都会被打上该层的头部信息.而接收端接收数据时,数据会从下层传输到上层,传输前会把下层的头部信息删除.
* 数据发送处理过程
  * 应用层将数据交给传输层,传输层添加上TCP的控制信息(称为TCP头部),这个数据单元称为段(Segment),加上控制信息的过程称为封装.然后将段交给网络层.
  * 网络层接收到段,再添加上IP头部,这个数据单元称为包(Packet).然后,将包交给数据链路层.
  * 数据链路层接收到包,再添加上MAC头部和尾部,这个数据单元称为帧(Frame).然后,将帧交给物理层.
  * 物理层将接收到的数据转化为比特流,然后在网线中传输.
* 数据接收处理过程
  * 物理层接收到比特流,经过处理后将数据交给数据链路层.
  * 数据链路层将接收到数据转化为数据帧,再除去MAC头部和尾部,这个除去控制信息的过程称为解封装,然后将包交给网络层.
  * 网络层接收到包,再除去IP头部,然后将段交给传输层.
  * 传输层接收到段,再除去TCP头部,然后将数据交给应用层.

#### HTTP连接

##### 短连接

* HTTP 协议最初（0.9/1.0）是个非常简单的协议，通信过程也采用了简单的“请求 - 应答”方式。
* 它底层的数据传输基于 TCP/IP，每次发送请求前需要先与服务器建立连接，收到响应报文后会立即关闭连接。
* 因为客户端与服务器的整个连接过程很短暂，不会与服务器保持长时间的连接状态，所以就被称为“**短连接**”（short-lived connections）。早期的 HTTP 协议也被称为是“**无连接**”的协议。
* 短连接的缺点相当严重，因为在 TCP 协议里，建立连接和关闭连接都是非常“昂贵”的操作。TCP 建立连接要有“三次握手”，发送 3 个数据包，需要 1 个 RTT；关闭连接是“四次挥手”，4 个数据包需要 2 个 RTT。

##### 长连接

* 针对短连接暴露出的缺点，HTTP 协议就提出了“**长连接**”的通信方式，也叫“持久连接”（persistent connections）、“连接保活”（keep alive）、“连接复用”（connection reuse）。
* 其实解决办法也很简单，用的就是“**成本均摊**”的思路，既然 TCP 的连接和关闭非常耗时间，那么就把这个时间成本由原来的一个“请求 - 应答”均摊到多个“请求 - 应答”上。
* 这样虽然不能改善 TCP 的连接效率，但基于“**分母效应**”，每个“请求 - 应答”的无效时间就会降低不少，整体传输效率也就提高了。
* 由于长连接对性能的改善效果非常显著，所以在 HTTP/1.1 中的连接都会默认启用长连接。不需要用什么特殊的头字段指定，只要向服务器发送了第一次请求，后续的请求都会重复利用第一次打开的 TCP 连接，也就是长连接，在这个连接上收发数据。
* 当然，我们也可以在请求头里明确地要求使用长连接机制，使用的字段是 **Connection**，值是“**keep-alive**”。
* 不过不管客户端是否显式要求长连接，如果服务器支持长连接，它总会在响应报文里放一个“**Connection: keep-alive**”字段，告诉客户端：“我是支持长连接的，接下来就用这个 TCP 一直收发数据吧”。

##### 长连接带来的问题

* 因为 TCP 连接长时间不关闭，服务器必须在内存里保存它的状态，这就占用了服务器的资源。如果有大量的空闲长连接只连不发，就会很快耗尽服务器的资源，导致服务器无法为真正有需要的用户提供服务。
* 所以，长连接也需要在恰当的时间关闭，不能永远保持与服务器的连接，这在客户端或者服务器都可以做到。
* 在客户端，可以在请求头里加上“**Connection: close**”字段，告诉服务器：“这次通信后就关闭连接”。服务器看到这个字段，就知道客户端要主动关闭连接，于是在响应报文里也加上这个字段，发送之后就调用 Socket API 关闭 TCP 连接。
* 服务器端通常不会主动关闭连接，但也可以使用一些策略。拿 Nginx 来举例，它有两种方式：
  * 使用“**keepalive_timeout**”指令，设置长连接的超时时间，如果在一段时间内连接上没有任何数据收发就主动断开连接，避免空闲连接占用系统资源。
  * 使用“**keepalive_requests**”指令，设置长连接上可发送的最大请求次数。比如设置成 1000，那么当 Nginx 在这个连接上处理了 1000 个请求后，也会主动断开连接。
* 另外，客户端和服务器都可以在报文里附加通用头字段“Keep-Alive: timeout=value”，限定长连接的超时时间。但这个字段的约束力并不强，通信的双方可能并不会遵守，所以不太常见。

##### 总结

* 服务器端设置keepalive_timeout表示多长时间没有数据则关闭连接。
* 服务器端设置keepalive_requests，表示该连接上处理多少个请求后关闭连接。
* 服务器端设置最大连接数，当连接达到上限之后拒绝连接，也可以采用限流措施等。
* 客户端设置keepalive_requests，表示该连接上发送多少个连接后关闭连接。
* 客户端设置keepalive_timeout，表示多长时间没有数据发送则关闭连接。
* 客户端设置响应超时后重试次数，当次数达到上限后关闭连接。

##### 队头阻塞

* “队头阻塞”（Head-of-line blocking，也叫“队首阻塞”.“队头阻塞”与短连接和长连接无关，而是由 HTTP 基本的“请求 - 应答”模型所导致的。
* 因为 HTTP 规定报文必须是“一发一收”，这就形成了一个先进先出的“串行”队列。队列里的请求没有轻重缓急的优先级，只有入队的先后顺序，排在最前面的请求被最优先处理。
* 如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本。

#### HTTP的优点

* HTTP 是灵活可扩展的，可以任意添加头字段实现任意功能；
* HTTP 是可靠传输协议，基于 TCP/IP 协议“尽量”保证数据的送达；
* HTTP 是应用层协议，比 FTP、SSH 等更通用功能更多，能够传输任意数据；
* HTTP 使用了请求 - 应答模式，客户端主动发起请求，服务器被动回复请求；
* HTTP 本质上是无状态的，每个请求都是互相独立、毫无关联的，协议不要求客户端或服务器记录请求相关的信息。

#### HTTP缺点

* HTTP 是无状态的，可以轻松实现集群化，扩展性能，但有时也需要用 Cookie 技术来实现“有状态”；
  * HTTP的无状态会导致服务器没有"记忆能力",就无法支持需要连续多个步骤的“事务”操作。
    * 例如电商购物，首先要登录，然后添加购物车，再下单、结算、支付，这一系列操作都需要知道用户的身份才行，但“无状态”服务器是不知道这些请求是相互关联的，每次都得问一遍身份信息，不仅麻烦，而且还增加了不必要的数据传输量。
* HTTP 是明文传输，数据完全肉眼可见，能够方便地研究分析，但也容易被窃听；
* HTTP 是不安全的，无法验证通信双方的身份，也不能判断报文是否被窜改；
* HTTP 的性能不算差，但不完全适应现在的互联网，还有很大的提升空间。

#### HTTP的重定向和跳转

* 由浏览器的使用者主动发起的被称为"**主动跳转**";
* 由服务器来发起的,浏览器使用者无法控制,相对地就可以被称为"**被动跳转**",这种跳转在HTTP协议中被称为"**重定向**"
* 重定向是"用户无感"的.
* "**Location**"字段属于响应字段,必须出现在响应报文中.但只有配合301/302状态码才有意义,它**标记了服务器要求重定向的URI**
  * 浏览器收到 301/302 报文，会检查响应头里有没有“Location”。如果有，就从字段值里提取出 URI，发出新的 HTTP 请求，相当于自动替我们点击了这个链接。
  * 在“Location”里的 URI 既可以使用绝对 URI，也可以使用相对 URI。所谓“绝对 URI”，就是完整形式的 URI，包括 scheme、host:port、path 等。所谓“相对 URI”，就是省略了 scheme 和 host:port，只有 path 和 query 部分，是不完整的，但可以从请求上下文里计算得到。
  * 注意，在重定向时如果只是在站内跳转，可以放心地使用相对 URI。但如果要跳转到站外，就必须用绝对 URI。

##### 重定向应用场景

* 最常见的原因就是“**资源不可用**”，需要用另一个新的 URI 来代替。
  * 如域名变更、服务器变更、网站改版、系统维护，这些都会导致原 URI 指向的资源无法访问，为了避免出现 404，就需要用重定向跳转到新的 URI，继续为网民提供服务。
* 另一个原因就是“**避免重复**”，让多个网址都跳转到一个 URI，增加访问入口的同时还不会增加额外的工作量。

##### 重定向的相关问题

* 第一个问题是“**性能损耗**”。很明显，重定向的机制决定了一个跳转会有两次请求 - 应答，比正常的访问多了一次。
  * 虽然 301/302 报文很小，但大量的跳转对服务器的影响也是不可忽视的。站内重定向还好说，可以长连接复用，站外重定向就要开两个连接，如果网络连接质量差，那成本可就高多了，会严重影响用户的体验。
* 第二个问题是“**循环跳转**”。如果重定向的策略设置欠考虑，可能会出现“A=>B=>C=>A”的无限循环，不停地在这个链路里转圈圈，后果可想而知.
  * 所以 HTTP 协议特别规定，浏览器必须具有检测“循环跳转”的能力，在发现这种情况时应当停止发送请求并给出错误提示。

##### 外部重定向与内部重定向

* 外部重定向，服务器会把重定向的地址给浏览器，然后浏览器再次的发起请求，地址栏的地址变化了。
* 内部重定向，服务器会直接把重定向的资源返给浏览器，不需要再次在浏览器发起请求，地址栏的地址不变。

#### HTTP的Cookie机制

##### Cookie的工作过程

* 响应头字段Set-Cookie和请求头字段Cookie

  > * 当用户通过浏览器第一次访问服务器的时候，服务器肯定是不知道他的身份的。所以，就要创建一个独特的身份标识数据，格式是“**key=value**”，然后放进 Set-Cookie 字段里，随着响应报文一同发给浏览器。
  >
  > * 浏览器收到响应报文，看到里面有 Set-Cookie，知道这是服务器给的身份标识，于是就保存起来，下次再请求的时候就自动把这个值放进 Cookie 字段里发给服务器。
  > * 因为第二次请求里面有了 Cookie 字段，服务器就知道这个用户不是新人，之前来过，就可以拿出 Cookie 里的值，识别出用户的身份，然后提供个性化的服务。
  > * 不过因为服务器的“记忆能力”实在是太差，一张小纸条经常不够用。所以，服务器有时会在响应头里添加多个 Set-Cookie，存储多个“key=value”。但浏览器这边发送时不需要用多个 Cookie 字段，只要在一行里用“;”隔开就行。

* Cookie 是由浏览器负责存储的，而不是操作系统。所以，它是“浏览器绑定”的，只能在本浏览器内生效。

##### Cookie的属性

* Cookie 就是服务器委托浏览器存储在客户端里的一些数据，而这些数据通常都会记录用户的关键识别信息。所以，就需要在“key=value”外再用一些手段来保护，防止外泄或窃取，这些手段就是 Cookie 的属性。

###### 设置Cookie的生存周期

* Cookie 的有效期可以使用 **Expires** 和 **Max-Age** 两个属性来**设置Cookie的生存周期**。

  * “**Expires**”俗称“过期时间”，用的是绝对时间点，可以理解为“截止日期”（deadline）。“**Max-Age**”用的是相对时间，单位是秒，浏览器用收到报文的时间点再加上 Max-Age，就可以得到失效的绝对时间。
  * Expires 和 Max-Age 可以同时出现，两者的失效时间可以一致，也可以不一致，但浏览器会优先采用 Max-Age 计算失效期。

###### 设置 Cookie 的作用域

* 设置 Cookie 的作用域可以让浏览器仅发送给特定的服务器和 URI，避免被其他网站盗用。
* 作用域的设置比较简单，“**Domain**”和“**Path**”指定了 Cookie 所属的域名和路径，浏览器在发送 Cookie 前会从 URI 中提取出 host 和 path 部分，对比 Cookie 的属性。如果不满足条件，就不会在请求头里发送 Cookie。
* 使用这两个属性可以为不同的域名和路径分别设置各自的 Cookie.

###### Cookie 的安全性

* 在 JS 脚本里可以用 document.cookie 来读写 Cookie 数据，这就带来了安全隐患，有可能会导致“跨站脚本”（XSS）攻击窃取数据。

* 属性“**HttpOnly**”会告诉浏览器，此 Cookie 只能通过浏览器 HTTP 协议传输，禁止其他方式访问，浏览器的 JS 引擎就会禁用 document.cookie 等一切相关的 API，脚本攻击也就无从谈起了。
* 另一个属性“**SameSite**”可以防范“跨站请求伪造”（XSRF）攻击，设置成“SameSite=Strict”可以严格限定 Cookie 不能随着跳转链接跨站发送，而“SameSite=Lax”则略宽松一点，允许 GET/HEAD 等安全方法，但禁止 POST 跨站发送。
* 还有一个属性叫“**Secure**”，表示这个 Cookie 仅能用 HTTPS 协议加密传输，明文的 HTTP 协议会禁止发送。但 Cookie 本身不是加密的，浏览器里还是以明文的形式存在。

##### Cooike的应用

* Cookie 最基本的一个用途就是**身份识别**，保存用户的登录信息，实现会话事务。
* Cookie 的另一个常见用途是**广告跟踪**。
  * 这种 Cookie 不是由访问的主站存储的，所以又叫“第三方 Cookie”（third-party cookie）。
  * 为了防止滥用 Cookie 搜集用户隐私，互联网组织相继提出了 DNT（Do Not Track）和 P3P（Platform for Privacy Preferences Project），但实际作用不大。
*  Cookie 并不属于 HTTP 标准（RFC6265，而不是 RFC2616/7230）

#### HTTP的缓存

* 服务器标记资源有效期使用的头字段是“**Cache-Control**”，里面的值“**max-age=30**”就是资源的有效时间，相当于告诉浏览器，“这个页面只能缓存 30 秒，之后就算是过期，不能用。”
  * 这里的 max-age 是“**生存时间**”（又叫“新鲜度”“缓存寿命”，类似 TTL，Time-To-Live），时间的计算起点是响应报文的创建时刻（即 Date 字段，也就是离开服务器的时刻），而不是客户端收到报文的时刻，也就是说包含了在链路传输过程中所有节点所停留的时间。
  * 比如，服务器设定“max-age=5”，但因为网络质量很糟糕，等浏览器收到响应报文已经过去了 4 秒，那么这个资源在客户端就最多能够再存 1 秒钟，之后就会失效。

* “max-age”是 HTTP 缓存控制最常用的属性，此外在响应报文里还可以用其他的属性来更精确地指示浏览器应该如何使用缓存:
  * no-store：**不允许缓存**，用于某些变化非常频繁的数据，例如秒杀页面；
  * no-cache：它的字面含义容易与 no-store 搞混，实际的意思并不是不允许缓存，而是**可以缓存**，但在使用之前必须要去服务器验证是否过期，是否有最新的版本；
  * must-revalidate：又是一个和 no-cache 相似的词，它的意思是如果缓存不过期就可以继续使用，但过期了如果还想用就必须去服务器验证。
* 当点“刷新”按钮的时候，浏览器会在请求头里加一个“**Cache-Control: max-age=0**”。因为 max-age 是“**生存时间**”，max-age=0 的意思就是“我要一个最最新鲜的西瓜”，而本地缓存里的数据至少保存了几秒钟，所以浏览器就不会使用缓存，而是向服务器发请求。服务器看到 max-age=0，也就会用一个最新生成的报文回应浏览器。
* Ctrl+F5 的“强制刷新”:它其实是发了一个“**Cache-Control: no-cache**”，含义和“max-age=0”基本一样，就看后台的服务器怎么理解，通常两者的效果是相同的。

### 实例

#### HTTP传输大文件的方法

##### 数据压缩

* 通常浏览器在发送请求时都会带着“**Accept-Encoding**”头字段，里面是浏览器支持的压缩格式列表，例如 gzip、deflate、br 等，这样服务器就可以从中选择一种压缩算法，放进“**Content-Encoding**”响应头里，再把原数据压缩后发给浏览器。
* 不过这个解决方法也有个缺点，gzip 等压缩算法通常只对文本文件有较好的压缩率，而图片、音频视频等多媒体数据本身就已经是高度压缩的，再用 gzip 处理也不会变小（甚至还有可能会增大一点），所以它就失效了。
* 在 Nginx 里就会使用“gzip on”指令，启用对“text/html”的压缩。

##### 分块传输

* 在 HTTP 协议里就是“**chunked**”分块传输编码，在响应报文里用头字段“**Transfer-Encoding: chunked**”来表示，意思是报文里的 body 部分不是一次性发过来的，而是分成了许多的块（chunk）逐个发送。
* “Transfer-Encoding: chunked”和“Content-Length”这两个字段是**互斥的**，也就是说响应报文里这两个字段不能同时出现，一个响应报文的传输要么是长度已知，要么是长度未知（chunked）

```
       Chunked-Body   = *chunk
                        last-chunk
                        trailer
                        CRLF

       chunk          = chunk-size [ chunk-extension ] CRLF
                        chunk-data CRLF
       chunk-size     = 1*HEX
       last-chunk     = 1*("0") [ chunk-extension ] CRLF

       chunk-extension= *( ";" chunk-ext-name [ "=" chunk-ext-val ] )
       chunk-ext-name = token
       chunk-ext-val  = token | quoted-string
       chunk-data     = chunk-size(OCTET)
       trailer        = *(entity-header CRLF)
```

##### 范围请求

* 范围请求允许客户端在请求头里使用专用字段来表示只获取文件的一部分，相当于是**客户端的“化整为零”**

* 范围请求不是 Web 服务器必备的功能，可以实现也可以不实现，所以服务器必须在响应头里使用字段“**Accept-Ranges: bytes**”明确告知客户端：“我是支持范围请求的”。

* 如果不支持的话该怎么办呢？服务器可以发送“Accept-Ranges: none”，或者干脆不发送“Accept-Ranges”字段，这样客户端就认为服务器没有实现范围请求功能，只能老老实实地收发整块文件了。

* 请求头 **Range** 是 HTTP 范围请求的专用字段，格式是“**bytes=x-y**”，其中的 x 和 y 是以字节为单位的数据范围。

  * 要注意 x、y 表示的是“偏移量”，范围必须从 0 计数，例如前 10 个字节表示为“0-9”，第二个 10 字节表示为“10-19”，而“0-10”实际上是前 11 个字节。
  * Range 的格式也很灵活，起点 x 和终点 y 可以省略，能够很方便地表示正数或者倒数的范围。假设文件是 100 个字节，那么：
    * “0-”表示从文档起点到文档终点，相当于“0-99”，即整个文件；
    * “10-”是从第 10 个字节开始到文档末尾，相当于“10-99”；
    * “-1”是文档的最后一个字节，相当于“99-99”；
    * “-10”是从文档末尾倒数 10 个字节，相当于“90-99”

* 服务器收到 Range 字段后，需要做四件事。

  * 第一，它必须检查范围是否合法，比如文件只有 100 个字节，但请求“200-300”，这就是范围越界了。服务器就会返回状态码 **416**，意思是“你的范围请求有误，我无法处理，请再检查一下”。

  * 第二，如果范围正确，服务器就可以根据 Range 头计算偏移量，读取文件的片段了，返回状态码“**206 Partial Content**”，和 200 的意思差不多，但表示 body 只是原数据的一部分。

  * 第三，服务器要添加一个响应头字段 **Content-Range**，告诉片段的实际偏移量和资源的总大小，格式是“**bytes x-y/length**”，与 Range 头区别在没有“=”，范围后多了总长度。例如，对于“0-10”的范围请求，值就是“bytes 0-10/100”。

  * 最后剩下的就是发送数据了，直接把片段用 TCP 发给客户端，一个范围请求就算是处理完了。

* 常用的下载工具里的多段下载、断点续传也是基于它实现的，要点是：

  * 先发个 HEAD，看服务器是否支持范围请求，同时获取文件的大小；

  * 开 N 个线程，每个线程使用 Range 字段划分出各自负责下载的片段，发请求传输数据；

  * 下载意外中断也不怕，不必重头再来一遍，只要根据上次的下载记录，用 Range 请求剩下的那一部分就可以了。

##### 多段数据

* 范围请求一次只获取一个片段，其实它还支持在 Range 头里使用多个“x-y”，一次性获取多个片段数据。
* 这种情况需要使用一种特殊的 MIME 类型：“**multipart/byteranges**”，表示报文的 body 是由多段字节序列组成的，并且还要用一个参数“**boundary=xxx**”给出段之间的分隔标记。
* 每一个分段必须以“- -boundary”开始（前面加两个“-”），之后要用“Content-Type”和“Content-Range”标记这段数据的类型和所在范围，然后就像普通的响应头一样以回车换行结束，再加上分段数据，最后用一个“- -boundary- -”（前后各有两个“-”）表示所有的分段结束。

```
   HTTP/1.1 206 Partial Content
   Date: Wed, 15 Nov 1995 06:25:24 GMT
   Last-Modified: Wed, 15 Nov 1995 04:58:08 GMT
   Content-type: multipart/byteranges; boundary=THIS_STRING_SEPARATES

   --THIS_STRING_SEPARATES
   Content-type: application/pdf
   Content-range: bytes 500-999/8000

   ...the first range...
   --THIS_STRING_SEPARATES
   Content-type: application/pdf
   Content-range: bytes 7000-7999/8000

   ...the second range
   --THIS_STRING_SEPARATES--
```

```
    HTTP/1.1 206 Partial Content
    Content-Type: multipart/byteranges; boundary=00000000001
    Content-Length: 189
    Connection: keep-alive
    Accept-Ranges: bytes
    --00000000001
    Content-Type: text/plain
    Content-Range: bytes 0-9/96
    // this is
    --00000000001
    Content-Type: text/plain
    Content-Range: bytes 20-29/96
    ext json d
    --00000000001--
```

