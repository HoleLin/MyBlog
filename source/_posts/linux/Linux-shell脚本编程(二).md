---
title: Linux-shell脚本编程(二)
date: 2021-08-07 22:19:09
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- shell编程 
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

  
