---
title: Linux-shell脚本编程(二)
date: 2021-08-07 22:19:09
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- shell编程 
categories:
- Linux
updated: 2021-08-10 18:33:09
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
  # xargs有一个选项-I，可以用-I指定一个替换字符串，这个字符串在xargs扩展时被替换掉，当-I与xargs结合使用时，对于每一个参数，命令都会被执行一次
  [root@holelin shellLearning]# cat cat.txt | xargs -I {} ./xargs1.sh -p {} -1
  -p args1 -1#
  -p args2 -1#
  -p args3 -1#
  ```
  

#### 用tr进行转换

* `tr`可以对来自标准输入的字符进行替换，删除以及压缩。可以将一组字符变为另一组字符，因而通常也被称为转换(translate)命令

* `tr`只能通过`stdin`，而无法通过命令行参数来接受输入

  ```sh
  # 将来自stdin的输入字符从set1映射到set2，并将其输出写入stdout。set1和set2是字符类或字符集。
  # 如果两个字符集到长度不相等，那么set2会不断重复其最后一个字符，直到长度与set1相同。
  # 如果set2长度大于set1，那么在set2中超出set1长度的那部分字符则全部被忽略
  tr [options] set1 set2
  
  [root@holelin shellLearning]# echo  "HELLO WHO IS THIS" | tr 'A-Z' 'a-z'
  hello who is this
  [root@holelin shellLearning]# echo  "HELLO WHO IS THIS" | tr 'A-C' 'a-b'
  bbbbb bbb bb bbbb
  [root@holelin shellLearning]# echo  "HELLO WHO IS THIS" | tr 'A-B' 'a-z'
  HELLO WHO IS THIS
  [root@holelin shellLearning]# echo  "HELLO WHO IS THIS" | tr 'A-H' 'a-z'
  heLLO WhO IS ThIS
  ```

* 用tr删除字符

  * `-d`可以通过指定需要被删除的字符集合，将出现在stdin中的特殊字符清除掉

  ```sh
  # 将stdin中的数字删除并打印出来
  echo "Hello 123 word 456" | tr -d '0-9'
  ```

* 字符集补集

  * set1的补集意味着这个集合中包含set1中没有的所有字符

  ```sh
  tr -c [set1] [set2] 
  
  # 从输入文本中将不再补集中的所有字符全部删除
  echo hello 1 char 2 next 4 | tr -d -c '0-9 \n'
  ```

* 用tr压缩字符

  * tr命令在很多文本处理环境中相当有用。多数情况下，连续的重复字符应该被压缩成单个字符
  * tr的`-s`选项可以压缩输入中重复的字符

  ```sh
  [root@holelin shellLearning]# echo "GUN is   not  UNIX. Recursive right ?" | tr -s ' '
  GUN is not UNIX. Recursive right ?
  
  [root@holelin shellLearning]# cat sum.txt 
  1
  2
  3
  4
  5
  6
  [root@holelin shellLearning]# cat sum.txt | echo $[ $(tr '\n' '+' ) 0 ]
  21
  ```

  * 字符类

    * `alnum`:字母和数字

    * `alpha`:字母

    * `cntrl`:控制(非打印)字符

    * `digit`:数字

    * `graph`:图形字符

    * `lower`:小写字母

    * `print`: 可打印字符

    * `punct`:标点符号

    * `space`:空白字符

    * `upper`:大写字母

    * `xdigit`:十六进制字符

    * 使用格式

      ```sh
      tr [:class:] [:class:]
      
      tr '[:lower:]' '[:upper:]'
      
      [root@holelin shellLearning]# echo "Hello 123 word 456" | tr '[:upper:]' '[:lower:]'
      hello 123 word 456
      ```

#### 校验和与核实

  * `md5sum`和`sha1sum`

    ```sh
    [root@holelin shellLearning]# md5sum cat.md5 cat.txt > cats.md5
    # 输出校验和是否匹配的消息
    [root@holelin shellLearning]# md5sum -c cats.md5 
    cat.md5: OK
    cat.txt: OK
    
    [root@holelin shellLearning]# sha1sum cat.md5  cat.txt > cats.sha1
    [root@holelin shellLearning]# sha1sum -c cats.sha1 
    cat.md5: OK
    cat.txt: OK
    ```

* 对目录进行校验

  * `md5deep`或`sha1deep`

  ```sh
  # -r 使用递归方式
  # -l 使用相对路径，默认情况下，md5deep会输出文件的绝对路径
  md5deep -rl directory_path > directory.md5
  # 结合find来递归计算校验和
  find direcotry_path -type f -print0 | xargs -0 md5sum >> direcotry.md5
  ```

#### 排序，单一与重复

* `sort`对文本文件和stdin进行排序操作

```sh
[root@holelin md5andsha1]# sort  1.txt 2.txt 3.txt  > sorted.txt
[root@holelin md5andsha1]# cat  sorted.txt 
12312
12312
12312
[root@holelin md5andsha1]# cat  sorted.txt | uniq > uniq_line.txt
[root@holelin md5andsha1]# cat uniq_line.txt 
12312
```

* 按数字进行排序:`sort -n file.txt`

* 按逆序进行排序:`sort -r file.txt`

* 按照月份进行排序(按照一月，二月，三月....):`sort -M months.txt`

* 检测一个文件是否被排过序

  ```sh
  #!/bin/bash
  sort -C file;
  if [ $? -eq 0 ] ; then
  	echo Sorted;
  else
  	echo Unsorted;
  fi
  ```

* 若需要合并两个排过序的文件，而且不需要对合并后的文件在进行排序:`sort -m sorted1 sorted2`

#### 分割文件和数据

* 生成一个大小为100KB的测试文件(data.file)

  ```sh
  dd if=/dev/zero bs=10k count=1 of=data.file
  ```

* 给分割后的文件设置数字尾缀

  ```sh
  # -a length指定后缀长度
  [root@holelin split]# split -b 10k data.file -d -a 4
  # 指定文件前缀lin_
  [root@holelin split]# split -b 1k data.file -d -a 4 lin_
  [root@holelin split]# ll
  total 92
  -rw-r--r-- 1 root root 10240 Aug 11 16:00 data.file
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0000
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0001
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0002
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0003
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0004
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0005
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0006
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0007
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0008
  -rw-r--r-- 1 root root  1024 Aug 11 16:02 lin_0009
  # 按照行进行分割文件
  [root@holelin split]# split -l 10 data.file 
  # /[REGEX]/表示文本样式，包括从当前行(第一行)直到(但不包括)包含“SERVER”的匹配行
  # {*}表示根据匹配重复执行分割，直到文件末尾为止，可以用{整数}的形式来指定分割执行行术
  # -s使命令进入静默模式，不打印其他信息
  # -n指定分割后的文件名后缀的数字个数
  # -f指定分割后的文件名前缀
  # -b指定后缀格式
  [root@holelin split]# csplit data.file /SERVER/ -n 2 -s {*} -f data -b "%02d.log%" ; rm data00.log
  ```

* 提取文件名

  ```sh
  file_jpg="sample.jpg"
  name=${file_jpg%.*}
  echo File name is :$name
  ```

* 提取扩展名

  ```sh
  extension=${file_jpg#*.}
  echo Extension is :$extension
  ```

  ```sh
  [root@holelin split]# URL=www.baidu.com
  # 移除.*所匹配的最右边的内容
  [root@holelin split]# echo ${URL%.*}
  www.baidu
  # 将从右边开始一直匹配到最左边的*.移除（贪婪操作符）
  [root@holelin split]# echo ${URL%%.*}
  www
  # 移除*.所匹配的最左边的内容
  [root@holelin split]# echo ${URL#*.}
  baidu.com
  # 将从左边开始一直匹配到最右边的*.移除（贪婪操作符）
  [root@holelin split]# echo ${URL##*.}
  com
  ```

  

  

  

