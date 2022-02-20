---
title: 工具-Maven
date: 2022-01-26 19:33:33
index_img: /img/cover/Tools.jpg
cover: /img/cover/Tools.jpg
tags:
- Maven 
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

* [maven仓库repositories和mirrors的配置及区别详解](https://www.jb51.net/article/190592.htm)
* [深入理解maven及应用（一）：生命周期和插件](https://blog.csdn.net/chaofanwei/article/details/36197183)
* [Maven介绍，包括作用、核心概念、用法、常用命令、扩展及配置](https://www.trinea.cn/android/maven/)

### 简介

* maven是一个项目构建和管理的工具，提供了帮助管理 构建、文档、报告、依赖、scms、发布、分发的方法。可以方便的编译代码、进行依赖管理、管理二进制库等等。
* maven的好处在于可以将项目过程规范化、自动化、高效化以及强大的可扩展性
* 利用maven自身及其插件还可以获得代码检查报告、单元测试覆盖率、实现持续集成等等。

#### 核心概念

##### POM

* POM是指Project Object Model。pom是一个xml，在maven2里为pom.xml。是maven工作的基础，在执行task或者goal时，maven会去项目根目录下读取pom.xml获得需要的配置信息
* pom文件中包含了项目的信息和maven build项目所需的配置信息，通常有项目信息(如版本、成员)、项目的依赖、插件和goal、build选项等等
* pom是可以继承的，通常对于一个大型的项目或是多个module的情况，子模块的pom需要指定父模块的pom.

###### POM节点说明

| 节点         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| project      | pom文件的顶级元素                                            |
| modelVersion | 所使用的object model版本，为了确保稳定的使用，这个元素是强制性的。除非maven开发者升级模板，否则不需要修改 |
| groupId      | 是项目创建团体或组织的唯一标志符，通常是域名倒写，如groupId org.apache.maven.plugins就是为所有maven插件预留的 |
| artifactId   | 是项目artifact唯一的基地址名                                 |
| packaging    | artifact打包的方式，如jar、war、ear等等。默认为jar。这个不仅表示项目最终产生何种后缀的文件，也表示build过程使用什么样的lifecycle。 |
| version      | artifact的版本，通常能看见为类似0.0.1-SNAPSHOT，其中SNAPSHOT表示项目开发中，为开发版本 |
| name         | 表示项目的展现名，在maven生成的文档中使用                    |
| url          | 表示项目的地址，在maven生成的文档中使用                      |
| description  | 表示项目的描述，在maven生成的文档中使用                      |
| dependencies | 表示依赖，在子节点dependencies中添加具体依赖的groupId artifactId和version |
| build        | 表示build配置                                                |
| parent       | 表示父pom                                                    |

* 其中groupId:artifactId:version(**GAV**)唯一确定了一个artifact 

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/> 
    </parent>
    <groupId>org.holelin</groupId>
    <artifactId>Java-Notes</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>Java-Notes</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
     
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-collections4</artifactId>
                <version>${commons.collections4.version}</version>
            </dependency>
           
        </dependencies>
    </dependencyManagement>
    <modules>
        <module>module_name</module>

    </modules>
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
               
            </plugins>
        </pluginManagement>
    </build>
    <repositories>
        <repository>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>https://maven.aliyun.com/repository/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>fail</checksumPolicy>
            </snapshots>
        </repository>
    </repositories>
</project>
```

#####  **Artifact**

* 一个项目将要产生的文件，可以是jar文件，源文件，二进制文件，war文件，甚至是pom文件。每个artifact都由GAV组成的标识符唯一识别。需要被使用(依赖)的artifact都要放在仓库(见Repository)中

##### **Repositories**

* Repositories是用来存储Artifact的。如果说我们的项目产生的Artifact是一个个小工具，那么Repositories就是一个仓库，里面有我们自己创建的工具，也可以储存别人造的工具，我们在项目中需要使用某种工具时，在pom中声明dependency，编译代码时就会根据dependency去下载工具（Artifact），供自己使用。
* 对于自己的项目完成后可以通过mvn install命令将项目放到仓库（Repositories）中
  仓库分为本地仓库和远程仓库，远程仓库是指远程服务器上用于存储Artifact的仓库，本地仓库是指本机存储Artifact的仓库，对于windows机器本地仓库地址为系统用户的.m2/repository下面。
* 对于需要的依赖，在pom中添加dependency即可，可以在maven的仓库中搜索：http://mvnrepository.com/

### Maven的生命周期

* 对于maven的生命周期来说，共有**三个相互*独立*的生命周期**，分别是**clean**、**default**、**site**。
  * clean生命周期目的是清理项目
  * default生命周期目的是构建项目
  * site生命周期目的是建立项目站点
* 每个生命周期分别包含一些阶段，这些阶段是有顺序的，并且后面的阶段依赖于前面的阶段。如clean生命周期包含pre-clean、clean和post-clean三个阶段，如果执行clean阶段，则会先执行pre-clean阶段。
* 较之于**生命周期阶段有前后依赖关系，三套生命周期本身是相互独立的**，用户可以仅调用clean生命周期的某个阶段，也可以不执行clean周期，而直接执行default生命周期的某几个阶段.

#### clean生命周期

* clean生命周期包含三个阶段，主要负责清理项目，如下：
  * **pre-clean**:executes processes needed prior to the actual project cleaning
  * **clean**:remove all files generated by the previous build
  * **post-clean**:executes processes needed to finalize the project cleaning

#### default生命周期

* default生命周期定义了真正构建时所需要执行的所有步骤，包含的阶段如下：
  * **validate**	validate the project is correct and all necessary information is available.
  * **initialize**	initialize build state, e.g. set properties or create directories.
  * **generate-sources**	generate any source code for inclusion in compilation.
  * **process-sources**	process the source code, for example to filter any values.
  * **generate-resources**	generate resources for inclusion in the package.
  * **process-resources**	copy and process the resources into the destination directory, ready for packaging.
  * **compile**	compile the source code of the project.
  * **process-classes**	post-process the generated files from compilation, for example to do bytecode enhancement on Java classes.
  * **generate-test-sources**	generate any test source code for inclusion in compilation.
  * **process-test-sources**	process the test source code, for example to filter any values.
  * **generate-test-resources**	create resources for testing.
  * **process-test-resources**	copy and process the resources into the test destination directory.
  * **test-compile**	compile the test source code into the test destination directory
  * **process-test-classes**	post-process the generated files from test compilation, for example to do bytecode enhancement on Java classes. For Maven 2.0.5 and above.
  * **test**	run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.
  * **prepare-package**	perform any operations necessary to prepare a package before the actual packaging. This often results in an unpacked, processed version of the package. (Maven 2.1 and above)
  * **package**	take the compiled code and package it in its distributable format, such as a JAR.
  * **pre-integration-test**	perform actions required before integration tests are executed. This may involve things such as setting up the required environment.
  * **integration-test**	process and deploy the package if necessary into an environment where integration tests can be run.
  * **post-integration-test**	perform actions required after integration tests have been executed. This may including cleaning up the environment.
  * **verify**	run any checks to verify the package is valid and meets quality criteria.
  * **install**	install the package into the local repository, for use as a dependency in other projects locally.
  * **deploy**	done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.

#### site生命周期

* siet生命周期的目的是建立和发布项目站点，maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息，包含的阶段如下：
  * **pre-site**	executes processes needed prior to the actual project site generation
  * **site**	generates the project's site documentation
  * **post-site**	executes processes needed to finalize the site generation, and to prepare for site deployment
  * **site-deploy**	deploys the generated site documentation to the specified web server

#### 命令行与生命周期

* **从命令行执行maven任务的最主要方式就是调用maven的生命周期\**阶段\****。需要注意的是，**各个生命周期是相互独立的，而一个生命周期的阶段是有前后依赖关系的**。例子如下：
  * `mvn clean`:该命令调用clean生命周期的clean阶段。实际执行的阶段为clean生命周期的pre-clean和clean阶段.
  * `mvn test`：该命令调用default生命周期的test阶段。实际调用的是default生命周期的validate、initialize等，直到test的所有阶段。
  * `mvn clean install`：该命令调换用clean生命周期的clean阶段和default生命周期的instal阶段

### Maven仓库配置优先级

* 仓库优先级为：本地仓库(localRepositories) > profile中的repositories仓库 > POM > mirrors全局仓库

#### 通过mirror配置

```xml
    <mirrors>
       <mirror>
            <id>alimaven</id>
            <name>aliyun central maven</name>
            <url>https://maven.aliyun.com/repository/central</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
        <mirror>
            <id>alimaven-public</id>
            <name>aliyun public maven</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <mirrorOf>public</mirrorOf>
        </mirror>
        <mirror>
            <id>alimaven-spring</id>
            <name>aliyun spring maven</name>
            <url>https://maven.aliyun.com/repository/spring</url>
            <mirrorOf>spring</mirrorOf>
        </mirror>
        <mirror>
            <id>alimaven-spring-plugin</id>
            <name>aliyun spring-plugin maven</name>
            <url>https://maven.aliyun.com/repository/spring-plugin</url>
            <mirrorOf>spring-plugin</mirrorOf>
        </mirror>
        <mirror>
            <id>alimaven-apache snapshots</id>
            <name>aliyun apache snapshots maven</name>
            <url>https://maven.aliyun.com/repository/apache snapshots</url>
            <mirrorOf>apache snapshots</mirrorOf>
        </mirror>
    </mirrors>
```

* maven是从aliyun仓库下载的jar包，不配置的时候，默认从apache的maven中央仓库下载的jar包。
* `<mirrorOf></mirrorOf>`的设置很重要，比如上面我设置的mirrorOf为`<mirrorOf>central</mirrorOf>`，如果`<mirrorOf></mirrorOf>`我随便设置一个参数，如`<mirrorOf>abc</mirrorOf>`，这时候我们配置的仓库就不起作用了，这是因为maven默认内置了如下一个仓库，这个默认仓库的id为central，当我们把mirrorOf设置为`<mirrorOf>central</mirrorOf>`时，maven就会查找有没有id为central的仓库，然后把id为central的仓库地址换成我们`<mirror>`标签配置的那个url，这样我们配置的mirror才会起作用。当然我们也可以把mirrorOf设置为`<mirrorOf>*</mirrorOf>`，表示所有仓库都使用我们配置的这个mirror作为jar包下载地址。

#### 通过`<repositories>`配置

* 通过setting.xml方式配置会对所有maven项目生效，如果只想在本项目中配置一个maven仓库，可以通过在pom.xml中配置`<repositories>`标签来实现。在自己的maven项目的pom.xml中添加如下配置，就配置好了一个仓库。这时候，maven会优先采用这个配置，而不会去读setting.xml中的配置了。这样配置好后，maven就会自动从aliyun下载jar包了.
* repositories标签下可以配置多个repository,如果我们配置了多个repository，是按出现顺序使用，如果第1个可用，就用第一个，如果不可用，就依次往下找.

```xml
<!--releases和snapshots中有个enabled属性，是个boolean值，默认为true，
表示是否需要从这个远程仓库中下载稳定版本或者快照版本的构建，
一般使用第三方的仓库，都是下载稳定版本的构建。-->
<repository>
  <id>aliyun-releases</id>
  <url>https://maven.aliyun.com/repository/public</url>
  <releases>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <enabled>false</enabled>
  </snapshots>
</repository>
```

### Maven常用参数和命令

####  **mvn常用参数**

* `mvn -e `显示详细错误
* `mvn -U` 强制更新snapshot类型的插件或依赖库（否则maven一天只会更新一次snapshot依赖）
* `mvn -o` 运行offline模式，不联网更新依赖
* `mvn -N`仅在当前项目模块执行命令，关闭reactor
* `mvn -pl module_name`在指定模块上执行命令
* `mvn -ff `在递归执行命令过程中，一旦发生错误就直接退出
* `mvn -Dxxx=yyy`指定java全局属性
* `mvn -Pxxx`引用profile xxx

#### **Build Lifecycle**命令

* `mvn test-compile` 编译测试代码
* `mvn test` 运行程序中的单元测试
* `mvn compile` 编译项目
* `mvn package` 打包，此时target目录下会出现maven-quickstart-1.0-SNAPSHOT.jar文件，即为打包后文件
* `mvn install` 打包并安装到本地仓库，此时本机仓库会新增maven-quickstart-1.0-SNAPSHOT.jar文件。
  每个phase都可以作为goal，也可以联合

#### **maven 日用三板斧**

* `mvn archetype:generate` 创建maven项目
* `mvn package `打包，上面已经介绍过了
* `mvn package -Prelease`打包，并生成部署用的包，比如deploy/*.tgz
* `mvn install` 打包并安装到本地库
* `mvn eclipse`eclipse 生成eclipse项目文件
* `mvn eclipse:clean` 清除eclipse项目文件
* `mvn site` 生成项目相关信息的网站

#### **maven插件常用参数**

* `mvn -Dwtpversion=2.0` 指定maven版本
* `mvn -Dmaven.test.skip=true` 如果命令包含了test phase，则忽略单元测试
* `mvn -DuserProp=filePath` 指定用户自定义配置文件位置
* `mvn -DdownloadSources=true -Declipse.addVersionToProjectName=true eclipse:eclipse` 生成eclipse项目文件，尝试从仓库下载源代码，并且生成的项目包含模块版本（注意如果使用公用POM，上述的开关缺省已打开）

#### **maven简单故障排除**

* `mvn -Dsurefire.useFile=false`如果执行单元测试出错，用该命令可以在console输出失败的单元测试及相关信息
* `set MAVEN_OPTS=-Xmx512m -XX:MaxPermSize=256m` 调大jvm内存和持久代
* `mvn -X maven log level`设定为debug在运行
* `mvn debug` 运行jpda允许remote debug



