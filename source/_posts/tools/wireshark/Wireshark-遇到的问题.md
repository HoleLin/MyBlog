---
title: Wireshark-遇到的问题
date: 2022-02-25 10:01:37
index_img: /img/cover/Wireshark.jpeg
cover: /img/cover/Wireshark.jpeg
tags:
- question
categories:
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

* 

### Mac安装WireShark提示"You can fix this by installing ChmodBPF."

* 解决办法: 

  ```sh
  sudo chmod 777 /dev/bpf*
  ```

  
