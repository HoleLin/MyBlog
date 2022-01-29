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

