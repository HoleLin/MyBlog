---
title: Linux-基础
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

