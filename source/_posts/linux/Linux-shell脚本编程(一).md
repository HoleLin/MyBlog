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

  ```sh
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

* **for循环**

  ```sh
  for var in list;
  do 
  	commands;
  done
  ```

* **while循环**

  ```sh
  while condition;
  do 
  	commands;
  done
  ```
  
* **until循环**

  ```sh
  until condition;
  do
  	commands;
  done
  ```

  * until 循环执行一系列命令直至条件为 true 时停止。until 循环与 while 循环在处理方式上刚好相反。一般while循环优于until循环，但在某些时候，也只是极少数情况下，until 循环更加有用。

#### 分支

* **if...fi**

  ```sh
  if condition;
  then
   commands;
  fi
  ```

* **if...else...fi**

  ```sh
  if condition;
  then
   commands;
  else
   commands;
  fi
  ```
  
* **if...elif...else...fi**

  ```sh
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

* case语句:`case...esac`

  ```sh
  #!/bin/bash/
  
  grade="B"
  
  case $grade in 
  	"A") echo "Very Good!";;
  	"B") echo "Good!";;
  	"C") echo "Come On!";;
  	*) 
  		echo "You Must Try!"
  		echo "Sorry!";;
  esac
  
  ```

  * 需要注意的是： **取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 `;;`。**`;;` 与其他语言中的 `break` 类似，意思是跳到整个 `case` 语句的最后。
  * 取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 `*` 捕获该值，再执行后面的命令。

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

```sh
操作符	说明	举例

-b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。

-c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -c $file ] 返回 false。

-d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ] 返回 false。

-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。

-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] 返回 false。

-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。

-p file	检测文件是否是具名管道，如果是，则返回 true。	[ -p $file ] 返回 false。

-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。

-r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。

-w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。

-x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。

-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。

-e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。

```

```sh
# !/bin/bash
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
* 检测字符串长度是否为0，为0返回 true:`[[ -z $str1 ]]`
* 检测字符串长度是否为0，不为0返回 true:`[[ -n $str1 ]]`
* 检测字符串是否为空，不为空返回 true:`[ $str1 ]`

```sh
# !/bin/bash -x
if [[ -z $1 ]]  then 
 echo this str is  empty
fi
if [[ -n $1 ]]  then 
 echo this str is not  empty
 if [[ -n $2 ]]  then 
  if [[ $1 == $2 ]]  then 
   echo str1 equals str2
  else
   echo str1 not equals str2
  fi 
 fi  
fi
```

