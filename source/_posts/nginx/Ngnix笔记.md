---
title: Nginx-学习笔记(一)
mermaid: true
date: 2021-07-31 17:34:11
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
- 基础 
categories:
- Nginx
---

## 目录

* Nginx的优点
* Nginx的三个应用场景
* Nginx的组成
* Nginx版本
* 编译安装Nginx

### Nginx的优点

* 高并发,高性能
* 可扩展性好
* 高可用
* 热部署
* BSD许可证

### Nginx的三个应用场景

* 静态资源服务
  * 通过本地文件系统提供服务
* 反向代理服务
  * Nginx的强大性能
  * 缓存
  * 负载均衡
* API服务
  * OpenResty

### Nginx的组成

* Nginx二进制可执行文件
  * 由各个模块源码编辑出的一个文件
* Nginx.conf配置文件
  * 控制Nginx的行为
* access.log访问日志
  * 记录每一条HTTP请求信息
* error.log错误日志
  * 定位问题

### Nginx版本

* 开源免费的Nginx http://www.nginx.org
* 商业版的Nginx Plus http://www.nginx.com
* 阿巴巴巴的Tengine http://tengine.taobao.org/
* 免费OpenResty http://openresty.org
* 商业版OpenResty http://openresty.com

### 编译安装Nginx

* 下载Ngnix

* Nginx各个目录

  ![img](http://www.chenjunlin.vip/img/nginx/nginx%E7%9B%AE%E5%BD%95.png)

  * 执行`cp -r contrib/vim/* ~/.vim/`,使得Vim识别nginx.conf的配置语法,使其高亮
  * 执行`./configure --help |more`,查看configure支持的配置

* Configure

* 中间文件(objs)

  ![img](http://www.chenjunlin.vip/img/nginx/nginx%E4%B8%AD%E9%97%B4%E6%96%87%E4%BB%B6(objs).png)

* 编译

  * `./configure --prefix=/home/holelin/nginx`
  * `make`

* 安装

  * `make install`

### Nginx编译参数详解

> **–prefix= 指向安装目录**
> –sbin-path 指向（执行）程序文件（nginx）
> **–conf-path= 指向配置文件（nginx.conf）**
> **–error-log-path= 指向错误日志目录**
> **–pid-path= 指向 pid文件（nginx.pid）**
> –lock-path= 指向lock文件（nginx.lock）（安装文件锁定，防止安装文件被别人利用，或自己误操作。）
> **–user= 指定程序运行时的非特权用户**
> –group= 指定程序运行时的非特权用户组
> –builddir= 指向编译目录
> –with-rtsig_module 启用 rtsig模块支持（实时信号）
> –with-select_module 启用 select模块支持（一种轮询模式,不推荐在高载环境下使用）禁用：–without- select_module
> –with-poll_module 启用 poll模块支持（功能与 select相同，与select特性相同，为一种轮询模式,不推荐在 高载环境下使用）
> –with-file-aio 启用 file aio支持（一种 APL文件传输格式） –with-ipv6 启用 ipv6支持
> **–with-http_ssl_module 启用 ngx_http_ssl_module 支持（使支持https请求，需已安装openssl）**
> –with-http_realip_module 启用 ngx_http_realip_module支持（这个模块允许从请求标头更改客户端的 IP 地 址值，默认为关）
>
> –with-http_addition_module 启用ngx_http_addition_module支持（作为一个输出过滤器，支持不完全缓冲， 分部分响应请求）
> –with-http_xslt_module 启用ngx_http_xslt_module 支持（过滤转换XML请求）
> –with-http_image_filter_module 启用ngx_http_image_filter_module支持（传输JPEG/GIF/PNG 图片的一个 过滤器）（默认为不启用。gd库要用到）
> –with-http_geoip_module 启用 ngx_http_geoip_module支持（该模块创建基于与 MaxMind GeoIP 二进制文件相 配的客户端 IP地址的ngx_http_geoip_module 变量）
> –with-http_sub_module 启用 ngx_http_sub_module 支持（允许用一些其他文本替换nginx响应中的一些文本） –with-http_dav_module 启用 ngx_http_dav_module 支持（增加 PUT,DELETE,MKCOL：创建集合,COPY和MOVE方 法）默认情况下为关闭，需编译开启
> –with-http_flv_module 启用 ngx_http_flv_module 支持（提供寻求内存使用基于时间的偏移量文件） –with-http_gzip_static_module 启用 ngx_http_gzip_static_module支持（在线实时压缩输出数据流） –with-http_random_index_module 启用ngx_http_random_index_module支持（从目录中随机挑选一个目录索 引）
> –with-http_secure_link_module 启用 ngx_http_secure_link_module支持（计算和检查要求所需的安全链接网 址）
> –with-http_degradation_module  启用 ngx_http_degradation_module支持（允许在内存不足的情况下返回 204 或444码）
> –with-http_stub_status_module 启用 ngx_http_stub_status_module支持（获取nginx自上次启动以来的工作 状态）
> –without-http_charset_module 禁用 ngx_http_charset_module支持（重新编码web页面，但只能是一个方向 –服务器端到客户端，并且只有一个字节的编码可以被重新编码）
> –without-http_gzip_module 禁用 ngx_http_gzip_module支持（该模块同-with-http_gzip_static_module功能 一样）
> –without-http_ssi_module 禁用 ngx_http_ssi_module支持（该模块提供了一个在输入端处理处理服务器包含 文件（SSI）的过滤器，目前支持SSI命令的列表是不完整的）
> –without-http_userid_module 禁用 ngx_http_userid_module支持（该模块用来处理用来确定客户端后续请求 的 cookies）
> –without-http_access_module 禁用 ngx_http_access_module支持（该模块提供了一个简单的基于主机的访问 控制。允许/拒绝基于ip地址）
> –without-http_auth_basic_module禁用ngx_http_auth_basic_module（该模块是可以使用用户名和密码基于 http基本认证方法来保护你的站点或其部分内容）
> –without-http_autoindex_module 禁用disable ngx_http_autoindex_module支持（该模块用于自动生成目录 列表，只在 ngx_http_index_module模块未找到索引文件时发出请求。）
> –without-http_geo_module 禁用 ngx_http_geo_module支持（创建一些变量，其值依赖于客户端的 IP地址） –without-http_map_module 禁用 ngx_http_map_module支持（使用任意的键/值对设置配置变量）
> –without-http_split_clients_module 禁用ngx_http_split_clients_module支持（该模块用来基于某些条件 划分用户。条件如：ip地址、报头、cookies等等）
> –without-http_referer_module 禁用 disable ngx_http_referer_module支持（该模块用来过滤请求，拒绝报 头中 Referer 值不正确的请求）
> –without-http_rewrite_module 禁用 ngx_http_rewrite_module支持（该模块允许使用正则表达式改变URI，并 且根据变量来转向以及选择配置。如果在 server级别设置该选项，那么他们将在 location之前生效。如果在 location还有更进一步的重写规则，location部分的规则依然会被执行。如果这个URI重写是因为 location部 分的规则造成的，那么 location部分会再次被执行作为新的 URI。 这个循环会执行10次，然后Nginx会返回一 个 500错误。）
> –without-http_proxy_module 禁用ngx_http_proxy_module支持（有关代理服务器）
> –without-http_fastcgi_module 禁用 ngx_http_fastcgi_module支持（该模块允许Nginx 与 FastCGI 进程交 互，并通过传递参数来控制 FastCGI 进程工作。 ）FastCGI一个常驻型的公共网关接口。
>
> –without-http_uwsgi_module 禁用ngx_http_uwsgi_module支持（该模块用来医用 uwsgi协议，uWSGI服务器相 关）
> –without-http_scgi_module 禁用 ngx_http_scgi_module支持（该模块用来启用 SCGI协议支持，SCGI协议是 CGI 协议的替代。它是一种应用程序与 HTTP服务接口标准。它有些像 FastCGI但他的设计 更容易实现。）
> –without-http_memcached_module 禁用ngx_http_memcached_module支持（该模块用来提供简单的缓存，以提 高系统效率）
> -without-http_limit_zone_module 禁用ngx_http_limit_zone_module支持（该模块可以针对条件，进行会话的 并发连接数控制）
> –without-http_limit_req_module 禁用ngx_http_limit_req_module支持（该模块允许你对于一个地址进行请 求数量的限制用一个给定的 session或一个特定的事件）
> –without-http_empty_gif_module 禁用ngx_http_empty_gif_module支持（该模块在内存中常驻了一个 1*1的 透明 GIF 图像，可以被非常快速的调用）
> –without-http_browser_module 禁用 ngx_http_browser_module支持（该模块用来创建依赖于请求报头的值。 如果浏览器为 modern ，则$modern_browser等于modern_browser_value 指令分配的值；如 果浏览器为old，则 $ancient_browser等于 ancient_browser_value指令分配的值；如果浏览器为 MSIE中的任意版本，则 $msie等 于 1）
> –without-http_upstream_ip_hash_module 禁用 ngx_http_upstream_ip_hash_module支持（该模块用于简单的 负载均衡）
> –with-http_perl_module 启用ngx_http_perl_module 支持（该模块使nginx可以直接使用perl或通过 ssi调 用 perl）
> –with-perl_modules_path= 设定 perl模块路径
> –with-perl= 设定perl库文件路径
> –http-log-path= 设定access log路径
> –http-client-body-temp-path= 设定 http客户端请求临时文件路径
> –http-proxy-temp-path= 设定http代理临时文件路径
> –http-fastcgi-temp-path= 设定 http fastcgi临时文件路径
> –http-uwsgi-temp-path= 设定http uwsgi 临时文件路径
> –http-scgi-temp-path= 设定 http scgi临时文件路径 -without-http 禁用 http server功能
> –without-http-cache 禁用 http cache功能
> –with-mail 启用 POP3/IMAP4/SMTP代理模块支持 –with-mail_ssl_module 启用 ngx_mail_ssl_module 支持
> –without-mail_pop3_module 禁用 pop3协议（POP3即邮局协议的第 3个版本,它是规定个人计算机如何连接到互 联网上的邮件服务器进行收发邮件的协议。是因特网电子邮件的第一个离线协议标 准,POP3协议允许用户从服务 器上把邮件存储到本地主机上,同时根据客户端的操作删除或保存在邮件服务器上的邮件。POP3协议是TCP/IP协 议族中的一员，主要用于 支持使用客户端远程管理在服务器上的电子邮件）
> –without-mail_imap_module 禁用 imap协议（一种邮件获取协议。它的主要作用是邮件客户端可以通过这种协 议从邮件服务器上获取邮件的信息，下载邮件等。IMAP协议运行在TCP/IP协议之上， 使用的端口是143。它与 POP3协议的主要区别是用户可以不用把所有的邮件全部下载，可以通过客户端直接对服务器上的邮件进行操 作。）
> –without-mail_smtp_module 禁用 smtp协议（SMTP即简单邮件传输协议,它是一组用于由源地址到目的地址传送 邮件的规则，由它来控制信件的中转方式。SMTP协议属于TCP/IP协议族，它帮助每台计算机在发送或中转信件时 找到下一个目的地。）
> –with-google_perftools_module 启用 ngx_google_perftools_module支持（调试用，剖析程序性能瓶颈） –with-cpp_test_module 启用 ngx_cpp_test_module 支持
> –add-module= 启用外部模块支持 –with-cc= 指向 C编译器路径 –with-cpp= 指向 C预处理路径
>
> –with-cc-opt= 设置 C编译器参数（PCRE库，需要指定–with-cc-opt=”-I /usr/local/include”，如果使用 select()函数则需要同时增加文件描述符数量，可以通过–with-cc- opt=”-D FD_SETSIZE=2048”指定。） –with-ld-opt= 设置连接文件参数。（PCRE库，需要指定–with-ld-opt=”-L /usr/local/lib”。） –with-cpu-opt= 指定编译的CPU，可用的值为: pentium, pentiumpro, pentium3, pentium4, athlon, opteron, amd64, sparc32, sparc64, ppc64
> –without-pcre 禁用 pcre库
> –with-pcre 启用 pcre库
> –with-pcre= 指向pcre库文件目录
> –with-pcre-opt= 在编译时为pcre库设置附加参数
> –with-md5= 指向 md5库文件目录（消息摘要算法第五版，用以提供消息的完整性保护）
> –with-md5-opt= 在编译时为md5库设置附加参数
> –with-md5-asm 使用 md5汇编源
> –with-sha1= 指向sha1库目录（数字签名算法，主要用于数字签名）
> –with-sha1-opt= 在编译时为sha1库设置附加参数
> –with-sha1-asm 使用 sha1汇编源
> –with-zlib= 指向zlib库目录
> –with-zlib-opt= 在编译时为zlib设置附加参数
> –with-zlib-asm= 为指定的CPU使用 zlib汇编源进行优化，CPU 类型为pentium, pentiumpro
> –with-libatomic 为原子内存的更新操作的实现提供一个架构
> –with-libatomic= 指向 libatomic_ops安装目录
> –with-openssl= 指向 openssl安装目录
> –with-openssl-opt 在编译时为openssl设置附加参数
> –with-debug 启用debug日志

### Nginx配置语法

* 配置文件由指令与指令块构成;
* 每条指令以`;`分号结尾,指令与参数间以空格符号分隔;
* 指令块以{}大括号将多条指令组织在一起;
* include语法允许组合多个配置文件以提升可维护性;
* 使用#符号添加注释,提高可读性;
* 使用$符号使用变量;
* 部分指令的参数支持正则表达式;

* 配置参数: 时间单位

  * ms: milliseconds
  * s: seconds
  * m: minutes
  * h: hours
  * d: days
  * w: weeks
  * M: months,30days
  * y: years,365days

* 配置参数: 空间单位

  * bytes
  * k/K: kilobytes
  * m/M: megabytes
  * g/G: gigabytes

### Nginx命令行

* 格式: `nginx -s reload`
  * 帮助:` -? /-h`
  * 使用指定的配置文件: ` -c`
  * 指定配置指令: `-g`
  * 指定运行目录: `-p`
  * 发送信号: `-s`
  *  测试配置文件是否有语法: `-t /-T`
  * 打印nginx的版本信息,编译信息等: `-v -V`
 * 立即停止服务: `nginx -s stop`
 * 优雅的停止服务: `nginx -s quit`
 * 重载配置文件: `nginx -s reload`
 * 重新开始记录日志文件: `nginx -s reopen`

### Nginx location配置

* 语法格式:`location [=|~|~*|^~] /url/ {...}`

  * `=`: 表示精确匹配，这个优先级最高；
  * `^~`:表示uri以某个常规字符串开头，理解为匹配url路径即可。Nginx不对url做编码。因此请求为`/static/20%/aa`可以被规则`^~/static/ /aa`匹配到；
  * `~`表示区分大小写的正则匹配；
  * `~*`表示不区分大小写的正则匹配；
  * `!~`表示区分大小写不匹配的正则；
  * `!~*`表示不区分大小写不匹配的正则；
  * `/`通用匹配，任何请求都会匹配到，默认匹配；

* 匹配优先级

  * 第一优先级：等号类型（`=`）的优先级最高。一旦匹配成功，则不再查找其他匹配项。
  * 第二优先级：`^~`类型表达式。一旦匹配成功，则不再查找其他匹配项。
  * 第三优先级：正则表达式类型（`~` `~*`）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。
  * 第四优先级：常规字符串匹配类型。按前缀匹配。

  ```
  (location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)
  ```

### Nginx root和alias配置

* `root`

  ```
  语法：root path 默认值：root html
  配置段：http、server、location、if
  ```

* `alias`

  ```
  语法：alias path 
  配置段：location
  ```

  * 使用alias时，目录名后面一定要加上**“/”**；
  * alias可以指定任何名称。
  * alias在使用正则匹配时，必须捕捉品牌的内容并指定的内容处使用；
  * alias只能位于location块中；

### Nginx `ngx_http_core_module`变量说明

| 参数名称               | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `$arg_PARAMETER`       | HTTP 请求中某个参数的值，如/index.php?site=`www.ttlsa.com`，可以用$arg_site取 得 `www.ttlsa.com` 这个值. |
| `$args HTTP`           | 请求中的完整参数。例如，在请求/index.php?width=400&height=200 中，$args表示 字符串 width=400&height=200. |
| `$binary_remote_addr ` | 二进制格式的客户端地址。例如：\x0A\xE0B\x0E                  |
| `$body_bytes_sent`     | 表示在向客户端发送的http响应中，包体部分的字节数             |
| `$content_length`      | 表示客户端请求头部中的 Content-Length 字段                   |
| `$content_type `       | 表示客户端请求头部中的 Content-Type 字段                     |
| `$cookie_COOKIE`       | 表示在客户端请求头部中的 cookie 字段                         |
| `$document_root`       | 表示当前请求所使用的root 配置项的值                          |
| `$uri`                 | 表示当前请求的 URI，不带任何参数                             |
| `$document_uri`        | 与$uri 含义相同                                              |
| `$request_uri `        | 表示客户端发来的原始请求 URI，带完整的参数。$uri和$document_uri 未必是用户的 原始请求，在内部重定向后可能是重定向后的 URI，而$request_uri 永远不会改变，始终是客户端的原始 URI. |
| `$host`                | 表示客户端请求头部中的 Host字段。如果Host字段不存在，则以实际处理的 server（虚拟主机）名称代替。如果 Host 字段中带有端口，如 IP:PORT，那么$host是去掉端口的，它的值为 IP。$host 是全小写的。这些特性与http_HEADER 中的 http_host不同，http_host只取出Host头部对应的值. |
| `$hostname `           | 表示 Nginx 所在机器的名称，与 gethostbyname调用返回的值相同  |
| `$http_HEADER`         | 表示当前 HTTP请求中相应头部的值。HEADER名称全小写。例如，示请求中 Host头部 对应的值  用 $http_host表 |
| `$sent_http_HEADER`    | 表示返回客户端的 HTTP响应中相应头部的值。HEADER名称全小写。例如，用 $sent_ http_content_type表示响应中 Content-Type头部对应的值 |
| `$is_args`             | 表示请求中的 URI是否带参数，如果带参数，$is_args值为 ?，如果不带参数，则是空字符串 |
| `$limit_rate`          | 表示当前连接的限速是多少，0表示无限速                        |
| `$nginx_version`       | 表示当前 Nginx的版本号                                       |
| `$query_string`        | 请求 URI中的参数，与 $args相同，然而 $query_string是只读的不会改变 |
| `$remote_addr`         | 表示客户端的地址                                             |
| `$remote_port`         | 表示客户端连接使用的端口                                     |
| `$remote_user `        | 表示使用 Auth Basic Module时定义的用户名                     |
| `$request_filename`    | 表示用户请求中的 URI经过 root或 alias转换后的文件路径        |
| `$request_body `       | 表示 HTTP 请求中的包体，该参数只在 proxy_pass或 fastcgi_pass中有意义 |
| `$request_body_file `  | 表示 HTTP 请求中的包体存储的临时文件名                       |
| `$request_completion`  | 当请求已经全部完成时，其值为 “ok”。若没有完成，就要返回客户端，则其值为空字符串；或者在断点续传等情况下使用 HTTP range访问的并不是文件的最后一块，那么其值也是空字符串。 |
| `$request_method `     | 表示 HTTP 请求的方法名，如 GET、PUT、POST等                  |
| `$scheme`              | 表示 HTTP scheme（协议），如在请求 https://nginx.com/中表示 https |
| `$server_addr`         | 表示服务器地址                                               |
| `$server_name`         | 表示服务器名称                                               |
| `$server_port`         | 表示服务器端口                                               |
| `$server_protocol`     | 表示服务器向客户端发送响应的协议，如 HTTP/1.1或 HTTP/1.0     |

### Nginx 日志配置

* `access_log`指令

  ```
  access_log    <FILE>    <NAME>;
      关键字         日志文件   格式标签
      关键字：其中关键字error_log不能改变
      日志文件：可以指定任意存放日志的目录
      格式标签：给日志文件套用指定的日志格式
  其他语法：
      access_log    off;  #关闭access_log，即不记录访问日志
      access_log path [format [buffer=size [flush=time]] [if=condition]];
      access_log path format gzip[=level] [buffer=size] [flush=time] [if=condition];
      access_log syslog:server=address[,parameter=value] [format [if=condition]];
  
  说明：
      buffer=size  #为存放访问日志的缓冲区大小
      flush=time  #为缓冲区的日志刷到磁盘的时间
      gzip[=level]  #表示压缩级别
      [if = condition]  #表示其他条件
      
  配置段: http, server, location, if in location, limit_except 
  ```

### 隐藏Nginx版本号的安全性与方法

* 编辑`nginx.conf`在http{...}中添加`server_tokens off;`

* 编辑`fastcgi.conf`或`fcgi.conf`

  ```
  找到：
  fastcgi_param SERVER_SOFTWARE nginx/$nginx_version; 
  改为：
  fastcgi_param SERVER_SOFTWARE nginx;
  ```

#### 热部署升级

* 复制一份最新的`nginx(/sbin/nginx)`;

  ```sh
  cp nginx nginx.old
  ```

* `kill -USR2  原PID` 平滑的启动新的进程(`master`和`worker`);

* `kill -WINCH 原PID` 优雅的关闭老的`worker`进程;

* 最终老的`master`进程会被保留,以备回滚版本时使用;

#### 日志切割

* 复制一份原有的日志文件 `mv xxx.log back.log`
* `../sbin/nginx -s reopen`
* 使用编写脚本,通过定时任务来进行日志切割

#### 配置静态服务器

* `gzip`

  ```tcl
  gzip配置的常用参数
  
  gzip on|off; #是否开启gzip
  
  gzip_buffers 32 4K| 16 8K #缓冲(压缩在内存中缓冲几块? 每块多大?)
  
  gzip_comp_level [1-9] #推荐6 压缩级别(级别越高,压的越小,越浪费CPU计算资源)
  
  gzip_disable #正则匹配UA 什么样的Uri不进行gzip
  
  gzip_min_length 200 # 开始压缩的最小长度(再小就不要压缩了,意义不在)
  
  gzip_http_version 1.0|1.1 # 开始压缩的http协议版本(可以不设置,目前几乎全是1.1协议)
  
  gzip_proxied # 设置请求者代理服务器,该如何缓存内容
  
  gzip_types text/plain application/xml # 对哪些类型的文件用压缩 如txt,xml,html ,css
  
  gzip_vary on|off # 是否传输gzip压缩标志
  ```

* `access_log`

  ```tcl
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
    
       access_log  logs/access.log  main;
  ```

* 配置上游服务

  ```tcl
  upstream local {
  	server 127.0.0.1:8080;
  }
  server {
  	server_name downstream_server;
  	listen 80;
  	
  	location / {
  		proxy_set_header Host $host;
  		proxy_set_header X-Real-IP $remote_addr;
  		proxy_set_header X-Forwarded-For $proxy_add_x_forward_for;
  		proxy_pass http://local;
  	}
  }
  ```

#### 日志查看

* `goaccess`安装

  ```
  $ wget http://tar.goaccess.io/goaccess-1.2.tar.gz
  $ tar -xzvf goaccess-1.2.tar.gz
  $ cd goaccess-1.2/
  $ ./configure --enable-utf8 --enable-geoip=legacy
  $ make
  # make install
  ```

* 出错一

  ```sh
  configure: error: 
      *** Missing development files for the GeoIP library
  ```

  * 解决方法：安装`GeoIP`

    ```
    $ wget https://github.com/maxmind/geoip-api-c/releases/download/v1.6.11/GeoIP-1.6.11.tar.gz
    $ tar -xzvf GeoIP-1.6.11.tar.gz
    $ cd GeoIP-1.6.11
    $ ./configure
    $ make
    # make install
    ```

* 出错二

  ```
  ./bin2c: error while loading shared libraries: libGeoIP.so.1: cannot open shared object file: No such file or directory
  ```

  * 解决方法：`ln -s /usr/local/lib/libGeoIP.so* /lib64/`

* 出错三

  ```sh
  [root@holelin logs]# goaccess access.log  -o ../html/report.html --real-time-html --time-format='%H:%M:%S' --date-format='%d/%b/%Y' --log-format=COMBINED
  Error Opening file /usr/local/share/GeoIP/GeoIP.dat
  WebSocket server ready to accept new client connections
  ```

  * 原因： 缺少`GeoIP.dat`文件 GeoIP数据库文件
  * 解决方法： 
    * 补充[`GeoIP.dat`文件](http://www.chenjunlin.vip/file/GeoIP.dat.gz)
    * [下载网址](https://www.miyuru.lk/geoiplegacy)

#### `SSL`协议

* `SSL(Secure Sockets Layer)`

* `TLS(Transport Layer Security)`

* `SSL3.0`-->`TLS1.0`-->`TLS1.1`-->`TLS1.2`-->`TLS1.3`

* `ISO/OSI`模型:**表示层**

* `TCP/IP`模型:**应用层**

  * 握手
  * 交换密钥
  * 告警
  * 对称加密的应用数据
  
##### `TLS`安全密码套件

* `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` 
  * `ECDHE`: 密钥交换
  * `RSA`: 身份验证
  * `AES`: 算法
  * `128`: 强度
  * `GCM`: 模式
  * `SHA256` : `MAC`或`PRF`
* 密钥交换算法
* 身份验证算法
* 对称加密算法、强度、分组模式
* 签名hash算法

##### 证书类型

* 域名验证（domain validated ,DV）证书
* 组织验证（organization validated,OV）证书
* 扩展验证（extended validation,EV）证书

##### 给站点附加SSL证书实现HTTPS

* `yum install python2-cerbot-nginx`

* 修改`nginx.conf`文件的`server_name`为`域名`

* `certbot --nginx --nginx-server-root=/usr/local/tengine/conf/ -d 域名`

  ```sh
  [root@holelin ~]# certbot --nginx --nginx-server-root=/usr/local/tengine/conf/ -d chenjunlin.vip
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  The nginx plugin is not working; there may be problems with your existing configuration.
  The error was: PluginError('Nginx build is missing SSL module (--with-http_ssl_module).',)
  [root@holelin ~]# nginx -V
  Tengine version: Tengine/2.3.2
  nginx version: nginx/1.17.3
  built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
  built with OpenSSL 1.0.2k-fips  26 Jan 2017
  TLS SNI support enabled
  configure arguments: --prefix=/usr/local/tengine
  ```

  * 缺少`--with-http_ssl_module`
    * 参考文献：https://www.cnblogs.com/NGames/p/12078503.html
    * 重新编译nginx服务器
    * `./configure --prefix=/usr/local/tengine/  --with-http_ssl_module`
    * `make`
    * `make install` 此处因为我之前只是简单安装尚未指定任何模块，故而直接重新安装

### Nginx进程结构

* 一个Master进程
* 多个Worker进程
* 一个Cache Manager进程
* 一个Cache Loader进程

### Nginx信号量

* Master进程
  * 监控Worker进程
    * `CHLD`
  * 管理Worker进程
  * 接受信号
    * `TERM,INT`
    * `QUIT`
    * `HUP`
    * `USR1`
    * `USR2`
    * `WINCH`
* Worker进程
  * 接受信号
    * `TERM,INT`
    * `QUIT`
    * `USR1`
    * `WINCH`
* Nginx命令行
  * `reload`: `HUP`
  * `reopen`: `USR1`
  * `stop`: `TERM`
  * `quit`: `QUIT`

### Nginx `reload` 流程

* 向`master`进程发送`HUP`(`reload`命令)
* `master`进程校验配置语法是否正确
* `master`进程打开新的监听端口
* `master`进程用新的配置启动新的`worker`子进程
* `master`进程向老`worker`子进程发送`QUIT`信号
* 老`worker`进程关闭监听句柄，处理完当前连接后结束进程

### Nginx热升级流程

* 将旧Nginx文件换成新Nginx文件（注意备份）
* 向`master`进程发送`USR2`信号
* `master`进程修改`pid`文件名，加上`.oldbin`
* `master`进程用新Nginx文件启动新`master`进程
* 向老`master`进程发送`QUIT`信号，关闭老`master`进程
* 回滚：向老`master`发送`HUP`，向新`master`发送`QUIT`

### Worker进程：优雅的关闭适合HTTP请求

* 设置定时器:`worker_shutdown_timeout`
* 关闭监听句柄
* 关闭空闲连接
* 在循环中等待全部连接关闭
* 退出进程
