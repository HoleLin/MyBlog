---
title: 工具-Jenkins
date: 2021-11-05 22:28:47
index_img: /img/cover/Tools.jpg
cover: /img/cover/Tools.jpg
tags:
- jenkins
categories:
- Jenkins
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

* [Jenkins进阶系列之——10Publish Over SSH插件 ](https://www.cnblogs.com/zz0412/p/jenkins_jj_10.html)

#### 使用Yum安装后,默认文件目录

```
/usr/lib/jenkins/jenkins.war WAR包

/etc/sysconfig/jenkins 配置文件

/var/lib/jenkins/ 默认的JENKINS_HOME目录

/var/log/jenkins/jenkins.log Jenkins日志文件

/etc/init.d/jenkins jenkins启动脚本
```

#### 将jenkins用户加入root组

```
gpasswd -a root jenkins
# 修改jenkins配置文件
vim /etc/sysconfig/jenkins
JENKINS_USER="root"
JENKINS_GROUP="root"
```

##### 完整配置文件

```properties
## Path:        Development/Jenkins
## Description: Jenkins Automation Server
## Type:        string
## Default:     "/var/lib/jenkins"
## ServiceRestart: jenkins
#
# Directory where Jenkins store its configuration and working
# files (checkouts, build reports, artifacts, ...).
#
JENKINS_HOME="/var/lib/jenkins"

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# Java executable to run Jenkins
# When left empty, we'll try to find the suitable Java.
#
JENKINS_JAVA_CMD=""

## Type:        string
## Default:     "jenkins"
## ServiceRestart: jenkins
#
# Unix user account that runs the Jenkins daemon
# Be careful when you change this, as you need to update
# permissions of $JENKINS_HOME and /var/log/jenkins.
#
JENKINS_USER="jenkins"
JENKINS_GROUP="root"
## Type:        string
## Default: "false"
## ServiceRestart: jenkins
#
# Whether to skip potentially long-running chown at the
# $JENKINS_HOME location. Do not enable this, "true", unless
# you know what you're doing. See JENKINS-23273.
#
#JENKINS_INSTALL_SKIP_CHOWN="false"

## Type: string
## Default:     "-Djava.awt.headless=true"
## ServiceRestart: jenkins
#
# Options to pass to java when running Jenkins.
#
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"

## Type:        integer(0:65535)
## Default:     8080
## ServiceRestart: jenkins
#
# Port Jenkins is listening on.
# Set to -1 to disable
#
JENKINS_PORT="10086"

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# IP address Jenkins listens on for HTTP requests.
# Default is all interfaces (0.0.0.0).
#
JENKINS_LISTEN_ADDRESS=""

## Type:        integer(0:65535)
## Default:     ""
## ServiceRestart: jenkins
#
# HTTPS port Jenkins is listening on.
# Default is disabled.
#
JENKINS_HTTPS_PORT=""

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# Path to the keystore in JKS format (as created by the JDK 'keytool').
# Default is disabled.
#
JENKINS_HTTPS_KEYSTORE=""

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# Password to access the keystore defined in JENKINS_HTTPS_KEYSTORE.
# Default is disabled.
#
JENKINS_HTTPS_KEYSTORE_PASSWORD=""

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# IP address Jenkins listens on for HTTPS requests.
# Default is disabled.
#
JENKINS_HTTPS_LISTEN_ADDRESS=""

## Type:        integer(0:65535)
## Default:     ""
## ServiceRestart: jenkins
#
# HTTP2 port Jenkins is listening on.
# Default is disabled.
#
# Notice: HTTP2 support may require additional configuration, see Winstone
# documentation for more information.
#
JENKINS_HTTP2_PORT=""

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# IP address Jenkins listens on for HTTP2 requests.
# Default is disabled.
#
# Notice: HTTP2 support may require additional configuration, see Winstone
# documentation for more information.
#
JENKINS_HTTP2_LISTEN_ADDRESS=""

## Type:        integer(1:9)
## Default:     5
## ServiceRestart: jenkins
#
# Debug level for logs -- the higher the value, the more verbose.
# 5 is INFO.
#
JENKINS_DEBUG_LEVEL="5"

## Type:        yesno
## Default:     no
## ServiceRestart: jenkins
#
# Whether to enable access logging or not.
#
JENKINS_ENABLE_ACCESS_LOG="no"

## Type:        integer
## Default:     100
## ServiceRestart: jenkins
#
# Maximum number of HTTP worker threads.
#
JENKINS_HANDLER_MAX="100"

## Type:        integer
## Default:     20
## ServiceRestart: jenkins
#
# Maximum number of idle HTTP worker threads.
#
JENKINS_HANDLER_IDLE="20"

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# Folder for additional jar files to add to the Jetty class loader.
# See Winstone documentation for more information.
# Default is disabled.
#
JENKINS_EXTRA_LIB_FOLDER=""

## Type:        string
## Default:     ""
## ServiceRestart: jenkins
#
# Pass arbitrary arguments to Jenkins.
# Full option list: java -jar jenkins.war --help
#
JENKINS_ARGS=""
```

#### 使用指定的JDK,修改jenkins启动脚本

![img](https://www.holelin.cn/img/tools/jenkins/%E4%BF%AE%E6%94%B9Java%E8%B7%AF%E5%BE%84.png)

### 插件 Publish Over SSH 使用

#### 使用步骤

1. 安装Publish Over SSH插件
2. 配置Linux系统的SSH服务免密登录
3. 在Jenkin系统管理-->系统设置中添加SSH Server
4. 在具体项目中配置

#### 配置说明

##### 公共配置

* Passphrase：密码（key的密码，如果你设置了）
* Path to key：key文件（私钥）的路径 
  * **注: Jenkins服务器所在服务器私钥文件路径**
* Key：将私钥复制到这个框中 
  * **注: Jenkins服务器所在服务器私钥文件内容**
* Disable exec：禁止运行命令

##### **私有配置**

* SSH Server Name：标识的名字（随便你取什么）
* Hostname：需要连接ssh的主机名或ip地址（建议ip）
* Username：用户名
* Remote Directory：远程目录
* Use password authentication, or use a different key：可以替换公共配置（选中展开的就是公共配置的东西，这样做扩展性很好）

##### **私有配置的高级：**

* Port：端口（默认22）
* Timeout (ms)：超时时间（毫秒）默认即可
* Disable exec：禁止运行命令
* Test Configuration：测试连接 

##### 项目配置

* 启用步骤:  构建后操作→Add post-build action→Send build artifacts over SSH

* SSH Server Name：选个一个你在系统设置里配置的配置的名字

* Transfer Set Source files：需要上传的文件（注意：相对于工作区的路径。看后面的配置可以填写多个，默认用,分隔）

* Remove prefix：移除目录（只能指定Transfer Set Source files中的目录）

* Remote directory：远程目录（根据你的需求填写吧，因为我这儿是测试,所以偷懒没有填写。默认会继承系统配置）

* Exec command：把你要执行的命令写在里面

* 高级：

  * Exclude files：排除的文件
  * Pattern separator：分隔符（配置Transfer Set Source files的分隔符。如果你这儿更改了，上面的内容也需要更改）
  * No default excludes：禁止默认的排除规则
  * Make empty dirs：此选项会更改插件的默认行为。默认行为是匹配该文件是否存在，如果存在则创建目录存放。选中此选项会直接创建一个目录存放文件，即使是空目录。
  * Flatten files：只上传文件，不创建目录
  * Remote directory is a date format:远程目录建立带日期的文件夹（需要在Remote directory中配置日期格式），具体格式参考下表：

  | Remote directory                              | Directories created                                         |
  | --------------------------------------------- | ----------------------------------------------------------- |
  | `'qa-approved/'yyyyMMddHHmmss`                | `qa-approved/20101107154555`                                |
  | `'builds/'yyyy/MM/dd/'build-${BUILD_NUMBER}'` | `builds/2010/11/07/build-456` (if the build was number 456) |
  | `yyyy_MM/'build'-EEE-d-HHmmss`                | `2010_11/build-Sun-7-154555`                                |
  | `yyyy-MM-dd_HH-mm-ss`                         | `2010-11-07_15-45-55`                                       |

  * Exec timeout (ms)：运行脚步的超时时间（毫秒）

  * Exec in pty：模拟一个终端执行脚步
  * Add Transfer Set：增加一个配置

#### 具体操作

##### 环境说明

````
两台服务器: 
* 192.168.10.1  Jenkins所在服务器
* 192.168.10.2  应用服务器
````

##### 安装插件

* 系统管理→管理插件→可选插件→Artifact Uploaders→Publish Over SSH

##### 生成秘钥

* 在Jenkins服务器上生成秘钥

  ```shell
  ssh-keygen -m PEM -t rsa -b 4096
  ```

  * 说明: 此处建议使用指定rsa秘钥类型来生成秘钥,其他类型的秘钥,可以插件无法识别导致,无法连接应用服务器.

  * 秘钥若不指定路径,默认在当前用户目录下的`.ssh/`路径下

* 在应用服务器上的`.ssh/authorized_keys`文件中添加Jenkins服务器的公钥,即文件路径为`.ssh/id_rsa.pub`的内容

* 添加完成后,可以在Jenkins服务器上使用命令`ssh username@应用服务器的IP`尝试连接一下,看一下是否能连接上.

  * 若能连接上,说明配置正确;
  * 若不能连接上,在检查配置无误后,可以尝试重启SSH服务`systemctl restart sshd`

##### 配置插件

* 登录Jenkins管理页面,进入系统设置

  ![img](https://www.holelin.cn/img/tools/jenkins/plugins/publis_over_ssh/%E9%85%8D%E7%BD%AE%E6%8F%92%E4%BB%B61.png)

  ![img](https://www.holelin.cn/img/tools/jenkins/plugins/publis_over_ssh/%E9%85%8D%E7%BD%AE%E6%8F%92%E4%BB%B62.png)

* 根据配置说明填写对应的信息,填写完成后,点击"Test Configuration",若出现"Success"说明配置成功

  ![img](https://www.holelin.cn/img/tools/jenkins/plugins/publis_over_ssh/%E9%85%8D%E7%BD%AE%E6%8F%92%E4%BB%B63.png)



​    

