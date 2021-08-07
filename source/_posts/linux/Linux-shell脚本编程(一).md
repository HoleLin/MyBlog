---
title: Linux-shell脚本编程(一)
date: 2021-08-06 16:55:13
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

### 参考文献

* Linux Shell脚本攻略

#### 脚本开头

```sh
 # !/bin/bash
```

#### 查看环境变量

* 查看所有环境变量`env`

* 查看单个程序的环境变量

  * `pgrep <程序名称>`

  * `cat /proc/$PID/environ`

  * 示例

    ```sh
    [root@holelin ~]# pgrep java
    23758
    29629
    [root@holelin ~]# cat  /proc/23758/environ | tr '\0' '\n'
    SHELL=/bin/bash
    USER=jenkins
    PATH=/sbin:/usr/sbin:/bin:/usr/bin
    PWD=/
    LANG=en_US.UTF-8
    HOME=/var/lib/jenkins
    SHLVL=2
    LOGNAME=jenkins
    _=/usr/local/jdk1.8/bin/java
    ```



#### 获取字符串长度

```sh
[root@holelin ~]# echo ${#USER}
4
```

#### 识别当前的shell版本

```sh
[root@holelin ~]# echo $SHELL
/bin/bash
[root@holelin ~]# echo $0
-bash
```

#### 检查是否为超级用户

```sh
# !/bin/bash
if [ $UID -ne 0 ]; then
echo Non root user. Please run as root.
else
echo "Root user"
fi
```

#### 文件描述符和重定向

* **标准输入(stdin--0)**

* **标准输出(stdout--1)**

* **标准错误(stderr--2)**

  ```sh
  cat a* 2> errot.txt
  ```

* `>`:  清空文件后再写入内容；

* `>>`: 将内容追加到现有文件到尾部；

* **命令和不成功的命令**

  > * 当一个命令发生错误并退回，它会返回一个非0的退出状态；
  > * 而命令成功后，它会返回数字0.
  > * 退出状态可以从特殊变量`$?`中获取

* 文件重定向到命令

  ```sh
  cmd < file
  [root@holelin shellLearning]# cat < a.txt 
  123123
  ```

* 重定向脚本内部的文本块

  ```sh
  # !/bin/bash
  cat <<EOF>>log.txt
  This is a test log flie
  Function: System statistics
  EOF
  
  [root@holelin shellLearning]# cat log.txt 
  This is a test log flie
  Function: System statistics
  ```

* 自定义文件描述符

  * 使用`exec`命令创建自定义的文件描述符
    * 只读模式
    * 截断模式
    * 追加模式
  * `<`操作符用于从文件读取至`stdin`；
  * `>`操作符用于截断模式的文件写入(数据在目录文件内容被截断之后写入)；
  * `>>`操作符用于追加模式的文件写入(数据被添加到文件的现有内容中，而且该目标文件中原有的内容不会丢失)

* 隐藏输入

  * `-echo`:禁止将输出发送到终端

  ```sh
  # !/bin/bash
  echo -e "Enter password: "
  stty -echo
  read password
  stty echo
  echo
  echo Password read.
  ```

* 检查一组命令所花费的时间

  ```sh
  # !/bin/bash
  start=$(date +%s)
  commands;
  statements;
  
  end=$(date +%s)
  difference=$((end - start))
  echo Time taken to execute commands is $difference seconds.
  ```

* 在脚本中生成延时

  ```sh
  sleep no_of_seconds
  
  # !/bin/bash
  echo -n Count:
  tput sc
  
  count=0;
  while true;
  do
  if [ $count -lt 40 ]
  then let count++;
  sleep 1;
  tput rc
  tput ed
  echo -n $count;
  else exit 0;
  fi
  done
  ```

* 调试脚本

  ```sh
  bash -x script.sh
  ```

  * 运行带有-x标志的脚本能打印出所有执行的每一行命令以及当前状态。-x标识将脚本中执行过的每一行都输出到stdout.
  * 使用`set build-in`来启用或禁止调试打印。
    * `set -x`:在执行时显示参数和命令；
    * `set +x`:禁止调试；
    * `set -v`:当命令进行读取时显示输入；
    * `set +v`:禁止打印输入；

* 函数和参数

  ```sh
  function fname(){
  	statements;
  }
  fname2(){
  	stattements;
  }
  
  # 执行函数
  fname;
  # 传递参数
  fname arg1 arg2;
  
  fname3(){
   # 访问参数1和参数2
   echo $1, $2;
   # 以列表的方式一次性打印所有参数
   echo "$@";
   # 类似于$@,但是参数被作为单个实体
   echo "$*";
   return 0;
  }
  ```

* 导出函数

  * 函数也能像环境变量一样用export导出,这样一来,函数的作用域就可以扩展到子进程中

  ```sh
  export -f fname
  ```

* 利用子shell生成一个独立的进程

  * 子shell本身就是独立的进程,可以使用`()`操作符来定义一个子shell

  ```sh
  pwd;
  (cd /bin; ls);
  pwd;
  ```

* 通过引用子shell的方式保留空格和换行符

  ```
  # 假设在使用子shell或反引用的方法将命令的输出的读入一个变量中,可以将它放入双引号中,以保留空格和换行符(\n)
  out = "$(cat text.txt)"
  ```

#### 以不按回车键的当时读取字符"n"

* 语法

  ```sh
  # 从输入中读取n个字符并存入变量variable_name
  read -n number_of_chars variable_name
  # 用不回显的方式读取密码
  read -s var
  # 显示提示信息
  read -p "Enter input:" var
  # 在特定时限内读取输入
  read -t timeout var
  # 用定界符结束输入行
  read -d delim_charvar
  ```

#### 字段分割符合迭代器

* **内部字段分割符(Internal Field Separator,IFS)**

  ```sh
  # !/bin/bash
  line="root:x:0:0:root:/root:/bin/bash"
  oldIFS=$IFS
  IFS=":"
  count=0
  for item in $line;
  do
    [ $count -eq 0 ] && user=$item;
    [ $count -eq 6 ] && shell=$item;
    let count++
  done;
  IFS=$oldIFS
  echo $user\'s shell is $shell
  ```

#### 循环

* for循环

  ```sh
  for var in list;
  do 
  	commands;
  done
  ```

* while循环

  ```sh
  while condition
  do 
  	commands
  done
  ```

#### 分支

* if

  ```
  if condition
  then
   commands;
  fi
  ```

* if else

  ```
  if condition
  then
   commands;
  elif condition
  then
   commands
  else
   commands
  fi
  ```

#### 算术比较

* 条件通常被放置在一个封闭的中括号内,一定要注意在`[ 或 ]`与操作数之间有一个空格.

  ```sh
  [ $var -eq 0 ] or [ $var -eq 0 ]
  ```

* `-gt`:大于

* `-lt`:小于

* `-ge`:大于或等于

* `le`:小于或等于

* `-a`:逻辑与

* `-o`:逻辑或

#### 文件系统相关

* 如果给定的变量包含正常的文件路径或文件名,则返回真: `[ -f $file_var ]`
* 如果给定的变量包含的文件可执行,则返回真: `[ -x $var ]`
* 如果给定的变量包含的是目录，则返回真:`[ -d $var]`
* 如果给定的变量包含的是文件存在，则返回真:`[ -e $var ]`
* 如果给定的变量包含的是一个字符设备文件的路径，则返回真:`[ -c $var ]`
* 如果给定的变量包含的是一个块设备文件的路径，则返回真:`[ -b $var ]`
* 如果给定的变量的文件可写，则返回真:`[ -w $var ]`
* 如果给定的变量的文件可读，则返回真:`[-r $var ]`

* 如果给定的变量包含的是一个符合链接，则返回真:`[ -L $var ]`

```sh
sh# !/bin/bash
if [ -f $1 ] ; then 
echo this is file.
  if [ -e $1 ] ; then 
  echo this file is exist.
    if [ -w $1 ] ; then
        echo this file can be wirte.
    fi
    if [ -r $1 ] ; then 
        echo this file can be read.
    fi
  fi
fi
if [ -x $1 ] ; then
echo this can be execute.
fi
if [ -d $1 ] ; then
echo this is directory.
fi
if [ -c $1 ] ; then
echo this is char file.
fi
if [ -b $1 ] ; then
echo this is block file.
fi
if [ -L $1 ] ; then 
echo this is link file.
fi
```

#### 字符串比较

* 检查两个文件是否相同
  * `[[ $str1 = $str2 ]]`
    * `=`前后有各有一个空格
  * `[[ $str1 == $str2 ]]`
* 检查两个文件是否不同
  * `[[ $str1 != $str2 ]]`
* 大于｜小于
  * `[[ $str1 > $str2 ]]`
  * `[[ $str1 < $str2 ]]`

* 检查字符串是否是空字符串:`[[ -z $str1 ]]`
* 检查字符串是否是非空字符串:`[[ -n $str1 ]]`

```sh
# !/bin/bash -x
if [[ -z $1 ]] ; then 
 echo this str is  empty
fi
if [[ -n $1 ]] ; then 
 echo this str is not  empty
 if [[ -n $2 ]] ; then 
  if [[ $1 == $2 ]] ; then 
   echo str1 equals str2
  else
   echo str1 not equals str2
  fi 
 fi  
fi
```

