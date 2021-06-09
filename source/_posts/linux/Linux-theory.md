---
title: Linux理论
date: 2021-05-30 17:17:47
index_img: /img/cover/Linux.jpg
tags:
- linux
- 理论
categories:
- Linux
---

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
| service foo start             | systemctl start foo.service              | 启动服务                           |
| service foo restart           | systemctl restart foo.service            | 重启服务                           |
| service foo stop              | systemctl stop foo.service               | 停止服务                           |
| service foo reload            | systemctl reload foo.service             | 重新加载配置文件(不终止服务)       |
| service foo status            | systemctl status foo.service             | 查看服务状态                       |
| chkconfig foo on              | systemctl enable foo.service             | 开机启动                           |
| chkconfig foo off             | systemctl disable foo.service            | 开机不启动                         |
| chkconfig foo                 | systemctl is-enabled foo.service         | 查看特定服务是否为开机自启动       |
| chkconfig foo --list          | systemctl list-unit-files --type=service | 查看各个级别下服务的启动和禁用情况 |

