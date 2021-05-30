---
layout: CentOS 配置yum
title: yum配置
date: 2021-05-30 16:25:54
tags: 
- yum
categories:
- Linux
---

### yum源描述

> yum需要一个yum库，也就是yum源。默认情况下，CentOS就有一个yum源。在/etc/yum.repos.d/目录下有一些默认的配置文件（可以将这些文件移到/opt下，或者直接在yum.repos.d/下重命名）。
>
> 　　首先要找一个yum库（源），然后确保本地有一个客户端（yum这个命令就是客户端），由yum程序去连接服务器。连接的方式是由配置文件决定的。通过编辑/etc/yum.repos.d/CentOS-Base.repo文件，可以修改设置。

#### yum配置源

* 首先备份系统自带yum源配置文件`/etc/yum.repos.d/CentOS-Base.repo`

  ```sh
  mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
  ```

* 进入yum源配置文件所在文件夹

  ```sh
  cd /etc/yum.repos.d
  ```

* 配置网易源

    * CentOS 7

      ```sh
      wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
      ```

    * CentOS 6

      ```sh
      wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
      ```

    * CentOS 5

      ```sh
      wget http://mirrors.163.com/.help/CentOS5-Base-163.repo
      ```

* 配置阿里源
    * CentOS 7

      ```sh
      wget http://mirrors.aliyun.com/repo/Centos-7.repo
      ```

    * CentOS 6

      ```sh
      wget http://mirrors.aliyun.com/repo/Centos-6.repo
      ```

    * CentOS 5

      ```sh
      wget http://mirrors.aliyun.com/repo/Centos-5.repo
      ```

* 运行`yum makecache`生成缓存
* 执行`yum -y update`

