---
title: Nginx-headers-more-nginx-module
mermaid: true
date: 2022-03-27 17:34:11
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
categories:
- Nginx
- module
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
# headers-more-nginx-module

> headers-more-nginx-module 模块用于添加、修改或清除 请求/响应头，该模块不是nginx自带的，默认不包含该模块，需要另外安装。
> 

Github地址：

[GitHub - openresty/headers-more-nginx-module: Set, add, and clear arbitrary output headers in NGINX http servers](https://github.com/openresty/headers-more-nginx-module)

1. 模块主要的指令
- more_set_headers 用于添加、修改、清除响应头
- more_clear_headers 用于清除响应头
- more_set_input_headers 用于添加、修改、清除请求头
- more_clear_input_headers 用于清除请求头

2.修改nginx web 服务器版本信息

```nginx
server {
    listen 80;
    server_name localhost;

    more_set_headers "Server: Web Server";
    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root html;
        index index.html index.htm;
    }
}
```

测试结果

```bash
root@elk:/etc/nginx# curl -L -I localhost
HTTP/1.1 200 OK
Date: Sun, 27 Mar 2022 09:03:37 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 06 Jul 2021 14:59:17 GMT
Connection: keep-alive
ETag: "60e46fc5-264"
Server: Web Server
Accept-Ranges: bytes
```

1. 添加状态码404 响应header

```nginx
server {
    listen 80;
    server_name localhost;

    more_set_headers "Server: Web Server";
    more_set_headers -s 404 "Error: Please Go Home Not found";
    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root html;
        index index.html index.htm;
    }
}
```

测试结果

```bash
root@elk:/etc/nginx# curl -L -I localhost/b
HTTP/1.1 404 Not Found
Date: Sun, 27 Mar 2022 09:09:02 GMT
Content-Type: text/html
Content-Length: 153
Connection: keep-alive
Server: Web Server
Error: Please Go Home Not found
```

4.隐藏Content-Type

```nginx
server {
    listen 80;
    server_name localhost;

    more_set_headers "Server: Web Server";
    more_set_headers -s 404 "Error: Please Go Home Not found";
    more_clear_headers "Content-Type:";
    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root html;
        index index.html index.htm;
    }
}
```

测试结果

```bash
root@elk:/etc/nginx# curl -L -I localhost/b
HTTP/1.1 404 Not Found
Date: Sun, 27 Mar 2022 09:18:14 GMT
Content-Length: 153
Connection: keep-alive
Server: Web Server
Error: Please Go Home Not found
```

5.设置请求头`Host`参数 会rewrite 到指定Host

```nginx
server {
    listen 80;
    server_name localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;
    location / {
        more_set_input_headers 'User-Agent: xxxfakedisoandroid';
        root html;
        index index.html index.htm;
    }
    location /a {
        set $my_host '192.168.253.251';
        more_set_input_headers 'Host: $my_host'
        more_set_input_headers 'User-Agent: xxxfakedisoandroid';
        root html;
        index a.html;
    }
}
```

测试结果

```bash
root@elk:/etc/nginx# curl -L -I localhost/a
HTTP/1.1 301 Moved Permanently
Server: nginx/1.21.1
Date: Sun, 27 Mar 2022 10:08:27 GMT
Content-Type: text/html
Content-Length: 169
Location: http://192.168.253.251/a/
Connection: keep-alive

HTTP/1.1 404 Not Found
Server: nginx/1.17.0
Date: Sun, 27 Mar 2022 10:08:27 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 294466
Connection: keep-alive
Accept-Ranges: none
Vary: Accept-Encoding

root@elk:/etc/nginx# tail -f /var/log/nginx/access.log
127.0.0.1 - - [27/Mar/2022:18:08:27 +0800] "HEAD /a HTTP/1.1" 301 0 "-" "xxxfakedisoandroid"
```

6.清除User-Agent

```nginx
server {
    listen 80;
    server_name localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;
    location / {
       
        root html;
        index index.html index.htm;
    }
    location /a {
        more_clear_input_headers 'User-Agent: ';
        root html;
        index a.html;
    }
}
```

测试结果

```bash
root@elk:/etc/nginx# nginx -s reload
root@elk:/etc/nginx# tail -f /var/log/nginx/access.log
#清除前
192.168.251.101 - - [27/Mar/2022:18:19:03 +0800] "GET /a/ HTTP/1.1" 200 16 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:96.0) Gecko/20100101 Firefox/96.0"
192.168.251.101 - - [27/Mar/2022:18:19:03 +0800] "GET /favicon.ico HTTP/1.1" 404 153 "http://192.168.253.243/a/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:96.0) Gecko/20100101 Firefox/96.0"

#清除后
192.168.251.101 - - [27/Mar/2022:18:19:24 +0800] "GET /a/ HTTP/1.1" 200 16 "-" "-"
192.168.251.101 - - [27/Mar/2022:18:19:24 +0800] "GET /favicon.ico HTTP/1.1" 404 153 "http://192.168.253.243/a/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:96.0) Gecko/20100101 Firefox/96.0"
```