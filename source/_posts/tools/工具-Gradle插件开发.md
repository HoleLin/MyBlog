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

