---
title: Nginx-rewrite
mermaid: true
date: 2022-03-23 17:34:11
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
- rewrite
categories:
- Nginx
- rewrite
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
### nginx rewrite

> rewrite模块即`ngx_http_rewrite_module`模块，主要功能是改写请求URI，是Nginx默认安装的模块。rewrite模块会根据PCRE正则匹配重写URI，然后发起内部跳转再匹配location，或者直接做30x重定向返回客户端。
> 
#### rewrite 语法

```nginx
rewrite regex replacement [flag]
```

* regex：表示正则匹配规则。

* replacement：表示跳转后的内容。

* flag：表示 rewrite 支持的 flag 标记。

  * flag标记说明:

    * `last`：本条规则匹配完成后，继续向下匹配新的location URL规则，一般用在server和 if 中。

    * `break`：本条规则匹配完成即终止，不再匹配后面的任何规则，一般使用在location中。

    * `redirect`：返回302临时重定向，浏览器地址会显示跳转后的URL地址。

    * `permanent`：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址。

      * 注：last和break最大的不同在于

        - break 是终止当前location的rewrite检测,而且不再进行location匹配

        - last 是终止当前location的rewrite检测,但会继续重试location匹配并处理区块中的rewrite规则


* 位置
  ```
  Syntax:	break;
  Default:	—
  Context:	server, location, if
  ```

#### rewrite 模块的两大功能

- 实现网站地址信息跳转
- 实现伪静态

#### 常用nginx 正则表达式

| regex       | 说明                                                         |
| --- | --- |
| ^ | 匹配输入字符串的起始位置 |
| $ | 匹配输入字符串的结束位置 |
| * | 匹配前面的字符零次或多次。如“ol*”能匹配“o”及“ol”、“oll” |
| + | 匹配前面的字符一次或多次。如“ol+”能匹配“ol”及“oll”、“olll”，但不能匹配“o” |
| ? | 匹配前面的字符零次或一次，例如“do(es)?”能匹配“do”或者“does”，”?”等效于”{0,1}” |
| . | 匹配除“\n”之外的任何单个字符，若要匹配包括“\n”在内的任意字符，请使用诸如“[.\n]”之类的模式 |
| \ | 将后面接着的字符标记为一个特殊字符或一个原义字符或一个向后引用。如“\n”匹配一个换行符，而“\$”则匹配“$” |
| \d | 匹配纯数字 |
| \w | 匹配字母活数字或下划线或汉字 |
| \s | 匹配任意换行符 |
| \b | 匹配单词的开始或结束 |
| {n} | 重复 n 次 |
| {n,}  | 重复 n 次或更多次 |
| {n,m}  | 重复 n 到 m 次 |
| []  | 定义匹配的字符范围 |
| [a-z]  | 匹配 a-z 小写字母的任意一个 |
| [a-zA-Z0-9]  | 匹配所有大小写字母或数字 |
| () |  表达式的开始和结束位置 例如：(jpg|

#### rewrite 执行顺序

1. 执行 `server` 模块里面的`rewrite` 指令
2. 执行选定的`location 中的`rewrite`指令
3. 执行选定的`location`中`if`中的`rewrite`指令

```nginx
http {
    server {
        # 优先级1 rewrite
        location ~*\.(jpg|gif|swf)$ {
            # 优先级2 rewrite
            valid_referers none blocked *.example.com example.com;
            if ( $invalid_referer ) {
                rewrite ^/ http://www.example.com; #优先级3
            }
        }
    }
}
```

#### nginx 全局变量

| 变量 | 说明 |
| --- | --- |
| $args | 请求中的完整参数 |
| \$arg_PARAMETER \| 获取指定参数，如：$arg_name，就能获取到 name 参数 ||
| $binary_remote_addr | 二进制客户端地址，如：\x0A\xE0B\x0E |
| $body_bytes_sent | 向客户端发送 HTTP 响应中包体部分的字节数，前面日志中用过 |
| $content_length | 客户端请求头部中 Content_Length 字段 |
| $content_type | 客户端请求头部中 Content_Type 字段 |
| $cookie_COOKIE | 获取指定 cookie 的值 |
| $document_root | 当前请求所使用的 root 配置项的值 |
| $uri | 当前请求的 uri，不带任何参数 |
| $document_uri | 与 uri 含义相同 |
| \$request_uri \| 原始请求 uri，带完整的参数。$uri 和 $document_uri 可能是内部重定向后的。 ||
| $host | 请求头部 Host 字段，字段不存在，则以实际 server（虚拟主机）名称代替。 |
| $hostname | nginx 所在机器的名称。 |
| $http_HEADER | 当前 HTTP 请求相应头部的值，全小写 |
| $sent_http_HEADER | 返回客户端 HTTP 响应中相应头部的值 |
| $is_args | 请求中的 uri 是否带参数，如果带参数，值为 ?，如果不带参数，值为空 |
| $limit_rate | 当前连接限速为多少，0 表示不限速 |
| $nginx_version | 当前 nginx 的版本号 |
| $query_string | 请求 uri 中的参数，与 args 相同，只读的 |
| $remote_addr | 客户端地址 |
| $remote_port | 客户端连接使用端口 |
| $remote_user | 客户端连接使用账户，使用 auth basic module 时定义的用户名 |
| $request_filename | 请求中 uri 经过 root 或 alias 转换以后的路径 |
| $request_body | HTTP 请求中的包体，该参数只在 proxy_pass 或 fastcgi_pass 中有意义 |
| $request_body_file | HTTP 请求中的包体存储的临时文件名 |
| $request_completion | 请求全部完成时，值为"ok"，若没完成，就返回客户端，值为空 |
| $request_method | HTTP 请求的方法名，如get，put，post |
| $scheme | scheme，如请求：https://www.baidu.com，值为 https |
| $server_addr | 服务器地址 |
| $server_name | 服务器名称 |
| $server_port | 服务器端口 |
| $server_protocol | 服务器向客户端发送响应的协议，如 HTTP/1.1 或者 HTTP/1.0 表示不限速 |

#### 测试功能

##### 去除api前缀

```nginx
server {
    # 对外暴露 80 端口
    listen 80;
    server_name 192.168.253.251;

    # 后端API地址暴露为：http://192.168.253.251/api
    location /api {
        proxy_pass http://application:6060;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300
        rewrite "^/api/(.*)$" /$1 ;
    }
    location / {
        proxy_pass http://homepage:3000;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-real-ip $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

`rewrite "^/api/(.*)$" /$1 ` 路径重写：

```bash
1、"^/api/(.*)$"：匹配路径的正则表达式，用了分组语法就是(.*)，把/api/以后的所有部分当做1组；
2、/$1：重写的目标路径，这里用$1引用前面正则表达式匹配到的分组，即/api/后面的所有。这样新的路径就是除去/api/以外的所有
```

##### 访问结果

###### 没跳转前页面

###### ![img](https://www.holelin.cn/img/nginx/nginx-rewrite.png)跳转后的页面

![img](https://www.holelin.cn/img/nginx/nginx-rewrite1.png)