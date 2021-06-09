---
title: Linux常用命令
date: 2021-05-30 16:52:41
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg

tags:
- linux
- 常用命令
categories:
- Linux
---

#### Linux 查看内核版本

* `cat /proc/version`

  ```sh
  [root@holelin ~]# cat /proc/version
  Linux version 3.10.0-1127.19.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) ) #1 SMP Tue Aug 25 17:23:54 UTC 2020
  ```

* `uname -a`

  ```sh
  [root@holelin ~]# uname -a
  Linux holelin 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
  ```

#### Linux查看系统版本命令

* `lsb_release -a` 列出所有版本信息

  ```sh
  [root@holelin ~]# lsb_release -a
  LSB Version:    :core-4.1-amd64:core-4.1-noarch:cxx-4.1-amd64:cxx-4.1-noarch:desktop-4.1-amd64:desktop-4.1-noarch:languages-4.1-amd64:languages-4.1-noarch:printing-4.1-amd64:printing-4.1-noarch
  Distributor ID: CentOS
  Description:    CentOS Linux release 7.9.2009 (Core)
  Release:        7.9.2009
  Codename:       Core
  
  ```

* `cat /etc/redhat-release `

  ```sh
  [root@holelin ~]# cat /etc/redhat-release
  CentOS Linux release 7.9.2009 (Core)
  ```

* `cat /etc/issue`

  > 只在centos6 看到系统版本号

#### VM虚拟机三种网络模式

* **桥接模式**

  > 相当于在物理主机与虚拟机网卡之间架设一座桥梁,从而可以通过物理主机的网卡访问外网.

* **NAT模式**

  > Network Address Translation 意即网络地址转换
  >
  > 让VM虚拟机的网络服务发挥路由的作用,使得通过虚拟机软件模拟的主机可以通过物理主机可以通过物理主机访问外网,在真机中NAT虚拟机网卡的物理网卡是VMnet8

* **仅主机模式**

  > 仅让虚拟机内的主机与物理主机通信,不能访问外网,在真机中仅主机模式对应的物理网卡是VMnet1

#### Wget常用命令

> 用于下载网络文件
>
> 命令格式: wget 参数 下载地址
>
> 参数: 
>
> * -b: 后台下载模式
> * -O: 下载到指定目录
> * -t: 最大尝试次数
> * -c: 断点续传
> * -p: 下载页面内所有资源,包括图片,视频等
> * -r: 递归下载



