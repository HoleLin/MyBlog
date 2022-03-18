---
title: 运维-禁止UbuntuServer自动休眠
date: 2022-03-18 16:36:56
tags:
- Ubuntu
categories:
- 运维
- Ubuntu
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

* [How To: Disable Sleep on Ubuntu Server](https://www.unixtutorial.org/disable-sleep-on-ubuntu-server/)

### 禁止Ubuntu server 自动休眠

> IDC机房机器重启20分钟左右会自动断开连接，网络就直接连接不上

查看系统日志

```bash
Mar  7 07:30:54 dn-idc102 NetworkManager[1935]: <info>  [1646638254.4260] manager: sleep: sleep requested (sleeping: no  enabled: yes)
Mar  7 07:30:54 dn-idc102 NetworkManager[1935]: <info>  [1646638254.4262] manager: NetworkManager state is now ASLEEP
Mar  7 07:30:54 dn-idc102 ModemManager[2048]: <info>  [sleep-monitor] system is about to suspend
Mar  7 07:30:54 dn-idc102 gnome-shell[2972]: Screen lock is locked down, not locking
Mar  7 07:30:54 dn-idc102 systemd[1]: Reached target Sleep.
Mar  7 07:30:54 dn-idc102 systemd[1]: Starting Suspend...
Mar  7 07:30:54 dn-idc102 systemd-sleep[3714]: Suspending system...
```

触发了`systemd`的自动休眠功能，检查休眠功能的状态以及历史记录，如下：

```bash
$ systemctl status sleep.target
```

作为服务器使用的时候，我们一般远程访问系统，这个功能就会导致我们无法远程控制服务器，因此我们需要关闭这个功能。

执行关闭休眠功能的命令，如下：

```bash
$ sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
 Created symlink /etc/systemd/system/sleep.target → /dev/null.
 Created symlink /etc/systemd/system/suspend.target → /dev/null.
 Created symlink /etc/systemd/system/hibernate.target → /dev/null.
 Created symlink /etc/systemd/system/hybrid-sleep.target → /dev/null.
```

再次观察系统休眠状态，如下：

```bash
	
$ systemctl status sleep.target
● sleep.target
   Loaded: masked (Reason: Unit sleep.target is masked.)
   Active: inactive (dead)
```
