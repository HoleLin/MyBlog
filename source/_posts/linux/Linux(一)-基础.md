---
title: Linux(一)-基础
date: 2021-07-18 15:09:28
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 基础
categories:
- Linux
---

### 参考文献

* 鸟哥的Linux私房菜基础学习篇(第四版)

#### 基础命令

* 显示日志与时间: `date`

* 显示日历: `cal`

* 计算器: `bc`

* 帮助命令: 

  * `man <command>` : `DATE(1)`

    | 代号 | 说明                                                         |
    | ---- | ------------------------------------------------------------ |
    | 1    | 用户在`Shell`环境中可以操作的命令或者可执行文件              |
    | 2    | 系统内核可调用的函数与工具等                                 |
    | 3    | 一些常用的函数(function)与函数库(library),大部分为C的函数库(libc) |
    | 4    | 设备文件的说明,通常在/dev下的文件                            |
    | 5    | 配置文件或是某些文件的格式                                   |
    | 6    | 游戏(games)                                                  |
    | 7    | 惯例与协议等.例如Linux文件系统,网络协议,ASCII代码等的说明    |
    | 8    | 系统管理员可使用的管理命令                                   |
    | 9    | 跟内核有关的文件                                             |

  * `<command> --help`

  * `info <command>`

  * 说明文档路径: `/usr/share/doc/`

* 获取终端支持的语系数据库:

  * `echo $LANG`

  * `locale`

    ```
    LANG=en_US.UTF-8
    LC_CTYPE="en_US.UTF-8"
    LC_NUMERIC="en_US.UTF-8"
    LC_TIME="en_US.UTF-8"
    LC_COLLATE="en_US.UTF-8"
    LC_MONETARY="en_US.UTF-8"
    LC_MESSAGES="en_US.UTF-8"
    LC_PAPER="en_US.UTF-8"
    LC_NAME="en_US.UTF-8"
    LC_ADDRESS="en_US.UTF-8"
    LC_TELEPHONE="en_US.UTF-8"
    LC_MEASUREMENT="en_US.UTF-8"
    LC_IDENTIFICATION="en_US.UTF-8"
    LC_ALL=
    ```


##### 正确的关机方法

* 观察系统的使用状态

  * 如果要看目前有谁在线,可以执行`who`命令;
  * 如果要看网络的联机状态,可以执行`netstat -a`命令;
  * 如果要看后台执行的程序,可以执行`ps -aux`命令;

* 正确的关机命令的使用

  * 将数据同步到硬盘: `sync`

  * 常用的关机命令: `shutdown`

    * 关机只有root才有权限使用;

    * `shutdown`可以完成以下的工作

      * 可以自由选择关机模式: 是要关机或重启均可;
      * 可以设置关机时间: 可以设置成现在立刻关机,也可以设置某个特定的时间才关机
      * 可以设置自定关机信息: 在关机之前,可以将自己设置的信息发送给在线用户;
      * 可以仅发出告警信息

    * `/sbin/shutdown [-krhc] [时间] [告警信息]` 

      | 参数 | 说明                               |
      | ---- | ---------------------------------- |
      | -k   | 不要真的关机,只是发送告警信息出去  |
      | -r   | 在将系统的服务停掉之后就重新启动   |
      | -h   | 将系统的服务停掉后,立即关机        |
      | -c   | 取消已经在进行的`shutdown`命令内容 |
      | 时间 | 指定系统关机的时间                 |

    * 示例

      * 立即关机: `shutdown -h now`
      * 指定时间点进行关机: `shutdown -h 20:25`
      * 系统再过10分钟关机: `shutdown -h +10`
      * 立即重启: `shutdown -r now`

  * 重启,关机: `reboot`,`halt`,`poweroff`

    * `sync;sync;sync;reboot`
    * `halt`: 系统停止,屏幕可能会保留系统已停止的信息;
    * `poweroff`: 系统关机

#### systemd和System V init的区别和作用

| System V init 运行级别 | systemd目标名称                    | 作用             |
| ---------------------- | ---------------------------------- | ---------------- |
| 0                      | runlevel0.target.poweroff.target   | 关机             |
| 1                      | runlevel1.target.rescue.target     | 单用户模式       |
| 2                      | runlevel2.target.multi-user.target | 等同于级别3      |
| 3                      | runlevel3.target.multi-user.target | 多用户的文本界面 |
| 4                      | runlevel4.target.multi-user.target | 等同于级别3      |
| 5                      | runlevel5.target.graphical.target  | 多用的图形界面   |
| 6                      | runlevel6.target.reboot.target     | 重启             |
| emergency              | emergency.target                   | 紧急Shell        |

* 将系统切换为"多用户,无图形"模式,可直接用`ln`命令把多用户模式目标文件链接到`/etc/systemd/system`目录

  ```sh
  ln -sf /lib/systemd/multi-user.target /etc/systemd/system/default.target
  ```

#### systemctl管理服务的启动,重启,停止等常用命令

| System V init命令(RHEL 6系统) | systemctl 命令(RHEL 7系统)               | 作用                               |
| ----------------------------- | ---------------------------------------- | ---------------------------------- |
| `service foo start`           | systemctl start foo.service              | 启动服务                           |
| `service foo restart`         | systemctl restart foo.service            | 重启服务                           |
| `service foo stop`            | systemctl stop foo.service               | 停止服务                           |
| `service foo reload`          | systemctl reload foo.service             | 重新加载配置文件(不终止服务)       |
| `service foo status`          | systemctl status foo.service             | 查看服务状态                       |
| `chkconfig foo on`            | systemctl enable foo.service             | 开机启动                           |
| `chkconfig foo off`           | systemctl disable foo.service            | 开机不启动                         |
| `chkconfig foo`               | systemctl is-enabled foo.service         | 查看特定服务是否为开机自启动       |
| `chkconfig foo --list`        | systemctl list-unit-files --type=service | 查看各个级别下服务的启动和禁用情况 |

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

