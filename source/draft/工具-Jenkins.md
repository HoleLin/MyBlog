---
title: 工具-Jenkins
date: 2021-07-17 23:35:29
index_img: /img/cover/Jenkins.jpg
cover: /img/cover/Jenkins.jpg
tags:
- Jenkins
categories:
- 工具
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

#### Jenkins安装

```sh
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins java-11-openjdk-devel
sudo systemctl daemon-reload
```

* 配置文件: `/etc/sysconfig/jenkins`

  * 默认工作空间: `JENKINS_HOME="/var/lib/jenkins"`

* 启动目录: `/usr/lib/jenkins`

* 修改Java目录

  * 查看Java路径: `which java`

    ```java
    [root@holelin jenkins]# which java
    /usr/local/jdk1.8/bin/java
    ```

  * 配置: 

    ```
    [root@holelin jenkins]# vim /etc/rc.d/init.d/jenkins
    ```

    
