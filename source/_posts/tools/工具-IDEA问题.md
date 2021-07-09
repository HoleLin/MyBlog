---
title: 工具-IDEA问题
mermaid: true
date: 2021-07-09 21:04:42
index_img: /img/cover/Tools.jpg
cover: /img/cover/Tools.jpg
tags:
- IDEA
categories:
- 工具
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

#### `Error running 'Application': Command line is too long. Shorten command line for Application or also for Spring Boot default configuration.`

* 解决办法:

  * 在IDEA的`.idea/workspace.xml`文件的`<component name="PropertiesComponent">`标签下加上

    ```xml
        <property name="dynamic.classpath" value="true" />
    ```

  ![img](http://www.chenjunlin.vip/img/tools/IDEA%E7%8E%AF%E5%A2%83%E5%91%BD%E4%BB%A4%E8%BF%87%E9%95%BF%E9%97%AE%E9%A2%98.png)

