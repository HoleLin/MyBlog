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



