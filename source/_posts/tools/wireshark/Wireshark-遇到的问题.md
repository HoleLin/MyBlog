---
title: Wireshark-遇到的问题
date: 2022-02-25 10:01:37
index_img: /img/cover/Wireshark.jpeg
cover: /img/cover/Wireshark.jpeg
tags:
- questions
categories:
- 工具
- WireShark
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

* [Mac 安装wireshark](https://blog.csdn.net/qq_38376348/article/details/121419684)

### Mac安装WireShark提示"You can fix this by installing ChmodBPF."

<img src="https://www.holelin.cn/img/tools/wireshark/Wireshark-chmodbpf.png" alt="img" style="zoom:50%;" />

* 解决办法: 

  ```sh
  sudo chmod 777 /dev/bpf*
  ```

  
