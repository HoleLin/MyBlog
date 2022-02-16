---
title: 工具-Gradle插件开发
date: 2022-02-14 18:05:03
cover: /img/cover/Gradle.png
tags:
- Gradle
- Gradle Plugin
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

* [如何从零开发一个 gradle 插件（一）](https://zhuanlan.zhihu.com/p/86295227)
* [如何从零开发一个 gradle 插件（二）](https://zhuanlan.zhihu.com/p/86297410?from_voters_page=true)
* [Developing Custom Gradle Plugins](https://docs.gradle.org/current/userguide/custom_plugins.html)
* [Gradle 插件开发概述](https://add7.cc/android-base/gradle-cha-jian-kai-fa-gai-shu)
* [深度探索 Gradle 自动化构建技术（四、自定义 Gradle 插件）](https://juejin.cn/post/6844904135314128903)
* [手把手教你写 Gradle 插件 | 数据采集](https://opensource.sensorsdata.cn/opensource/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%86%99-gradle-%E6%8F%92%E4%BB%B6-%E6%95%B0%E6%8D%AE%E9%87%87%E9%9B%86/)

### Gradle 基础知识

* Gradle有两个重要的概念:`Project`和`Task`.

#### Project

* IDE结构中的项目相当于一个父 Project，而一个项目中所有的 Module 都是该父 Project 的子 Project
* 每个 Project 都会对应一个 build.gradle 配置文件，因此使用IDE创建一个项目的时候在根目录下有一个 build.gradle 文件，在每个 Module 的目录下又各有一个 build.gradle 文件;
* Gradle 是通过 settings.gradle 文件去进行多项目构建;
  * 父 Project 对象可以获取到所有的子 Project 对象，这样就可以在父 Project 对应的 build.gradle 文件中做一些统一的配置

#### Task

* Project 在构建过程中会执行一系列的 Task。Task 的中文翻译是 “任务”，它的作用其实也就是抽象了一系列有意义的任务，用 Gradle 官方的话说就是：`Each task perform some basic work`。
* Task 由两个部分组成：Task 所在的 Module 名和 Task 的名称。在运行 Task 的时候，也需要按照这样的方式去指定一个 Task。

#### Plugin 

* Plugin 和 Task 从它们的作用来看其实区别不大，都是把一些业务逻辑封装起来，Plugin 适用的场景是打包需要复用的编译逻辑（即把一部分编译逻辑模块化出来）。可以自定义 Gradle 插件，实现必要的逻辑后把它发布到远程仓库或者打成本地 JAR 包分享出去。这样，后续想要再次使用它或者想分享给别人使用的时候，就可以直接引用远程仓库的包或者引用本地的 JAR 包。

#### Extension

* 如果希望插件的功能更加灵活的话，一般会预留一些可配置的参数;

* 只需要新建一个普通的类，类中定义的属性就是 Extension 可以接收的配置。它不需要继承任何类，也不需要实现任何接口

  ```groovy
  class MyExtension{
      public String nam = "name"
      public String sur = "surname"
  }
  ```

* 通过 ExtensionContainer 来创建和管理 Extension，ExtensionContainer 对象可以通过 Project 对象的 getExtensions 方法获取

  ```groovy
  def extension = project.getExtensions().create('myExt',MyExtension)
  project.afterEvaluate {
      println("Hello from " + extension.toString())
  }
  ```

### Gradle 构建的生命周期

* 官方对于 Gradle 构建的生命周期的定义：Gradle 的核心是一种基于依赖的语言，用 Gradle 的术语来说这意味着你能够定义 Task 和 Task 之间的依赖关系。Gradle 会保证这些 Task 按照依赖关系的顺序执行并且每个 Task 只会被执行一次，这些 Task 根据依赖关系构成了一个有向无环图。Gradle 在执行任何 Task 之前都会用内部的构建工具去完成这样这样一个图，这就是 Gradle 的核心。这种设计使得很多原本不可能的事情成为可能。
* 每次 Gradle 构建都需要经过三个不同的阶段：
  * **初始化阶段**：Gradle 是支持单项目和多项目构建的，因此在初始化阶段，Gradle 会根据 settings.gradle 确定需要参与构建的项目，并为每个项目创建一个 Project 实例。Android Studio 的项目和 Module 对 Gradle 来说都是项目;
  * **配置阶段**：在这个阶段会配置 Project 对象，并且所有项目的构建脚本都会被执行。例如：Extension 对象、Task 对象等都是在这个阶段被放到 Project 对象里；
  * **执行阶段**：经过了配置阶段，此时所有的 Task 对象都在 Project 对象中。然后，会根据终端命令指定的 Task 名字从 Project 对象中寻找对应的 Task 对象并执行。

![img](https://www.holelin.cn/img/tools/gradle/Gradle%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%B5%81%E7%A8%8B%E7%AE%80%E5%9B%BE.png)

### Gradle插件简介

* 自定义 Gradle 插件的本质就是把逻辑独立的代码进行抽取和封装，以便于我们更高效地通过插件依赖这一方式进行功能复用。
* Gradle 插件共分为 两大类，如下所示：
  - 脚本插件：同普通的 Gradle 脚本编写形式一样，通过` apply from: 'JsonChao.gradle' `引用。
  - 对象插件：通过插件全路径类名或 id 引用，它主要有 三种编写形式，如下所示：
    - 1）在当前构建脚本下直接编写。
      - 优势: 插件会自动编译并包含在构建脚本的类路径中
      - 劣势: 构建脚本之外是不可见的,因此复用性较差
    - 2）在 `buildSrc` 目录下编写。
      - 插件的源代码放在`rootProjectDir/buildSrc/src/main/java`目录中（`rootProjectDir/buildSrc/src/main/groovy`或`rootProjectDir/buildSrc/src/main/kotlin`
    - 3）在完全独立的项目中编写。

### Gradle插件开发流程

* 环境

  ```
  Gradle 7.1-bin
  ```

* 新建Gradle项目,选择添加Groovy框架支持

    ![img](https://www.holelin.cn/img/tools/gradle/plugin/%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%911.png)

    ![img](https://www.holelin.cn/img/tools/gradle/plugin/%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%912.png)

* 新建`src/main/java`,`src/main/groovy`,`src/main/resources`目录

  * `groovy`目录和`resources`目录必须要创建,`java`目录可选

  ![img](https://www.holelin.cn/img/tools/gradle/plugin/%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95.png)

* 在`src/main/groovy`目录下编写插件代码

  ```groovy
  package cn.holelin.plugin
  
  import org.gradle.api.Plugin
  import org.gradle.api.Project
  
  class MyPlugin implements Plugin<Project> {
      @Override
      void apply(Project project) {
          project.task('test-plugin') {
              doLast {
                  println 'Hello from the GreetingPlugin'
              }
          }
      }
  }
  ```

* 注册Gradle插件

  * 在`src/main/resources`目录下创建`META-INF`目录,

  * 在`META-INF`下面创建`gradle-plugins`目录

  * 在`gradle-plugins`目录下创建`[插件名].properties`文件

  * 在`[插件名].properties`文件中添加`implementation-class`来注册编写的插件实现类

    ```properties
    // 文件名 cn.holelin.plugin.test.properties
    implementation-class=cn.holelin.plugin.MyPlugin
    ```

    * `cn.holelin.plugin.test.properties`中的插件名很重要,就是在`apply`时的插件名

* 插件发布

    * 可以使用`maven-publish`插件来将编写好的插件发布到本地或者远程服务器上.
    
    ```groovy
    plugins {
        id 'maven-publish'
    }
    
    publishing {
        publications {
            mavenPub(MavenPublication) {
                // 这一行表示将 jar 包包含在要发布的组件中
                from components.java
                // 描述性信息
                groupId = 'cn.holelin.plugin'
                artifactId = 'test'
                version = '1.0-SNAPSHOT'
            }
        }
        repositories {
            maven {
                // 本地 repo 地址，这里写上你们电脑上自己的地址
                url = '/Users/holelin/.m2/repository'
            }
        }
    }
    ```
    
* 最终的`build.gradle`内容

    ```groovy
    plugins {
        id 'groovy'
        id 'java'
        id 'maven-publish'
    }
    
    group 'cn.holelin.plugin'
    version '1.0-SNAPSHOT'
    
    repositories {
        maven {
            url = 'https://maven.aliyun.com/repository/public'
        }
        mavenCentral()
    }
    
    dependencies {
        implementation gradleApi()
        implementation localGroovy()
    }
    
    publishing {
        publications {
            mavenPub(MavenPublication) {
                // 这一行表示将 jar 包包含在要发布的组件中
                from components.java
                // 描述性信息
                groupId = 'cn.holelin.plugin'
                artifactId = 'test'
                version = '1.0-SNAPSHOT'
            }
        }
        repositories {
            maven {
                // 本地 repo 地址，这里写上你们电脑上自己的地址
                url = '/Users/holelin/.m2/repository'
            }
        }
    }
    ```
    
* 使用插件(本地)

    * 声明插件需要在 Project 级别的 build.gradle 文件中完成，在 build.gradle 文件中有一个块叫做 buildscript，buildscript 块又分为 repositories 块和 dependencies 块。repositories 块用来声明需要引用的依赖所在的远程仓库地址，dependencies 块用来声明具体引用的依赖。

        * 在需要使用到的应用的`build.gradle`中添加或者补充

        ```groovy
        // 添加本地仓库源以及依赖
        buildscript {
            repositories {
                maven{
                    url = '/Users/holelin/.m2/repository'
                }
            }
            dependencies {
                classpath "cn.holelin.plugin:test:1.0-SNAPSHOT"
            }
        }
        // 应用插件
        apply plugin: 'cn.holelin.plugin.test'
        ```

* 添加完成后,更新应用的`build.gradle`文件,会在`Task`中发现我们编写的插件,双击运行一下,则会打印`MyPlugin`中的输出信息

    ![img](https://www.holelin.cn/img/tools/gradle/plugin/%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%914.png)

    ![img](https://www.holelin.cn/img/tools/gradle/plugin/%E8%BF%90%E8%A1%8C%E6%8F%92%E4%BB%B6.png)

