---
title: Linux-文件和目录管理
date: 2021-07-18 20:24:59
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 文件/目录管理
categories:
- Linux
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

* 鸟哥的Linux私房菜基础学习篇(第四版)

#### 目录和路径

* 相对路径和绝对路径
  * **绝对路径**: 路径的写法**一定由根目录/写起**,例如`/usr/share/doc`这个目录;
  * **相对路径**: 路径的写法**不是由/写起**,例如由`cd ../home`

#### 目录的相关操作

* `.`: 代表此层目录;
* `..`: 代表上一层目录;
* `-`: 代表前一个工作目录;
* `~`: 代表目前使用者身份所在的家目录;
* `~holelin`: 代表`holelin`这个使用者的家目录
* `cd`: (change directory)切换目录
* `pwd`: (print working directory)显示当前目录
* `mkdir`: (make directory)建立一个新目录
* `rmdir`: 删除一个空目录

#### 文件与目录管理

* `ls`: 文件与目录的查看

  ```sh
  ls [-aAdfFhilnrRSt] 文件名或目录名称
  ls [--color={never,auto,always}] 文件名或目录名称
  ls [--full-time] 文件名或目录名称
  ```

  * **`-a`: 全部的文件,连同隐藏文件一起列出来**

  * `-A`: 全部的文件,连同隐藏文件一起列出来,但不包括`.`和`..`这个两个目录

  * **`-d`: 仅列出目录本身,而不是列出目录内的文件数据**

  * `-f`: 直接列出目录结果,而不进行排序(`ls` 默认会以文件名称进行排序)

  * `-F`: 根据文件,目录等信息,给予附加数据结构,例如

    * `*`:代表可执行文件
    * `/`: 代表目录
    * `=`: 代表socket文件
    * `|`:代表FIFO文件

  * `-h`: 将文件容器以人类较易读的方式(如GB,KB)

  * **`-l`: 详细信息展示,包含文件的属性与权限等数据**

  * `-i`: 列出`inode`号码

  * `-n`: 列出`UID`和`GID`而非使用者与用户组的名称

  * `-r`: 将排序结果反向输出

  * `-R`: 连同子目录内容一起列出来,等于该目录下的所有文件都会显示出来

  * `-S`: 以文件容量大小排序,而不是文件名排序

  * `-t`: 依照时间排序,而不是用文件名

  * `--color=never`: 不要依据文件特性给予颜色显示

  * `--color=always`: 显示颜色

  * `--color=auto`: 让系统自行依据设置来判断是否给予颜色

  * `--full-time`: 以完整时间模式(包含年,月,日,时,分)输出

  * `--time={atime,ctime}`: 输出access时间或改变权限属性时间(`ctime`),而非内容修改时间(`modification time`)

* 复制,删除与移动: `cp`,`rm`,`mv`

  * `cp`: 除了单纯的复制之外,还可以建立链接文件(快捷方式),对比两文件的新旧而予更新,以及复制整个目录等功能

    ```sh
    cp [-adfilprsu] 源文件(source) 目标文件(destination)
    cp [options] source1 source2 source3 .... directory
    ```

    * `-a`: 相当于`-dr --preserve=all`的意思
    * `-d`: 若源文件为链接文件的属性,则复制链接文件属性而非文件本身;
    * `-f`: 为了强制(force),若目标文件已经存在且无法开启,则删除后再尝试一下;
    * `-i`: 若目标文件(destination)已经存在时,在覆盖时会优先询问操作的进行;
    * `-l`: 进行硬链接的链接文件建立,而非复制文件本身;
    * `-p`: 连同文件的属性(权限,用户,时间)一起复制过去,而非使用默认属性(备份时常用)
    * `-r`: 递归复制,用于目录的复制操作
    * `-s`: 复制称为符号链接文件(symbol link),即"快捷方式"
    * `-u`: destination比source旧才更新destination,或destination不存在的情况下才能复制
    * `--preserve=all`: 除了`-p`的权限相关参数外,还加入`SELinux`的属性,`links`,`xattr`等也复制
    * **注意:**如果源文件有两个以上,则最后一个目录文件一定是"目录"才行;
    * **在默认的条件中,`cp`的源文件与目标文件的权限是不同的,目标文件的拥有者通常会是命令操作者本身**

  * `mv`: 移动目录和文件,还可以做重命名的查找

    ```sh
    mv [-fiu] source destination
    mv [options] source1 source2 source3 ... directory
    ```

    * `-f`: 如果目标文件已经存在,不会询问而直接覆盖;
    * `-i`: 若目标文,已经存在时,就会询问是否覆盖;
    * `-u`: 若目标文件已经存在,且source比较新,才会更新(update)

  * `rm`: 删除

    ```sh
    rm [-fir] 文件或目录
    ```

    * `-f`: 就是`force`,忽略不存在的文件,不会出现告警信息
    * `-i`: 交互模式,在删除前会询问使用者是否操作
    * `-f`: 递归删除,最常用于目录的删除

    



