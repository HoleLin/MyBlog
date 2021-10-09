---
title: Linux(五)-备份压缩
date: 2021-07-18 20:24:59
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 备份压缩
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

####  Linux的备份压缩

* 最早的Linux备份介质是磁带,使用的命令是`tar`;
* 可以打包的磁带文件进行压缩存储,压缩命令是`gzip`和`bzip2`;
* 经常使用的扩展名是:`.tar.gz`,`.tar.bz2`,`.tgz`

#### `tar`格式

```sh
tar (选项) (参数)
-A或--catenate：新增文件到以存在的备份文件；
-B：设置区块大小；
-c或--create：建立新的备份文件；
-C <目录>：切换工作目录，先进入指定目录再执行压缩/解压缩操作，可用于仅压缩特定目录里的内容或解压缩到特定目录；
-d：记录文件的差别；
-x或--extract或--get：从归档文件中提取文件，可以搭配-C（大写）在特定目录解开，
                     需要注意的是-c、-t、-x不可同时出现在一串命令中；
-t或--list：列出备份文件的内容；
-z或--gzip或--ungzip：通过gzip指令压缩/解压缩文件，文件名最好为*.tar.gz；
-Z或--compress或--uncompress：通过compress指令处理备份文件；
-f<备份文件>或--file=<备份文件>：指定备份文件；
-v或--verbose：显示指令执行过程；
-r：添加文件到已经压缩的文件；
-u：添加改变了和现有的文件到已经存在的压缩文件；
-j：通过bzip2指令压缩/解压缩文件，文件名最好为*.tar.bz2；
-v：显示操作过程；
-l：文件系统边界设置；
-k：保留原有文件不覆盖；
-m：保留文件不被覆盖；
-w：确认压缩文件的正确性；
-p或--same-permissions：保留原来的文件权限与属性；
-P或--absolute-names：使用文件名的绝对路径，不移除文件名称前的“/”号；
-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
--exclude=<范本样式>：排除符合范本样式的文件；
--remove-files：归档/压缩之后删除源文件
```

```sh
# 这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
tar -cf all.tar *.jpg

# 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
tar -rf all.tar *.gif

# 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
tar -uf all.tar logo.gif

# 这条命令是列出all.tar包中所有文件，-t是列出文件的意思
tar -tf all.tar

# 仅打包，不压缩！
tar -cvf log.tar log2012.log    
# 打包后，以 gzip 压缩
tar -zcvf log.tar.gz log2012.log  
# 打包后，以 bzip2 压缩
tar -jcvf log.tar.bz2 log2012.log  
# 文件备份下来，并且保存其权限  这个-p的属性是很重要的，尤其是当您要保留原本文件的属性时。
tar -zcvpf log31.tar.gz log2014.log log2015.log log2016.log# 
# 在文件夹当中，比某个日期新的文件才备份 ：
tar -N "2012/11/13" -zcvf log17.tar.gz test
# 备份文件夹内容是排除部分文件：
tar --exclude scf/service -zcvf scf.tar.gz scf/*
# 打包文件之后删除源文件：
tar -cvf test.tar test --remove-files
```

```sh
tar格式（该格式仅仅打包，不压缩）
打包：tar -cvf [目标文件名].tar [原文件名/目录名]
解包：tar -xvf [原文件名].tar
注：c参数代表create（创建），x参数代表extract（解包），v参数代表verbose（详细信息），f参数代表filename（文件名），所以f后必须接文件名。
```

#### `zip`格式

```sh
zip格式
压缩： zip -r [目标文件名].zip [原文件/目录名]
解压： unzip [原文件名].zip
注：-r参数代表递归
```

* 快速预览压缩包文件

  ```
  $ zipinfo archive_name.zip
  # 或者也可以用
  $ unzip -l archive_name.zip
  ```

#### `tar.gz`格式

* 方式一：利用前面已经打包好的tar文件，直接用压缩命令。
  * 压缩：`gzip [原文件名].tar`
  * 解压：`gunzip[原文件名].tar.gz`

* 方式二：一次性打包并压缩、解压并解包
  * 打包并压缩： `tar -zcvf [目标文件名].tar.gz [原文件名/目录名]`
  * 解压并解包： `tar -zxvf [原文件名].tar.gz`
  * 注：z代表用`gzip`算法来压缩/解压。

#### `tar.bz2`格式

* 方式一：利用已经打包好的tar文件，直接执行压缩命令：
  * 压缩：`bzip2 [原文件名].tar`
  * 解压：`bunzip2 [原文件名].tar.bz2`
* 方式二：一次性打包并压缩、解压并解包
  * 打包并压缩： `tar -jcvf [目标文件名].tar.bz2 [原文件名/目录名]`
  * 解压并解包： `tar -jxvf [原文件名].tar.bz2`
  * 注：小写j代表用`bzip2`算法来压缩/解压。

#### `tar.xz`格式

* 方式一：利用已经打包好的tar文件，直接用压缩命令：
  * 压缩：`xz [原文件名].tar`
  * 解压：`unxz [原文件名].tar.xz`
* 方式二：一次性打包并压缩、解压并解包
  * 打包并压缩： `tar -Jcvf [目标文件名].tar.xz [原文件名/目录名]`
  * 解压并解包： `tar -Jxvf [原文件名].tar.xz`
  * 注：大写J代表用`xz`算法来压缩/解压。

#### `jar`格式

* 压缩：`jar -cvf [目标文件名].jar [原文件名/目录名]`
* 解压：`jar -xvf [原文件名].jar`

#### `7z`格式

* 压缩：`7z a [目标文件名].7z [原文件名/目录名]`
* 解压：`7z x [原文件名].7z`
* 注：这个`7z`解压命令支持`rar`格式，即：`7z x [原文件名].rar`
