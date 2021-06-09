---
title: Git学习笔记(待补充)
date: 2021-06-02 20:13:10
index_img: /img/cover/Git.jpg
tags: 
- git
categories: 
- 工具
---

### Git

#### 参考文献

* [Git Book](https://git-scm.com/book/en/v2)

#### **哈希**

> 哈希是一个系列的加密算法，各个不同的哈希算法虽然加密强度不同，但是有以下几个共同点：
>   ①不管输入数据的数据量有多大，使用同一个哈希算法，得到的加密结果长度固定
>   ②哈希算法确定，输入数据确定，输出结果保证不变
>   ③哈希算法确定，输入数据有变化，输出结果一定有变化，而且通常变化很大
>   ④哈希算法不可逆
>   ⑤哈希算法中不区分英文大小写
>   哈希算法有很多种，如：MD5、SHA-1等。Git 底层采用的是 **SHA-1** ，因为哈希算法可以被用来验证文件，Git 就是靠这种机制来从根本上保证数据完整性的

#### **Git特点**

> * 直接记录快照,而并非差异比较
> * Git把数据看作是对小型文件系统的一组快照,每次提交更新或Git中保存项目状态时,Git主要**对当时的全部文件制作一个快照并保存这个快照的索引**.为了高效,如果文件没有修改,Git不再重新存储改文件,而是只保留一个链接指向之前存储的文件,Git对待数据更像一个**快照流**.

#### **Git中文件的三种状态**

* **已暂存(staged)**: 表示对一个已修改的文件的当前版本做了标记,使之包含在下次提交的快照中.
* **已修改(modified)**: 表示修改了文件,但还没有保存到数据库中;

* **已提交(committed)**: 表示数据已经安全的保存在本地数据库中;

#### Git 三个区域

* **工作目录(Working tree)**: 是对项目的某个版本独立提取出来的内容;

* **暂存区(staging area)**: 是一个文件,保存了下次提交的文件信息,一般在Git仓库目录中.

* **Git仓库(Git directory)**: Git用来保存项目的元数据和对象数据库的地方,这是Git中最重要的部分,从其他计算机克隆仓库时,拷贝的就是这里的数据.

* **基本工作流程:**

  * 在工作目录中修改文件

    > 工作目录下的文件有两种状态:
    >
    > * 已跟踪: 指那些被纳入版本控制的文件
    >   * `add`之后文件处于`staged`状态等待`commit`
    >   * `commit`之后文件处于`unmodified`，**这里之所以是modified是因为文件会跟仓库中的文件对比**
    >   * 当`unmodified`的文件被修改则会变为`modified`状态
    >   * `modified`之后的文件add之后将继续变为staged状态
    >   * `unmodifed`的文件还有一种可能是已经不再需要了，那么可以`remove`它不再追踪变为`untracked`状态
    > * 未跟踪(untracked): 未被纳入版本控制的文件
    >
    > ![文件状态的生命周期](http://www.chenjunlin.vip/img/git/文件状态的生命周期.png)
    >
    > ```sh
    > git init 初始化git生成git仓库
    > git status 查看git状态
    > git add <filename>添文件到暂存区
    > git add .加入所有文件到暂存区
    > git commite -m 'message'提交文件到本地仓库
    > git reset <filename>将尚没有commite之前加入到暂存区的文件重新拉回
    > ```
  
  * 暂存文件,将文件的快照放入暂存区域
  
  * 提交更新,找到暂存区域中的文件,将快照永久性存储到Git仓库目录中;
  
  <img src="http://www.chenjunlin.vip/img/git/Git三个区域.png" alt="img" style="zoom:67%;" />

#### Git操作流程图

<img src="http://www.chenjunlin.vip/img/git/工作流程原理.png" alt="工作流程原理" style="zoom:80%;" />

#### Git命令

##### 初步配置

> git `config`可以配置Git外观和行为的配置变量,这些变量存储在三个不同的位置
>
> * `/etc/gitconfig`文件: **系统级** 使用`--system`设置
> * `~/.gitconfig`或`~/.config/git/config`: **只针对用户**,使用`--global`设置
> * 当前使用仓库的Git目录中的`config`文件(就是`.git/config`): **针对仓库**,使用该`--local`选项强制 Git 读取和写入此文件
>
> **Tips: 低级别的配置会覆盖高级别的配置**,即`.git/config`配置的变量会覆盖`/etc/gitconfig`中的配置信息

##### 用户信息

```sh
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

> 若使用了`--global`选项,那么该命令只需要运行一次,因为之后无论在系统上做任何事,Git都会使用这些配置信息.
>
> 若想对针对特定项目使用不同的用户名称与邮箱,可以在那个项目目录中配置用户信息时,不使用`--global`选项

##### 检查配置信息

```sh
# 查看所有配置
git config --list
# 查看单个配置
# 例: git config user.name
git config <key>  
# 查看配置来源哪个配置文件
git config --show-origin <key>
```

##### 获取帮助信息

```sh
git help <command>
git <command> --help
man git-<command>
```

##### 初始化仓库

```sh
git init 
```

##### 克隆已有仓库

```sh
git clone <url>
```

##### 检查文件状态

```sh
git status
# 显示简要信息
git status -s /git status --short
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
# 说明: 
# * ??: 新添加未跟踪的文件
# * A: 新添加到暂存区中的文件
# * M: 修改过的文件
# * MM: 在工作区修改并提交到了暂存区,又在工作区被修改了[输出有两列: 左侧表示暂存区的状态;右侧表示工作树的状态]
# *  M:出现在右边的M表示该文件被修改过但为加入到暂存区
# * M :出现在左边的M表示改文件被修改了并加入到了暂存区
```

##### 跟踪新文件(添加内容到下次提交中)

```sh
# 添加执行文件
git add <file>
# 添加所有文件
git add .
```

> * 可以用来开始跟踪新文件
> * 把已跟踪的文件放到暂存区
> * 用于合并是把有冲突的文件标记为已解决状态等

##### 忽略文件

> 处理无需纳入git管理的文件,文件后缀名为`.gitignore`
>
> `.gitignore`文件的模式规则如下: 
>
> * 所有空行或者以`#`开头的行都会被git忽略
>
> * 可以使用标准的[glob模式](https://en.wikipedia.org/wiki/Glob_(programming))匹配
>
>   * glob模式是指shell所使用的简化了的正则表达式。
>
>     * 星号（*）匹配零个或多个任意字符；
>
>     * [abc]匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；
>     * 问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。
>
> * 匹配模式可以以`/`开头防止递归
>
> * 匹配模式可以以`/`结尾指定目录
>
> * 要忽略指定模式以外的文件或目录,可以在模式前加上`!`
>
> [模板](https://github.com/github/gitignore)
>
> Tips:在简单的情况下，存储库`.gitignore`的根目录中可能只有一个文件，该文件递归地应用于整个存储库。但是，也可以`.gitignore`在子目录中包含其他文件。这些嵌套`.gitignore`文件中的规则仅适用于它们所在目录下的文件。Linux 内核源代码库有 206 个`.gitignore`文件。

##### 对比已暂存和未暂存的更改

```sh
# 要查看您已更改但尚未暂存的内容,此命令比较的是工作目录中当前文件和暂存区快照之间的差异,也就是修改之后还没有暂存起来的变化内容
git diff 
# 查看将进入下一次提交的暂存内容
git diff --staged / git diff --cached
```

> 需要注意的是，`git diff `本身并不显示自上次提交以来所做的所有更改 — 仅显示尚未暂存的更改。 如果您已暂存所有更改，则 `git diff `不会给您任何输出。

##### 提交更新

```sh
git commit -m "提交的信息"
# 跳过暂存区,执行该命令Git会自动把所有已经跟踪过的文件暂存起来一并提交,从而跳过git add步骤
# 等价于先执行git add + git commit -m
git commit -a -m "提交的信息"
# 提交完了发现漏掉了几个文件没有添加,或者提交信息写错了,可采用--amend参数
# 这个命令会将暂存区中的文件提交,若自上次提交以来未做任何修改(在上次提交后立马执行此命令),那么快照会保持不变,而修改的知识提交信息
git commit --amend -m [message]
# 重做上一次commit，并包括指定文件的新变化
git commit --amend [file1] [file2] ...
```

> **提交时记录的是放在暂存区的快照.**任何还未暂存的任然保持已修改状态,可以在下次提交时纳入版本管理.每次运行提交操作,都是对项目做一次快照,以后可以回到这个状态,或者进行比较.

##### 移除文件

```sh
# 该命令会将文件从暂存区移除,并连带工作目录的文件一并删除
# git rm log/\*.log
git rm <file>
# 强制删除
git rm -f <file>
# 保留在工作目录中,但是将其在暂存区中删除
git rm --cached <file>
```

> 从git中移除某个文件,就必须要从已跟踪文件清单移除(即从暂存区移除),然后提交

##### 移动文件

```sh
# 等价于下列三步
# mv README.md README
# git rm README.md
# git add README
git mv file_from file_to
```

##### 查看提交历史

```sh
# 不带参数的话,git log 会按提交时间列出所有更新,最近更新排在最上面
git log

# 用来显示每次提交的内容差异,也可以加上-2来仅显示最近两次提交
git log -P -2

# 用来显示每次提交的简略统计信息
# --stat 选项在每次提交的下面列出所有被修改过的文件,有多少文件被修改了以及修改过的文件哪些行被移除或添加,在每次提交的最后还有个总结
git log --stat

# 美化显示格式
# --pretty可以指定使用不同格式的方式展示提交历史
# 可选项有：oneline,short,medium,full,fuller,email,raw以及format:<string>,默认为medium
git log --pretty=oneline

# 列出那些添加或移除了某些字符串的提交
# 找出添加或移除了某个特定函数的引用的提交
git log -S function_name

# 显示某个文件的版本历史，包括文件改名
git log --follow [file]
git whatchanged [file]

# 显示指定文件相关的每一次diff
git log -p [file]

# 显示过去5次提交
git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
git shortlog -sn
# 格式化log
git log --pretty=format:"%h - %an, %ar : %s"
# 图形化
git log --pretty=format:"%h %s" --graph

```

| 说明符 | 输出说明                           |
| :----- | :--------------------------------- |
| `%H`   | 提交哈希                           |
| `%h`   | 缩写提交哈希                       |
| `%T`   | 树哈希                             |
| `%t`   | 缩写树哈希                         |
| `%P`   | 父哈希                             |
| `%p`   | 缩写的父哈希                       |
| `%an`  | 作者姓名                           |
| `%ae`  | 作者邮箱                           |
| `%ad`  | 作者日期（格式尊重 --date=option） |
| `%ar`  | 作者日期，亲属                     |
| `%cn`  | 提交者名称                         |
| `%ce`  | 提交者电子邮件                     |
| `%cd`  | 提交日期                           |
| `%cr`  | 提交日期，相对                     |
| `%s`   | 主题                               |

| 选项              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `-p`              | 显示每次提交引入的补丁。                                     |
| `--stat`          | 显示每次提交中修改的文件的统计信息。                         |
| `--shortstat`     | 仅显示来自 --stat 命令的更改/插入/删除行。                   |
| `--name-only`     | 显示提交信息后修改的文件列表。                               |
| `--name-status`   | 显示受添加/修改/删除信息影响的文件列表。                     |
| `--abbrev-commit` | 仅显示 SHA-1 校验和的前几个字符，而不是全部 40 个字符。      |
| `--relative-date` | 以相对格式（例如，“2 周前”）而不是使用完整日期格式显示日期。 |
| `--graph`         | 在日志输出旁边显示分支和合并历史的 ASCII 图形。              |
| `--pretty`        | 以替代格式显示提交。选项值包括 oneline、short、full、fuller 和 format（在其中指定您自己的格式）。 |
| `--oneline`       | `--pretty=oneline --abbrev-commit`一起使用的简写。           |

###### 限制日志输出

```sh
# 按照时间做限制的选项: --since --util
git log --since=2.weeks
# 根据若干条件搜索
# 显示指定作者的提交
git log --author "xxx"
# 搜索提交说明中的关键字
git log --grep "xxx"
# 指定目录或文件名，则可以将日志输出限制为对这些文件进行更改的提交
git log -- path/to/file
# 综合示例
git log --pretty="%h - %s" --author='xxx' --since="2021-06-01" --before="2021-07-01" --no-merges -- 
```

> tips: 若要得到同事满足这两个选项搜索条件的提交,就必须用`--all-match`选项,否则满足任意一个条件的提交都会被匹配;

| 选项                  | 描述                                     |
| :-------------------- | :--------------------------------------- |
| `-<n>`                | 只显示最近的 n 次提交                    |
| `--since`, `--after`  | 将提交限制为在指定日期之后进行的提交。   |
| `--until`, `--before` | 将提交限制为在指定日期之前进行的提交。   |
| `--author`            | 仅显示作者条目与指定字符串匹配的提交。   |
| `--committer`         | 仅显示提交者条目与指定字符串匹配的提交。 |
| `--grep`              | 只显示带有包含字符串的提交消息的提交     |
| `-S`                  | 只显示提交添加或删除匹配字符串的代码     |

##### 撤销

```sh
# 撤销对文件的修改 恢复暂存区的指定文件到工作区
git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
git checkout .

# 取消暂存文件 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
git reset [file]
# 操作暂存区与工作目录中已修改的文件
git reset HEAD <file>
# 重置暂存区与工作区，与上一次commit保持一致
git reset --hard
# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
git reset [commit]
# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
git reset --hard [commit]
# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
git reset --keep [commit]

# Git 2.23.0 版引入了一个新命令：git restore. 它基本上是git reset我们刚刚介绍的替代方案。从Git版本2.23.0开始，Git会使用git restore，而不是git reset许多撤消操作。
git restore --staged <file>
# 取消修改已修改的文件
git restore <file>

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
git stash
git stash pop
```

##### 操作远程仓库

###### 查看远程仓库

```sh
# 列出指定的每一个远程服务的简写
# origin 仓库服务器的默认名字
git remote 
# 显示相应读写远程仓库使用的Git保存的简写与其对应的URL
git remote -v 
```

###### 添加远程仓库

```sh
# 添加一个新的远程Git仓库,同时指定一个可以轻松引用的简写
git remote add <shortname> <url>
# 示例
git remote add pb https://github.com/paulboone/ticgit
```

###### 从远程仓库拉取

```>
git fetch <remote-name>
```

###### 推送到远程仓库

```sh
# 克隆是通常会设置好两个名字[origin master]
git push <remote-name> <branch-name>
```

> **Tips: 只有有所克隆服务器的写入权限,并且之前没有人推送是,这条命令才能生效.**
>
> 当你和其他人在同一时间克隆,他们先推送到上游后,你再推送到上游,你的推送将会被拒绝,必须先将他们的工作拉取下来并合并进你的工作目录后才能推送
>
> 故在每次推送之前建议先拉取一下最新的代码,若存在冲突,先解决冲突后在进行提交,操作如下:
>
> ````sh
> git pull 
> # 若有冲突,解决冲突
> git push
> ````

###### 查看某个远程仓库

```
git remote show <remote-name>
```

###### 修改远程仓库地址

```sh
# 第一种: 
git remote set-url <remote-name> <url>
# 第二种: 先删除后添加
git remote rm <remote-name>
git remote add <remote-name> <url>
# 第三种: 直接修改config文件
```

