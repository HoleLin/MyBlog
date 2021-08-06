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

### 参考文献

* 深入理解Nginx模块开发与架构解析 陶辉
* 极客时间 Nginx核心知识150讲 陶辉

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

### 使用Nginx的必备软件

* `GCC`编译器

  * `GCC(GNU Complier Collection)`可以用来编译C语言程序

  ```sh
  yum install -y gcc
  yum install -y gcc-c++
  ```

* `PCRE`库

  * `PRCE`(`Perl Compatible Regular Expressions`, Perl 兼容正则表达式)

  ```sh
  yum install -y pcre pcre-devel
  ```

* `zilb`库

  * `zilb`库用于对HTTP包内容做gzip格式的压缩

  ```sh
  yum install -y zilb zlib-devel
  ```

* `OpenSSL`开发库

  ```sh
  yum install -y openssl openssl-devel
  ```

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

### Linux内核参数优化

```tcl
# 表示进程（如一个worker进程）可以同时打开的最大句柄数，这个参数直接限制最大并发连接数，需要根据实际情况配置
fs.file.max = 99999
# 这个参数设置为1，表示允许将TIME-WAIT状态的socket重新用于新的TCP连接
net.ipv4.tcp_tw_resuse = 1
# 这个参数表示当keepalive启用事，TCP发送keepalive消息的频度。默认为2小时。若将其设置得小一点，可以更快的清理无效连接
net.ipv4.tcp_keepalive_time = 600
# 这个参数表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2的状态的最大时间
net.ipv4.tcp_fin_timeout = 30
# 表示操作系统允许TIME_WAIT套接字数量的最大值，若超过这个数字，TIME_WAIT将立即被清除并打印告警信息。该参数默认为18000，过多的TIME_WAIT套接字会使WEB服务器变慢
net.ipv4.tcp_max_tw_buckets = 5000
# 这个参数定义了在UDP和TCP连接中本地（不包括连接的远端）端口的取值范围
net.ipv4.ip_local_port_range = 1024 610000
# 这个参数定义了TCP接收缓存（用于TCP接收滑动窗口）的最小值，默认值，最大值
net.ipv4.tcp_rmem = 4096 32768 262142
# 这个参数定义了TCP发送缓存（用于TCP接收滑动窗口）的最小值，默认值，最大值
net.ipv4.tcp_wmem = 4096 32768 262142

# 当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包，这个参数表示该队列的最大值。
net.core.netdev_max_backlog = 8096
# 这个参数表示内核套接字接收缓存区默认的大小
net.core.rmem_default= = 262144
# 这个参数表示内核套接字发送缓存区默认的大小
net.core.wmem_default= = 262144
# 这个参数表示内核套接字接收缓存区的最大大小
net.core.rmem_max = 2097152
# 这个参数表示内核套接字发送缓存区的最大大小
net.core.wmem_max = 2097152

# 该参数与性能无关 用于解决TCP的SYN攻击
net.ipv4.tcp_syncookies = 1
# 这个参数表示TCP三次握手建立阶段接收SYN请求队列的最大长度，默认为1024，将其设置得大一些可以使出现Nginx繁忙来不及accept新连接的情况时，Linux不至于丢失客户端发起的连接请求。
net.ipv4.tcp_max_syn.backlog = 1024

# 编辑完成后执行sysctl -p使之失效
```

### Nginx编译参数详解

* 路径相关的参数

| 参数名称                            | 含义                                                         | 默认值                                                       |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `--prefix=PATH`                     | Nginx安装部署后的根目录                                      | 默认为`/usr/local/nginx`目录。注意：这个目录的设置会影响其他参数重的相对目录。例如，如果设置了`--sbin-path=sbin/nginx`那么实际上可执行文件会被放到`/usr/local/nginx/sbin/nginx`中 |
| `--sbin-path=PATH`                  | 可执行文件的 放置路径                                        | `<prefix>/sbin/nginx`                                        |
| `--conf-path=PATH`                  | 配置文件的放置路径                                           | `<prefix>/conf/nginx.conf`                                   |
| `--error-log-path=PATH`             | error日志文件的放置路径。error日志用于定位问题，可输出多种级别（包括debug调试级别）的日志。它的配置非常灵活，可以在nginx.conf里面配置为不同请求的日志输出到不同的log文件中，这里是默认的Nginx核心日志路径 | `<prefix>/logs/error.log`                                    |
| `--pid-path=PATH`                   | pid存放路径。这个文件里仅以ASCII存放着Nginx master的进程ID，有了这个进程ID，在使用命令行（如 nginx -s reload）通过读取master进程ID向master进程发送信号时，才能对运行中的Nginx服务产生作用 | `<prefix>/logs/nginx.pid`                                    |
| `--lock-path=PATH`                  | lock文件的放置路径                                           | `<prefix>/logs/nginx.lock`                                   |
| `--builddir=DIR`                    | configure执行时与编译期间产生的临时文件放置的目录，包括产生的Makefile，C源文件，目标文件，可执行文件 | `<nginx source path>/objs`                                   |
| `--with-perl_modules_path=PATH`     | perl module放置的路径，只有使用了第三方的perl module，才需要配置这个路径 | 无                                                           |
| `--with-perl=PATH`                  | perl binary 放置的路径，如果配置的Nginx会执行Perl脚本，那么就必须要设置此路径 | 无                                                           |
| `--http-log-path=PATH`              | access日志放置的位置。每一个HTTP请求在结束时都会记录的访问日志 | `<prefix>/logs/access.log`                                   |
| `--http-client-body-temp-path=PATH` | 处理HTTP请求时如果请求的包体需要暂时存放到临时磁盘文件中，则把这样的临时文件放置到该路径下 | `<prefix>/client_body_temp`                                  |
| `--http-proxy-temp-path=PATH`       | Nginx作为HTTP反向代理服务器时，上游服务器产生的HTTP包体在需要临时存放到磁盘文件时，这样的临时文件将放到该路径下 | `<prefix>/proxy_temp`                                        |
| `--http-fastcgi-temp-path=PATH`     | Fastcgi所使用临时文件放置目录                                | `<prefix>/fastcgi_temp`                                      |
| `--http-uwsgi-temp-path=PATH`       | uWSGI所使用临时文件的放置目录                                | `<prefix>/uwsgi_temp`                                        |
| `--http-scgi-temp-path=PATH`        | SCGI所使用临时文件的放置目录                                 | `<prefix>/scgi_temp`                                         |

* 编译相关的参数

| 编译参数                | 含义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `--with-cc=PATH`        | C编译器的路径                                                |
| `--with-cpp=PATH`       | C预编译的路径                                                |
| `--with-cc-opt=OPTIONS` | 如果希望在Nginx编译期间指定加入一些编译选项，如指定宏或者使用-I加入某些需要包含的目录，这时可以使用该参数达成目的 |
| `--with-ID-opt=OPTIONS` | 最终的二进制可执行文件是由编译后生成的目标文件与一些第三方库链接生成的，在执行链接操作时可能会需要指定链接参数，--with-Id-opt就是用于加入链接时的参数。如要将某个库链接到Nginx程序中，需要在这里加入--with-Id-opt=librayName -LlibaryPath,其中libaryName是目标库的名称，libraryPath则是目标库所在的路径 |
| `--with-cpu-opt=CPU`    | 指定CPU处理架构，只能从以下取值：`pentium`,`pentiumpro`,`pentium3`,`pentium4`,`athlon`,`opteron`,`sparc32`,`sparc64`,`ppc64` |

* 依赖软件的相关参数

| PCRE库的设置参数          | 意义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `--without-pcre`          | 如果确认Nginx不用解析正则表达式，也就是说，nginx.conf配置文件中不会出现正则表达式，那么可以使用这个参数 |
| `--with-pcre`             | 强制使用PCRE库                                               |
| `--with-pcre=DIR`         | 指定PCRE库源码位置，在编译Nginx时会进入该目录编译PCRE源码    |
| `--with-pcre-opt=OPTIONS` | 编译PCRE源码时希望加入的编译选项                             |

| OpenSSL库的设置参数      | 意义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `--with-openssl=DIR`     | 指定OpenSSL库的源码位置，在编译Nginx时会进入该目录编译OpenSSL源码 |
| `--with-openssl=OPTIONS` | 编译OpenSSL源码时希望加入的编译选项                          |

* 其他参数

* | 其他参数               | 含义                                              |
  | ---------------------- | ------------------------------------------------- |
  | `--with-debug`         | 将Nginx需要打印debug调试级别日志的代码编译进Nginx |
  | `--add-module=PATH`    |                                                   |
  | `--without-http`       |                                                   |
  | `--without-http-cache` |                                                   |
  | `--with-file-aio`      |                                                   |
  | `--with-ipv6`          |                                                   |
  | `--user=USER`          |                                                   |
  | `--group=GROUP`        |                                                   |


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
  * alias在使用正则匹配时，必须捕捉匹配的内容并指定的内容处使用；
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
  * `nginx -s reload`: `kill -s SIGHUP <master pid>`
  * `nginx -s reopen`: `kill -s SIGUSR1 <master pid>`
  * `nginx -s stop`: `kill -s SIGTERM <master pid>`
  * `nginx -s quit`: `kill -s SIGQUIT <master pid>`

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

### Nginx日志级别

* 从上至下级别依次增大，当设定为一个级别时，大于或等于该级别的日志都会被输出到日志文件中，小于该级别的日志则不会输出。
  * `debug`
  * `Info`
  * `notice`
  * `warn`
  * `error`
  * `crit`
  * `alert`
  * `emerg`
* 如果日志级别设定到`debug`，必须在`configure`时加入`--with-debug`配置项
  
  
