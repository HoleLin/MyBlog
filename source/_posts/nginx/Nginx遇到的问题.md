---
title: Nginx遇到问题
date: 2021-05-24 23:33:33
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags: 
- Redis
- 主从复制
categories: 
- Redis
mermaid: true
---

## 1. 起始页为"403 Forbidden"

* 由于启动用户和nginx工作用户不一致所致

  * 看nginx的启动用户，发现是nobody，而为是用root启动的

  * `ps aux | grep "nginx: worker process" | awk '{print $1}'`

  * 解决方法: 将nginx.config的user改为和启动用户一致

    ```shell
    vi conf/nginx.conf
    // 修改 user 配置项
    ```

* 缺少index.html或者index.php文件，就是配置文件中index index.html index.htm这行中的指定的文件。

* 权限问题，如果nginx没有web目录的操作权限，也会出现403错误。

  * 解决办法：修改web目录的读写权限，或者是把nginx的启动用户改成目录的所属用户，重启Nginx即可解决

* SELinux设置为开启状态（enabled）的原因。

  * 查看当前selinux的状态 ` /usr/sbin/sestatus`

  * 将SELINUX=enforcing 修改为 SELINUX=disabled 状态 `vi /etc/selinux/config`

    > \#SELINUX=enforcing
    >
    >  SELINUX=disabled

  * 重启生效 

  