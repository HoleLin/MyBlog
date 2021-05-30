---
layout: yum
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

#### yum 常用操作

> Yum 基于RPM的软件包管理器
>
> 命令语法: `yum [options] [command] [package ...]`
>
> - **options：**可选，
>   - -h: 显示帮助信息
>   - -y: 对所有的提问都回答"yes"
>   - -c: 指定配置文件
>   - -q: 安静模式
>   - -v: 详细信息
>   - -d: 设置调试等级(0-10)
>   - -e: 设置错误等级(0-10)
>   - -R: 设置yum处理一个命令的最大等待时间
>   - -C: 完全从缓存中运行,而不去下载或更新任何头文件
> - **command：**要进行的操作。
>   - install: 安装rpm软件包
>   - update: 更新rpm软件包
>   - check-update: 检查是否有可用的更新rpm软件包
>   - remove: 删除指定的rpm软件包
>   - list: 显示软件包的信息
>   - search: 检查软件包信息
>   - info: 显示指定的rpm软件包的描述信息和概要信息
>   - clean: 清理yum过期的缓存
>   - shell: 进入yum的shell提示符
>   - resolvedep: 显示rpm软件包的依赖关系
>   - localinstall: 安装本地rpm软件包进行更新
>   - deplist: 显示rpm软件的所有依赖关系
> - **package：**安装的包名。

##### yum常用命令

- 列出所有可更新的软件清单命令：**yum check-update**
- 更新所有软件命令：**yum update**
- 仅安装指定的软件命令：**yum install <package_name>**
- 仅更新指定的软件命令：**yum update <package_name>**
- 列出所有可安裝的软件清单命令：**yum list**
- 删除软件包命令：**yum remove <package_name>**
- 查找软件包命令：**yum search <keyword>**
- 清除缓存命令:
  - **yum clean packages**: 清除缓存目录下的软件包
  - **yum clean headers**: 清除缓存目录下的 headers
  - **yum clean oldheaders**: 清除缓存目录下旧的 headers
  - **yum clean, yum clean all (= yum clean packages; yum clean oldheaders)** :清除缓存目录下的软件包及旧的 headers
