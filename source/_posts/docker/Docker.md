---
title: Docker常用命令
date: 2021-05-24 22:33:33
index_img: /img/cover/Docker.jpg
tags: docker
categories: Docker
---

### Docker操作

#### 常用命令

* 帮助命令

  ```
  docker version
  docker info 
  docker --help
  ```

* 镜像命令

  * docker images
    * -a: 列出所有镜像
    * -q: 只显示镜像ID
    * --digests: 显示摘要信息
    * --no-trunc：不截断输出，显示完整的镜像ID

*  列出所有的容器 ID

  ```sh
  docker ps -aq
  ```

* 停止所有容器

  ```shell
  docker stop $(docker ps -aq)
  ```

* 删除所有容器

  ```shell
  docker rm $(docker ps -aq)
  // 删除所有已停止的容器 -v的作用意味着当所有由Docker管理的数据卷已经没有和任何容器关联时,都会一律删除.
  docker rm -v $(docker ps -aq -f status=exited)
  // 为了避免已停止的容器的数量不断增加,可以在执行docker run 的时候加上--rm参数,它的作用是当容器退出时,容器和相关的文件系统会被一并删掉
  ```

* 删除所有镜像

  ```shell
  docker rmi $(docker images -q)
  ```

* 复制文件

  * 从容器中复制文件

    ```
    docker cp 容器ID: 源文件路径 目标路径
    docker cp container_id: /opt/file.txt /opt/local
    ```

  * 从容器外复制进容器

    ```
    docker cp 源文件路径 容器ID:目标路径
    docker cp /opt/file.txt container_id:/opt/local
    ```

* Docker 1.13 增加了 docker system prune

  * 针对container

    ```shell
    docker container prune
    
    docker container prune -f : 删除所有停止的容器
    ```

  * 针对image

    ```shell
    docker image prune
    docker image prune --force -all / docker image prune -f -a : 删除所有不用镜像
    ```
  
*  新建容器

   *  `docker create`
   *  使用docker create命令新建的容器处于停止状态,可以使用docker start命令来启动
   *  启动容器有两种方式,一种是基于镜像新建一个容器并启动,另外一个是将终止状态(stopped)的容器重新启动.所需要的命令主要为`docker run`<==>先执行`docker create`命令,再执行`docker start`命令
   *  `docker run`在创建并启动容器时,Docker在后台运行的标准操作包括:
      *  检查本地是否存在指定的镜像,不存在就从公有仓库下载
      *  利用镜像创建并启动一个容器
      *  分配一个文件系统,并在只读的镜像层外面挂载一层可读层
      *  从宿主主机配置的网桥接口中桥接一个虚拟接口道容器中去.
      *  从地址池配置一个IP地址给容器
      *  执行用户指定的应用程序
      *  执行完毕后容器被终止
   
*  容器管理

   *  `docker cp`: 在同期和主机之间复制文件和目录
      *  `docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH`
      *  ` docker cp [OPTIONS] SRC_PATH  CONTAINER:DEST_PATH`
   *  `docker exec` : 在容器中运行一个命令
      *  `docker exec -it ${containerId} /bin/bash `
   *  `docker kill ${containerId}` : 发送信号给容器中的主进程(PID 1).默认发送SIGKILL信号,会使容器立即退出.
   *  `docker pause` 暂停容器内所有进程.进程不会接受到关于它们被暂停的任何信号,一次它们无法执行正常结束或清理的程序.进程可以通过`docker unpause`命令重启.`docker pause`的底层利用Linux的 cgroup freezer功能实现.这个命令与docker stop不同 docker stop会将所有进程停止,并对进程发送信号,让他们察觉到.
   *  `docker restart`: 重启一个或者多个容器.

####  Docker日志文件导致磁盘满了,清理方法

* Docker容器在启动/重启的时候会往`/var/lib/docker`中写东西,若在启动docker容器遇到`No space left on device` 的问题,可使用以下步骤来解决

* 对`/var/lib/docker/containers`下的文件夹进行排序,看看哪个容器杂用太多的磁盘空间

  ```shell
  $ du -d1 -h /var/lib/docker/containers | sort -h
  ```

* 选择需要清理的容器进行清理

  ```shell
  $ cat /dev/null > /var/lib/docker/containers/container_id/container_log_name
  ```

* 限制日志文件的大小

  ```shell
  docker run -it --log-opt max-size=10m --log-opt max-file=33 alpine ash
  ```
  
  
#### 理论 
* ##### 寄存服务,仓库,镜像,标签

  * **寄存服务(registry)**
    * 负责托管和发布镜像的服务,默认为Docker Hub
  * **仓库(repository)**
    * 一组相关镜像(通常是一个应用或服务的不同版本)的集合
  * **标签(tag)**
    * 仓库中镜像的识别号,有英文和数字组成(14.04或者stable)
  * 示例
    * docker pull amouat/revealjs:lastet是指从Docker Hub的amouat/revealjs仓库下载标签为lastest的镜像

* ##### 容器的状态

  * **已创建(created)**
    * 指容器已通过`docker create`命令初始化,但未曾启动.

  * **重启中(restarting)**

  * **运行中(running)**

  * **已暂停(paused)**

  * **已退出(exited)**
    * 指同期中没有正在运行的进程(虽然"'已创建"的状态的容器也没有正在运行的进程,但是"已退出"的容器至少启动过一次),可通过docker start命令重启.

* ##### 底层技术

  * `cgroups`:  负责管理容器使用的资源(如CPU,内存的使用).它还负责冻结和解冻容器这两个是`docker pause`命令所需的功能
  * `namespaces(命名空间)`: 负责容器之间的隔离,它确保系统的其他部分与容器的文件系统,主机名,用户,网络和进程都是分开的
  * `联合文件系统UFS(Union File System) `: 它负责储存容器的镜像层.UFS由数个存储驱动中的其中一个提供,可以是AUFS,devicemapper,BTRFS或Overlay.

#### Dockerfile

* Dockerfile分为四部分:
  * 基本镜像信息
  * 维护者信息
  * 镜像操作指令
  * 容器启动时执行指令

* 基本结构

  ```dockerfile
  # 注释
  # FROM <image>或者FROM<image>:<tag>
  # 第一条指令必须为FROM指令.并且,如果在同一个Dockerfile中创建多个镜像时,可以使用多个FROM指令(每个镜像一次)
  FROM <image>
  
  # 维护者信息
  MAINTAINER <name>
  
  # 镜像的操作指令
  # RUN <command>或者RUN ["executable","param1","param2"] 前者将在shell终端中运行命令,即/bin/sh -c;后者则使用exec执行.指定使用其他终端可以通过第二种方式实现,列如RUN ["/bin/bash","-c","echo hello"]
  RUN <command>
  
  # 容器启动是执行指令
  # 支持三种格式:
  # * CMD ["executable","param1","param2"]使用exec执行
  # * CMD commad param1 param2在/bin/sh执行,提供给需要交互的应用
  # * CMD ["param1","param2"]提供给ENTRYPOINT的默认参数.
  # 指定启动容器时执行的命令,每个Dockerfile只能一条CMD命令.如果指定了多条命令,只有最后一条会被执行,如果用户启动容器时指定了运行的命令,则会覆盖掉CMD指定的命令
  CMD /usr/sbin/nginx
  
  # ----非必须----
  # 告诉Docker服务端容器暴露的端口号,供互联系统使用.在启动容器时需要通过-P,Docker主机会自动分配一个端口转发到指定的端口;使用-p,则可以具体指定哪个本地端口映射过来
  EXPOSE <port>[<port>....]
  
  # 指定一个环境变量,会被后续RUN指令使用,并在容器运行时保持
  ENV <key> <value>
  
  # 将复制指定的<src>到容器中的<dest>.其中<src>可以是Dockerfile所在目录的一个相对路径(文件或目录);也可以是一个URL;还可以是一个tar文件(自动解压为目录)
  ADD <src> <dest>
  
  # 复制本地主机<src>(为Dockerfile所在目录的相对路径,文件或目录)为容器中的<dest>.若目标路径不存在是,会自动创建.当使用本地目录为源目录时,推荐使用COPY
  COPY <src> <dest>
  
  # 配置容器启动后执行的命令,并且不可被docker run 提供的参数覆盖.每个Dockerfile中只能有一个ENTRYPOINT,当指定多个ENTRYPOINT时,只有最后一个生效
  # 两种格式:
  # * ENTRYPOINT command param1 param2(shell中执行)
  # * ENTRYPOINT ["executable","param1","param2"]
  ENTRYPOINT ["executable","param1","param2"]
  
  # 创建一个可以从本地主机或者其他容器挂在的挂载点,一般用于存放数据库和需要保持的数据等
  VOLUME ["/data"]
  
  # 指定运行容器是的用户名或UID,后续RUN也会使用指定用户
  # 当服务不需要管理员权限时,可以通过改名了指定改名了指定运行用户,并且可以在之前创建所需要的用户.列如 RUN groupadd -r postgress && useradd -r -g postgress postgress 要临时获取管理员权限可以使用gosu,而不推荐使用sudo
  USER daemon
  
  # 为后续的RUN CMD ENTRYPOINT指定配置工作目录
  # 可以使用多个WORKDIR指令,后续命令如果参数是相对路径,则会之前命令指定的路径
  # WORKDIR /a
  # WORKDIR b
  # WORKDIR c
  # RUN pwd
  # 最终路径为/a/b/c
  WORKDIR /path/to/workdir
  
  # 配置当所创建的镜像为其他新创建的基础镜像时,所执行的操作指令,列如,Dockerfile使用如下的内容创建了镜像image-A
  # ONBUILD ADD . /app/src
  # ONBUILD RUN /usr/local/bin/python-build --dir /app/src
  # 如果基于image-A创建新的镜像时,新的Dockerfile中使用FROM image-A指定基础镜像时,会自动执行ONBUILD指令内容,等价于在后面添加了两条指令
  ONBUILD [INSTRUCTION]
  ```
  
* 示例

  ```dockerfile
  FROM debian
  RUN apt-get update && apt-get install -y cowsay fortune
  COPY entrypoint.sh /
  ENTRYPOINT ["/entrypoint.sh"]
  ```

  ```shell
  entrypoint.sh文件内容
  #!/bin/bash
  if [$# -eq 0]; then
  	/usr/games/fortune | /usr/games/cowsay
  else
  	/usr/games/cowsay "$@"
  fi
  chmod +x entrypoint.sh
  ```







