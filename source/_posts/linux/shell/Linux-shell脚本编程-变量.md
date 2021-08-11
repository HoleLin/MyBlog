---
title: Linux-shell编程-变量
date: 2021-08-11 10:05:47
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 变量
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

* https://github.com/52fhy/shell-book/blob/master/chapter1.md

#### 定义变量

* 定义变量时，变量名不加美元符号($)

  ```sh
  variableName="value"
  ```

  * 注意，**变量名和等号之间不能有空格**
  * 变量名的命令遵循如下规则：
    * 首字母必须为字母(A-Z,a-z)
    * 中间不能有空格，可以使用下划线(_)
    * 不能使用标点符号
    * 不能使用bash里的关键字

* 在变量前面加`readonly` 命令可以将变量定义为只读变量，只读变量的值不能被改变。

* 使用 `unset` 命令可以删除变量。语法：

  ```
  unset variable_name
  ```

  * 变量被删除后不能再次使用；unset 命令不能删除只读变量。

#### 使用变量

* 使用一个定义过的变量，只要在变量名前面加美元符号($)

  ```sh
  name="holelin"
  echo $name
  echo ${name}
  ```

  * 变量名外面的花括号时可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界

#### 变量类型

*  **局部变量** 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
* **环境变量** 所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
* **shell变量** shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

#### 特殊变量

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| `$0` | 当前脚本的文件名                                             |
| `$n` | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是`$1`，第二个参数是`$2`。 |
| `$#` | 传递给脚本或函数的参数个数。                                 |
| `$*` | 传递给脚本或函数的所有参数。                                 |
| `$@` | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 `$*` 稍有不同 |
| `$?` | 上个命令的退出状态，或函数的返回值。                         |
| `$$` | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。 |

```sh
#!/bin/bash
echo "File Name: $0"
echo "First Parameter : $1"
echo "Second Parameter : $2"
echo "Quoted Values: $@"
echo "Quoted Values: $*"
echo "Total Number of Parameters : $#"
echo "This shell pid: $$"
echo "This return value: $?"

[root@holelin shellLearning]# ./myecho.sh 1 2 3 4 5
File Name: ./myecho.sh
First Parameter : 1
Second Parameter : 2
Quoted Values: 1 2 3 4 5
Quoted Values: 1 2 3 4 5
Total Number of Parameters : 5
This shell pid: 22472
This return value: 0
```

#### 命令替换

命令替换是指Shell可以先执行命令，将输出结果暂时保存，在适当的地方输出。

语法：

```
`command`
```

**注意是反引号，不是单引号，这个键位于 Esc 键下方。**

下面的例子中，将命令执行结果保存在变量中：

```sh
#!/bin/bash
DATE=`date`
echo "Date is $DATE"
```

#### 变量替换

* 变量替换可以根据变量的状态（是否为空、是否定义等）来改变它的值。

* 可以使用的变量替换形式：

| 形式              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `${var}`          | 变量本来的值                                                 |
| `${var:-word}`    | 如果变量 `var` 为空或已被删除(unset)，那么返回 word，但不改变 `var` 的值。 |
| `${var:=word}`    | 如果变量 `var` 为空或已被删除(unset)，那么返回 word，并将 `var` 的值设置为 word。 |
| `${var:?message}` | 如果变量 `var` 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 `var` 是否可以被正常赋值。若此替换出现在Shell脚本中，那么脚本将停止运行。 |
| `${var:+word}`    | 如果变量 `var` 被定义，那么返回 word，但不改变 var 的值。    |
