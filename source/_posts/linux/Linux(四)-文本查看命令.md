---
title: Linux(四)-文本查看命令
date: 2021-07-18 20:24:59
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 文本查看命令
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

#### `cat`

* 文本内容显示到终端

```sh
cat demo.yml
```

#### `head`

* 查看文件开头

```sh
head demo.yml
head -5 demo.yml
```

#### `tail`

* 查看文件结尾
* 常用参数`-f`文件内容更新后,显示信息同步更新

#### `wc`

* 统计文件内容信息

```sh
wc -l demo.yml
```

#### `grep`

* 检索文本内容

  ```sh
  grep -i "待查询的字符串信息" filename
  ```

* 说明：`grep`能在文件中检索特定内容

  - `-i`：大小写敏感
  - `-A`/`-B`/`-C` `<N>`：顺带显示前后文，`-A`表示后面 N 行，`-B`表示前面 N 行，`-C`表示前后各 N 行
  - `-E`：使用正则表达式来匹配
  - `-v`：反选（输出不匹配的行）
  - `-l`：只输出能匹配到内容的**文件名**
  - `-F`：不要将检索内容视为正则表达式
  - `-r`：递归匹配目录下所有文件的内容
  - `-o`：只输出匹配上了的部分（而不是整行）
  - `-a`：也对二进制文件进行检索，而不是忽略它们！



