---
title: Linux-Shell脚本编程
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

### 补充内容

##### 获取字符串长度

```sh
[root@holelin ~]# echo ${#USER}
4
```

##### 识别当前的shell版本

```sh
[root@holelin ~]# echo $SHELL
/bin/bash
[root@holelin ~]# echo $0
-bash
```

##### 检查是否为超级用户

```sh
# !/bin/bash
if [ $UID -ne 0 ]; then
echo Non root user. Please run as root.
else
echo "Root user"
fi
```

### 文件描述符和重定向

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

