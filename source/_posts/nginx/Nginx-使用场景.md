---
title: Nginx-使用场景
mermaid: true
date: 2021-07-11 17:34:11
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
- 使用场景
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

### 参考文献

* [全面了解 Nginx 主要应用场景](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247491016&idx=1&sn=292d86dec1eb0766e94312c78b8ec425&chksm=ebd622e4dca1abf2c8452dc400ea07bda73cbe0eda5c4cecaaaa9f54ec5e7fe813a7272c81c6&scene=0&xtrack=1#rd)
* [彻底搞懂 Nginx 的五大应用场景](https://mp.weixin.qq.com/s/-NoWQKHP842BNzU_K7YRVQ)

#### `Nginx`主要使用场景

* 反向代理
* 负载均衡
* HTTP服务器(包含动静分离)
* 正向代理

#### 反向代理

* 反向代理应该是`Nginx`使用最多的功能了，**反向代理(Reverse Proxy)**方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

* 简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已;

* 反向代理通过`proxy_pass`指令来实现。

  ```
  server {
      listen       80;
      server_name  localhost;
      client_max_body_size 1024M;
  
      location / {
          proxy_pass http://localhost:8081;
          proxy_set_header Host $host:$server_port;
          # 设置用户ip地址
          proxy_set_header X-Forwarded-For $remote_addr;
          # 当请求服务器出错去寻找其他服务器
          proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; 
      }
  }  
  ```

#### 负载均衡

* 负载均衡也是`Nginx`常用的一个功能，负载均衡其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

* 简单而言就是当有2台或以上服务器时，根据规则随机的将请求分发到指定的服务器上处理，负载均衡配置一般都需要同时配置反向代理，通过反向代理跳转到负载均衡。而`Nginx`目前支持自带3种负载均衡策略，还有2种常用的第三方策略。

  * **RR（round robin:轮询 默认）**: 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

    ```
    	# 负载均衡的核心代码
    	upstream test {
            server localhost:8080;
            server localhost:8081;
        }
        server {
            listen       80;
            server_name  localhost;
            client_max_body_size 1024M;
    
            location / {
                proxy_pass http://localhost:8081;
                proxy_set_header Host $host:$server_port;
                # 设置用户ip地址
                proxy_set_header X-Forwarded-For $remote_addr;
                # 当请求服务器出错去寻找其他服务器
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; 
            }
        }  
    ```

    * 这里配置了2台服务器，当然实际上是一台，只是端口不一样而已，而8081的服务器是不存在的,也就是说访问不到，但是我们访问`http://localhost `的时候,也不会有问题，会默认跳转到`http://localhost:8080`
    * 具体是因为`Nginx`会自动判断服务器的状态，如果服务器处于不能访问（服务器挂了），就不会跳转到这台服务器，所以也避免了一台服务器挂了影响使用的情况，由于`Nginx`默认是RR策略，所以我们不需要其他更多的设置。

  * **权重** : 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况

    ```
       upstream test {
            server localhost:8080 weight=9;
            server localhost:8081 weight=1;
        }
    ```

  * `IP_Hash`: `IP_Hash`的每个请求按访问`IP`的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决`session`的问题

    ```
      upstream test {
            ip_hash;
            server localhost:8080;
            server localhost:8081;
        }
    ```

  * fair（第三方）:按后端服务器的响应时间来分配请求，响应时间短的优先分配。

    ```
        upstream backend { 
            fair; 
            server localhost:8080;
            server localhost:8081;
        } 
    ```

  * `url_hash`（第三方）

    * 按访问`url`的hash结果来分配请求，使每个`url`定向到同一个后端服务器，后端服务器为缓存时比较有效。在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法

    ```
       upstream backend { 
            hash $request_uri; 
            hash_method crc32; 
            server localhost:8080;
            server localhost:8081;
        } 
    ```

#### HTTP服务器

* `Nginx`本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用`Nginx`来做服务器，同时现在也很流行动静分离，就可以通过`Nginx`来实现，首先看看`Nginx`做静态资源服务器

  ```
  user holelin;
  
  http {
      server {
          listen       80;
          server_name  localhost;
          client_max_body_size 1024M;
  
          # 默认location
          location / {
              root   /usr/local/var/www/html;
              index  index.html index.htm;
          }
      }
  }
  ```

##### 动静分离

* 动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路

* 静态服务器location的映射一般有两种方式：

  - 使用路径，如 /images/ 一般图片都会放在某个图片目录下，
  - 使用后缀，如 `.jpg`、`.png` 等后缀匹配模式


  ```
  	upstream test{  
         server localhost:8080;  
         server localhost:8081;  
      }   
  
      server {  
          listen       80;  
          server_name  localhost;  
  
          location / {  
              root   /holelin/pic;  
              index  index.html;  
          }  
  
          # 所有静态请求都由nginx处理，存放目录为html  
          location ~ \.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {  
              root    /holelin/html;  
          }  
  
          # 所有动态请求都转发给tomcat处理  
          location ~ \.(jsp|do)$ {  
              proxy_pass  http://localhost:8080;  
          }  
  
          error_page   500 502 503 504  /50x.html;  
          location = /50x.html {  
              root   /holelin/error;  
          }  
      }  
  ```

#### 正向代理

* 正向代理，意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理。





