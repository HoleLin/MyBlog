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

## 目录

* Nginx的优点
* Nginx的三个应用场景
* Ngnix的组成
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

### Ngnix的组成

* Nginx二进制可执行文件
  * 由各个模块源码编辑出的一个文件
* Ngnix.conf配置文件
  * 控制Nginx的行为
* access.log访问日志
  * 记录每一条HTTP请求信息
* error.log错误日志
  * 定位问题

### Nginx版本

* 开源免费的Nginx http://www.nginx.org
* 商业版的Nginx Plus http://www.nginx.com
* 阿巴巴巴的Tenginehttp://tengine.taobao.org/
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

* 格式: n`ginx -s reload`
  * 帮助:` -? /-h`
  * 使用指定的配置文件: ` -c`
  * 指定配置指令: `-g`
  * 指定运行目录: `-p`
  * 发送信号: `-s`
  *  测试配置文件是否有语法: `-t /-T`
  * 打印nginx的版本信息,编译信息等: `-v -V`

 * 立即停止服务: `stop`
 * 优雅的停止服务: `quit`
 * 重载配置文件: `reload`
 * 重新开始记录日志文件: `reopen`

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

