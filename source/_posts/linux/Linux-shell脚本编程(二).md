---
title: Linux-shell脚本编程(二)
date: 2021-08-07 22:19:09
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- shell编程 
categories:
- Linux
updated: 2021-08-09 15:33:09
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

#### 用cat进行拼接

* `cat`: (concatenate)拼接,`cat`命令不仅可以读取文件和拼接数据，它还能够从标准输入中进行读取。要从标准输入中读取，就要使用管道操作符：

  ```sh
  OUTPUT_FROM_SOME COMMANDS | cat
  ehco 'Text through stdin' | cat - file.txt
  - 被作为来自stdin文本的文件名
  ```

  * `cat file1 file2 file3`

* 压缩空白行

  * `cat -s file.txt`
  * `cat file.txt | tr -s '\n'`

* 将制表符显示为^|

  * `cat -T file.txt`
  
* 显示行号:`cat -n file.txt`
  
#### 录制和回放终端

*  `script -t 2> timing.log -a output.session` 输入`exit`退出
  * `timing.log`文件用于存储时序信息，描述每个命令在何时运行
  * `out.session`文件用于存储命令输出。
  * `-t` 选项用于将时序数据`stderr`
  * `2>`则用于`stderr`重定向到`timing.log`

* 按播放命令序列输出:`scriptreplay timing.log output.session`

##### 实现原理

* `script`命令可以用于建立可在多个用户之间进行广播到视频会话。
  * Terminal1:`mkfifo srciptfifo`
  * Terminal2:`cat scriptfio`
  * Terminal1:`script -f scriptfifo`

#### 文件查找于文件列表

* 要列出当前目录以及子目录下所有的文件和文件夹:`find base_path`

  ```sh
  # '\n'作为用于分割文件的定界符
  find . -print
  # 指明'\0'作为定界符
  find . -print0
  ```

* 根据文件名或正则表达式匹配搜索

  ```sh
  find base_path -name "*.txt" -print
  
  find . -name ".txt" -print
  # 匹配名字时忽略字母大小
  find base_path -iname "*.txt" -print
  # 匹配多个条件可以采用OR条件操作
  find . \( -name "*.txt" -o -name "*.pdf" \) -print
  ```

* 否定参数

  ```sh
  find . ! -name "*.txt" -print
  ```

* 基于目录深度

  ```sh
  find -maxdepth/-mindepth 1 -type f -print
  ```

* 根据文件类型搜索

  ```
  find . -type d -print
  * 普通文件 f
  * 符号连接 l
  * 目录 		d
  * 字符设备 c
  * 块设备   b
  * 套接字   s
  * Fifo.   p
  ```

* 根据文件时间搜索

* * 访问时间(-atime):用户最近一次访问文件的时间；
  * 修改时间(-mtime):文件内容最后一次被修改的时间；
  * 变化时间(-ctime):文件元数据(metadata,例如权限或所有权)最后一次改变时间；
    * 计量单位为天

  ```sh
  # 打印出最近7天被访问过的所有文件
  find . -type f -atime -7 -print
  # 打印出恰好在七天前被访问过的所有文件
  find . -type f -atime 7 -print
  # 打印出访问时间超过七天的所有文件
  find . -type f -atime +7 -print
  ```

  * -amin(访问时间)
  * -mmin(修改时间)
  * -cmin(变化时间)
    * 计量单位为分钟

* 基于文件大小的搜索

  ```sh
  # 大于2K的文件
  find . -type f -size +2k
  # 小于2K的文件
  find . -type f -size -2k
  # 等于2K的文件
  find . -type f -size 2k
  ```
  
  * 块(b) 512字节
  * 字节(c)
  * 字(w) 2字节
  * 千字节(k)
  * 兆字节(M)
  * 吉字节(G)
  
* 删除匹配的文件

  ```sh
  find . -type f -name "*.txt" -delete
  ```

* 基于文件权限和所有权的匹配

  ```sh
  find . -type f -perm 644 -print
  ```

* 结合find执行命令或动作

  * find命令可以借助选项`-exec`与其他命令进行结合。

  ```sh
  # {}是一个特殊的字符串，与-exec选项结合使用，对于每个一个匹配的文件{}，会被替换成相应的文件名
  # find命令找到来那个文件test1.txt和test2.txt，其所有者均为slynux
  find . -type f -user root -exec chown slynux {} \;
  
  # 将给定目录中所有的C程序文件拼接起来写入单个文件all_c_files.txt
  # 使用>操作符，没有使用>>(追加)的原因是因为find命令的全部输出只是一个单数据流(stdin),而只有当多个数据流被追加到单个文件中的时候才有必要使用>>.
  find . -type f -name "*.c" -exec cat {} \;>all_c_files.txt
  
  # 将10天前的.txt文件复制到OLD目录中
  find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD \;
  ```

  * 无法在`-exec`参数中直接使用多个命令，只能够接受单个命令，但是可以吧多个命令写入到一个shell脚本中，再执行`-exec`

    ```
    -exec ./commads.sh {} \;
    ```

* 让find跳过特定的目录

  ```sh
  # \( -name ".git" -prune \) 用于排除
  find devel/souce_path \( -name ".git" -prune \) -o \( -type f -print\)
  ```

#### 玩转xargs

* xargs擅长将标准输入数据转换为命令行参数。xargs能够处理stdin并将其转换为特定命令的命令行参数。xargs也可以将单行或多行文本转换为其他格式。

* 将多行输入转换为单行输出

  ```sh
  cat cat.txt | xargs
  ```

* 将单行输入转换为多行输出

  ```sh
  cat cat.txt | xargs -n 3
  ```

* 读取stdin，将格式化参数传递给命令

  ```sh
  # !/bin/bash
  echo $*'#'
  
  # 格式
  INPUT | xargs -n X
  
  [root@holelin shellLearning]# cat cat.txt 
  args1 
  args2
  args3
  [root@holelin shellLearning]# cat cat.txt | xargs -n 1 ./xargs1.sh 
  args1#
  args2#
  args3#
  ```

  

