---
title: Nginx-配置HTTPS
date: 2021-11-06 12:06:15
index_img: /img/cover/Nginx.jpg
cover: /img/cover/Nginx.jpg
tags:
- 配置HTTPS
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

### 通过阿里云免费SSL证书配置HTTPS

* 登录阿里云网页端,进入SSL证书模块,选择免费证书,创建证书,填写好相关信息后,提交审核.

  ![img](https://www.holelin.cn/img/nginx/Nginx%E9%85%8D%E7%BD%AEHTTPS1.png)

* 审核通过后,下载证书,选择Nginx,可参考阿里云帮助手册,进行部署

* 把下载好的证书上传至服务器,可以在`nginx`相关目录中新建文件夹`cert`用于存放证书相关信息

* 然后,修改nginx的配置信息,可新建一个`conf.d`目录里面存放单独的配置文件

  ```
  mkdir conf.d
  chmod 777 conf.d
  # 新建配置文件
  vim xxx.xxx.cn.conf
  # 注意要保持server_name的值与之前申请的证书所填写的一致,不然会出现"您与此网站之间建立的连接并非完全安全"即证书域名和设置的域名不一致问题
  server {
      listen 443 ssl;
      server_name xx.xxxxx.cn; #需要将yourdomain.com替换成证书绑定的域名。
      ssl_certificate ../../cert/xx.xxxxx.cn.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
      ssl_certificate_key ../../cert/xx.xxxxx.cn.key; #需要将cert-file-name.key替换成已上传的证书私钥文件的名称。
      ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
      #表示使用的加密套件的类型。
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
      ssl_prefer_server_ciphers on;
      ssl_session_cache shared:SSL:10m;
      ssl_session_timeout 10m;
      error_page 497  https://$host$request_uri;
  
      #Location配置
      location / {
          proxy_set_header X-Rea $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_set_header X-Nginx-Proxy true;
          proxy_pass http://xxx.xxx.xxx.xx:xxxx;
          proxy_set_header X-Forwarded-Proto $scheme;
  
      }
  }
  # 在nginx.conf的http{}中添加
  include conf.d/*.conf;
  # 重启nginx
  nginx -s reload
  ```

  
