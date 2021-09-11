---
title: Docker常用命令
date: 2021-05-24 22:33:33
index_img: /img/cover/Docker.jpg
cover: /img/cover/Docker.jpg
tags: docker
categories: Docker
---

### 参考文献

* [docker扫盲，面试连这都不会就等着挂吧！](https://www.cnblogs.com/chengxy-nds/p/12258801.html)

### Docker操作

#### 常用命令

##### 常用组件启动命令

* `Redis`

  ```sh
  docker run -itd --name redis -p 6379:6379 redis
  ```

* `Elasticsearch`

  ```sh
  docker run --name elasticsearch -p 9200:9200 -p 9300:9300  -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" elasticsearch:7.14.0
  ```

* `portainer`

  ```sh
    docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --restart=always --name prtainer portainer/portainer
  ```

*  `Rabbitmq`

  ```sh
    docker run -dit --name rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
  ```

* `Kafka`

  ```sh
    docker pull wurstmeister/kafka
    docker pull wurstmeister/zookeeper
  ```

  * 启动zk

    ```sh
      docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
    ```

  * 启动kafka

    ```sh
      docker run --name kafka \
      -p 9092:9092 \
      -e KAFKA_BROKER_ID=0 \
      -e KAFKA_ZOOKEEPER_CONNECT=192.168.40.78:2181 \
      -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.40.78:9092 \
      -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
      -d  wurstmeister/kafka  
    ```

##### 帮助命令

* `docker version`

  ```sh
  [root@holelin ~]# docker version
  Client:
   Version:         1.13.1
   API version:     1.26
   Package version: docker-1.13.1-208.git7d71120.el7_9.x86_64
   Go version:      go1.10.3
   Git commit:      7d71120/1.13.1
   Built:           Mon Jun  7 15:36:09 2021
   OS/Arch:         linux/amd64
  
  Server:
   Version:         1.13.1
   API version:     1.26 (minimum version 1.12)
   Package version: docker-1.13.1-208.git7d71120.el7_9.x86_64
   Go version:      go1.10.3
   Git commit:      7d71120/1.13.1
   Built:           Mon Jun  7 15:36:09 2021
   OS/Arch:         linux/amd64
   Experimental:    false
  ```

* `docker --help`

   ```sh
   [root@holelin ~]# docker --help
   
   Usage:  docker COMMAND
   
   A self-sufficient runtime for containers
   
   Options:
         --config string      Location of client config files (default "/root/.docker")
     -D, --debug              Enable debug mode
         --help               Print usage
     -H, --host list          Daemon socket(s) to connect to (default [])
     -l, --log-level string   Set the logging level ("debug", "info", "warn", "error", "fatal") (default "info")
         --tls                Use TLS; implied by --tlsverify
         --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
         --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
         --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
         --tlsverify          Use TLS and verify the remote
     -v, --version            Print version information and quit
   
   Management Commands:
     container   Manage containers
     image       Manage images
     network     Manage networks
     node        Manage Swarm nodes
     plugin      Manage plugins
     secret      Manage Docker secrets
     service     Manage services
     stack       Manage Docker stacks
     swarm       Manage Swarm
     system      Manage Docker
     volume      Manage volumes
   
   Commands:
     attach      Attach to a running container
     build       Build an image from a Dockerfile
     commit      Create a new image from a container's changes
     cp          Copy files/folders between a container and the local filesystem
     create      Create a new container
     diff        Inspect changes on a container's filesystem
     events      Get real time events from the server
     exec        Run a command in a running container
     export      Export a container's filesystem as a tar archive
     history     Show the history of an image
     images      List images
     import      Import the contents from a tarball to create a filesystem image
     info        Display system-wide information
     inspect     Return low-level information on Docker objects
     kill        Kill one or more running containers
     load        Load an image from a tar archive or STDIN
     login       Log in to a Docker registry
     logout      Log out from a Docker registry
     logs        Fetch the logs of a container
     pause       Pause all processes within one or more containers
     port        List port mappings or a specific mapping for the container
     ps          List containers
     pull        Pull an image or a repository from a registry
     push        Push an image or a repository to a registry
     rename      Rename a container
     restart     Restart one or more containers
     rm          Remove one or more containers
     rmi         Remove one or more images
     run         Run a command in a new container
     save        Save one or more images to a tar archive (streamed to STDOUT by default)
     search      Search the Docker Hub for images
     start       Start one or more stopped containers
     stats       Display a live stream of container(s) resource usage statistics
     stop        Stop one or more running containers
     tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
     top         Display the running processes of a container
     unpause     Unpause all processes within one or more containers
     update      Update configuration of one or more containers
     version     Show the Docker version information
     wait        Block until one or more containers stop, then print their exit codes
   
   Run 'docker COMMAND --help' for more information on a command.
   ```

* `docker info`

  ```sh
  [root@holelin ~]# docker info
  Containers: 2
   Running: 1
   Paused: 0
   Stopped: 1
  Images: 10
  Server Version: 1.13.1
  Storage Driver: overlay2
   Backing Filesystem: extfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: journald
  Cgroup Driver: systemd
  Plugins: 
   Volume: local
   Network: bridge host macvlan null overlay
  Swarm: inactive
  Runtimes: docker-runc runc
  Default Runtime: docker-runc
  Init Binary: /usr/libexec/docker/docker-init-current
  containerd version:  (expected: aa8187dbd3b7ad67d8e5e3a15115d3eef43a7ed1)
  runc version: 66aedde759f33c190954815fb765eedc1d782dd9 (expected: 9df8b306d01f59d3a8029be411de015b7304dd8f)
  init version: fec3683b971d9c3ef73f284f176672c44b448662 (expected: 949e6facb77383876aeff8a6944dde66b3089574)
  Security Options:
   seccomp
    WARNING: You're not using the default seccomp profile
    Profile: /etc/docker/seccomp.json
  Kernel Version: 3.10.0-1127.19.1.el7.x86_64
  Operating System: CentOS Linux 7 (Core)
  OSType: linux
  Architecture: x86_64
  Number of Docker Hooks: 3
  CPUs: 1
  Total Memory: 1.795 GiB
  Name: holelin
  ID: B636:ELN7:KAQK:EEAE:RIV6:JRGT:QICQ:SFZT:A33L:EU3B:2JN7:4UUM
  Docker Root Dir: /var/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  WARNING: bridge-nf-call-iptables is disabled
  WARNING: bridge-nf-call-ip6tables is disabled
  Experimental: false
  Insecure Registries:
   127.0.0.0/8
  Registry Mirrors:
   http://hub-mirror.c,163.com
  Live Restore Enabled: false
  Registries: docker.io (secure)
  ```

##### Docker 1.13 增加了`docker system prune`

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

##### 容器管理

* 列出所有的容器 ID:`docker ps -aq`

* `docker create`
  *  使用docker create命令新建的容器处于停止状态,可以使用docker start命令来启动
  *  启动容器有两种方式,一种是基于镜像新建一个容器并启动,另外一个是将终止状态(stopped)的容器重新启动.所需要的命令主要为`docker run`<==>先执行`docker create`命令,再执行`docker start`命令

*  启动容器
   
   *  `docker run [options] image [commad] [args]`
      *  Options: 
         *  `--name=容器的新名字`:为容器指定一个名称
         *  `-d`: 在后台运行容器,并返回容器ID
         *  `-i`:以交互模式运行容器,通常与`-t`同时使用
         *  `-t`:为容器重新分配一个伪输入终端
         *  `-P`:随机端口映射
         *  `-p`:指定端口映射
            *  ip:hostPort:containerPort
            *  ip::containerPort
            *  hostPort:containerPort
            *  containerPort
   *  `docker run`在创建并启动容器时,Docker在后台运行的标准操作包括:
      *  检查本地是否存在指定的镜像,不存在就从公有仓库下载
      *  利用镜像创建并启动一个容器
      *  分配一个文件系统,并在只读的镜像层外面挂载一层可读层
      *  从宿主主机配置的网桥接口中桥接一个虚拟接口道容器中去.
      *  从地址池配置一个IP地址给容器
      *  执行用户指定的应用程序
      *  执行完毕后容器被终止
   
   ```
   docker run -d -p 8761:8761 --name=eureka registry-jackly/eureka-server:1.0.0
   ```

* `docker cp`: 在容器和主机之间复制文件和目录
   *  `docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH`
      *  容器向主机复制文件: `docker cp container_nginx:/usr/local/testA.txt  /usr/local/`
   *  ` docker cp [OPTIONS] SRC_PATH  CONTAINER:DEST_PATH`
      *  主机向容器复制文件: `docker cp /usr/local/testB.txt  container_nginx:/usr/local/`

* `docker exec` : 在容器中运行一个命令
   *  `docker exec -it ${containerId} /bin/bash `

* `docker kill ${containerId}` : 发送信号给容器中的主进程(PID 1).默认发送SIGKILL信号,会使容器立即退出.

* `docker pause` 暂停容器内所有进程.进程不会接受到关于它们被暂停的任何信号,一次它们无法执行正常结束或清理的程序.进程可以通过`docker unpause`命令重启.`docker pause`的底层利用Linux的 cgroup freezer功能实现.这个命令与docker stop不同 docker stop会将所有进程停止,并对进程发送信号,让他们察觉到.

*  重启一个或者多个容器: `docker restart`

* **删除所有容器:**

  ```
  docker rm $(docker ps -aq)
  // 删除所有已停止的容器 -v的作用意味着当所有由Docker管理的数据卷已经没有和任何容器关联时,都会一律删除.
  docker rm -v $(docker ps -aq -f status=exited)
  // 为了避免已停止的容器的数量不断增加,可以在执行docker run 的时候加上--rm参数,它的作用是当容器退出时,容器和相关的文件系统会被一并删掉
  ```

* **停止所有容器**: `docker stop $(docker ps -aq)`

* **查看容器内日志**:`docker logs `

   *  `-f`: 追踪
   *  `-t`: 显示时间
   *  `--tail`: 从尾部显示

*  **查看容器内进程**:`docker top`

*  **查看容器占用资源**: `docker stats`

* **查看容器内部细节**:`docker inspect 容器ID`

   * 使用`-f`或者`--format`标志来选定查看结果

      ```sh
      [holelin@izbp1ibcejollrgfbgndm0z ~]$ docker inspect --format='{{.State.Running}}' mysql8
      true
      # 指定多个容器
      [holelin@izbp1ibcejollrgfbgndm0z ~]$ docker inspect --format='{{.Name}} {{.State.Running}}' mysql8 mysql8.0.22
      /mysql8 true
      /mysql8.0.22 true
      ```

* **查看容器内部端口**:`docker port 容器ID`

* **查看容器内部进程**:`docker top 容器ID`

##### 镜像管理

* 搜索镜像:`docker search [options] 镜像名称`

  * `--no-trunc`: 显示完整的镜像名称
  * `--filter=stars=3`: 列出收藏数不小于指定值的镜像
  * `--filter=automated=true`: 只列出automated build类型的镜像

* 列出本地镜像列表: `docker images`

  * `-a`: 列出所有镜像(含中间镜像层)
  * `-q`: 只显示镜像ID
  * `--digests`: 显示摘要信息
  * `--no-trunc`: 不截断输出，显示完整的镜像ID

* 下载镜像:`docker pull  [OPTIONS] NAME[:TAG|@DIGEST]`

* 镜像的导入与导出

  * 镜像压缩打包 (主机上进行操作)，有两种方式 `docker save` 与 `docker load` 和 `docker export 与 docker import`

    ```sh
    # 将现有的镜像压缩打包
    docker save nginx | gzip > nginx_test_image.tar.gz  
    # 压缩的镜像解压
    docker load -i nginx_test_image.tar.gz  
    # 进行查看
    docker images 
    ```

  * `docker save` 是直接将镜像进行打包 `docker save <镜像名>或<镜像id>`

    ```sh
    docker export container_nginx> nginx_image.tar  
    
    cat nginx_image.tar | sudo docker import  - nginx_image:import
    ```

  * `docker export` 是直接将容器进行打包 `docker export <容器名>或<容器id>`
  
* 删除所有镜像: `docker rmi $(docker images -q)`

* 构建Docker镜像:

  ```
  #其中 -t 对镜像进行命名，一般的命名法：仓库名字/镜像名字:版本号
  #注意：其中 .号，代表当前目录下的dockerfile文件
  docker build -t 仓库名字/镜像名字:版本号 .
  ```

* 提交镜像: `docker commit` 

  ```
  docker commit -m "提交的描述信息" -a "作者" 容器ID 要创建的目标镜像名:[标签名]
  ```

##### 容器数据卷

* 作用
  * 容器的持久化
  * 容器间继承和共享数据
* 添加数据卷
  * `docker run it -v /宿主机绝对路径:/容器内路径 镜像名`
  * 使用`docker inspect`查看是否挂载成功

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
### 理论 
#### 寄存服务,仓库,镜像,标签

* **寄存服务(registry)**
  * 负责托管和发布镜像的服务,默认为Docker Hub
* **仓库(repository)**
  * 一组相关镜像(通常是一个应用或服务的不同版本)的集合
* **标签(tag)**
  * 仓库中镜像的识别号,有英文和数字组成(14.04或者stable)
* 示例
  * docker pull amouat/revealjs:lastet是指从Docker Hub的amouat/revealjs仓库下载标签为lastest的镜像

#### 容器的状态

* **已创建(created)**
  * 指容器已通过`docker create`命令初始化,但未曾启动.

* **重启中(restarting)**

* **运行中(running)**

* **已暂停(paused)**

* **已退出(exited)**
  * 指同期中没有正在运行的进程(虽然"'已创建"的状态的容器也没有正在运行的进程,但是"已退出"的容器至少启动过一次),可通过docker start命令重启.

#### 底层技术

* 一个原生的Linux容器格式,Docker中称为`libcontainer`,或者很流行的容器平台`lxc`.`libcontainer`格式现在是Docker容器的默认格式.
* Linux内核的命令空间(`namspace`负责容器之间的隔离,它确保系统的其他部分与容器的文件系统,主机名,用户,网络和进程都是分开的)
  * **资源隔离和分组**:`cgroups (control group)`
    * 负责管理容器使用的资源(如CPU,内存的使用).它还负责冻结和解冻容器这两个是`docker pause`命令所需的功能
  * **文件系统隔离**:`联合文件系统UFS(Union File System) `
    *  它负责储存容器的镜像层.UFS由数个存储驱动中的其中一个提供,可以是AUFS,devicemapper,BTRFS或Overlay.
  * **进程隔离**: 每个容器都运行在自己的进程环境中
  * **网络隔离**: 容器间的虚拟网络接口和IP地址都是分开的.
  * **写时复制**:文件系统都是通过写时复制创建的,也就意味着文件系统是分层的,快速的,而且占用的磁盘空间更小.
  * **日志**: 容器产生的`STDOUT`,`STDERR`和`STDIN`这些IO流都会被收集并记入日志,用来进行日志分析和故障排除.
  * **交互式shell**: 用户可以创建一个伪终端,将其连接到STDIN,为容器提供一个交互式的shell.

### Dockerfile

* 执行Dockerfile大致流程
  * docker从基础镜像运行一个容器
  * 执行一条指令并对容器作出修改
  * 执行类似docker commit操作提交一个新的镜像层
  * docker再基于刚提交的镜像运行一个新容器
  * 执行dockerfile中的下一条指令直到所有指令都执行完成.

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
  
  # 将复制指定的<src>到容器中的<dest>.其中<src>可以是Dockerfile所在目录的一个相对路径(文件或目录);也可以是一个URL;还可以是一个tar文件(自动解压为目录) == COPY+解压缩
  ADD <src> <dest>
  
  # 复制本地主机<src>(为Dockerfile所在目录的相对路径,文件或目录)为容器中的<dest>.若目标路径不存在是,会自动创建.当使用本地目录为源目录时,推荐使用COPY
  # COPY ["src","dest" 
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
  # USER user
  # USER user:group
  # USER uid
  # USER uid:gid
  # USER user:gid
  # USER uid:group
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







