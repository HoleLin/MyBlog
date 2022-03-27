---
title: Nginx-add header
mermaid: true
date: 2022-03-26 17:34:11
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
categories:
- Nginx
- header
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
# add_header

> [Adds the specified field to a response header provided that the response code equals 200, 201 (1.3.10), 204, 206, 301, 302, 303, 304,307 (1.1.16, 1.0.13), or 308 (1.13.0).Parameter value can contain variables.](https://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)
> 

> [There could be several `add_header` directives.These directives are inherited from the previous configuration level if and only if there are no `add_header` directivesdefined on the current level.](https://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)
> 

官方文档的意思也就是在响应状态码**成功**时，`add_header`指令才生效，并且当前“作用域”下没有 `add_header`指令时，会向上层继承。

- 作用：使用add_header指令来设置response header
- 官方示例

```bash
Syntax: 	add_header name value [always];
Default: 	—
Context: 	http, server, location, if in location

#在不添加always的情况下，add_header只对响应码为200, 201 (1.3.10), 204, 206, 301, 302, 303, 304, 307 (1.1.16, 1.0.13), 308 (1.13.0)这些生效（括号内是nginx版本）。也就是说当服务端返回响应异常，响应码不是上述之一的话，即使nginx有配跨域头信息，浏览器仍然会显示跨域错误。原因就是因为nginx对异常响应码添加add_header无效
```

- 示例
1. 给根目录下添加header ，名字为name 值为tom

```nginx
location / {
    add_header name tom;
    root html;
    index index.html index.htm;
}
```

访问响应头部信息

```bash
root@elk:/etc/nginx# curl -I -L localhost
HTTP/1.1 200 OK
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:20:27 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 06 Jul 2021 14:59:17 GMT
Connection: keep-alive
ETag: "60e46fc5-264"
name: tom
Accept-Ranges: bytes
```

1. 如果状态码不是200 204等测试是否生效

```nginx
location / {
    add_header name tom;
    root html;
    index index.html index.htm;
    return 510;
}
```

测试结果 没有添加等头部信息

```bash
root@elk:/etc/nginx# curl -I -L localhost
HTTP/1.1 510
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:26:17 GMT
Content-Length: 0
Connection: keep-alive
```

3、添加always

```nginx
location / {
    add_header name tom always;
    root html;
    index index.html index.htm;
    return 510;
}
```

结果

```bash
root@elk:/etc/nginx# curl -I -L localhost
HTTP/1.1 510
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:25:20 GMT
Content-Length: 0
Connection: keep-alive
name: tom
```

3.每一层都可以从上层继承 add_header，但是如果当前层添加了add_header，则不能继承

1. http 模块
2. server 模块
3. location 模块
4. location中的 if 模块

```nginx
server {
    listen 80;
    server_name localhost;

    #charset koi8-r;
    add_header name tom;

    #access_log  logs/host.access.log  main;

    location / {
        root html;
        index index.html index.htm;
    }
    location /a {
        add_header name tomcat;
        root html;
        index a.html;
    }
}    
```

访问结果: 继承server 的add header

```bash
root@elk:/var/log/nginx# curl -L -I localhost
HTTP/1.1 200 OK
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:34:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 06 Jul 2021 14:59:17 GMT
Connection: keep-alive
ETag: "60e46fc5-264"
name: tom
Accept-Ranges: bytes
```

访问/a 目录 使用自己的 add header

```bash
root@elk:/var/log/nginx# curl -L -I localhost/a
HTTP/1.1 301 Moved Permanently
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:35:49 GMT
Content-Type: text/html
Content-Length: 169
Location: http://localhost/a/
Connection: keep-alive
name: tomcat

HTTP/1.1 200 OK
Server: nginx/1.21.1
Date: Sat, 26 Mar 2022 15:35:49 GMT
Content-Type: text/html
Content-Length: 16
Last-Modified: Sat, 26 Mar 2022 15:29:01 GMT
Connection: keep-alive
ETag: "623f313d-10"
name: tomcat
Accept-Ranges: bytes
```