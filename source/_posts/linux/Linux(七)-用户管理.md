---
title: Linux(七)-用户管理
date: 2021-07-24 20:35:56
index_img: /img/cover/Linux.jpg
cover: /img/cover/Linux.jpg
tags:
- 用户管理
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

#### 用户管理

##### `useradd`

* 新建用户

```sh
-c<备注>：加上备注文字。备注文字会保存在passwd的备注栏位中；
-d<登入目录>：指定用户登入时的启始目录；
-D：变更预设值；
-e<有效期限>：指定帐号的有效期限；
-f<缓冲天数>：指定在密码过期后多少天即关闭该帐号；
-g<群组>：指定用户所属的群组；
-G<群组>：指定用户所属的附加群组；
-m：自动建立用户的登入目录；
-M：不要自动建立用户的登入目录；
-n：取消建立以用户名称为名的群组；
-r：建立系统帐号；
-s<shell>：指定用户登入后所使用的shell；
-u<uid>：指定用户id。
```

```sh
#-g：加入主要组、-G：加入次要组
useradd –g sales jack –G company,employees    
```

##### `userdel`

* 删除用户

```sh
-f：强制删除用户，即使用户当前已登录；
-r：删除用户的同时，删除与用户相关的所有文件。
```

##### `passwd`

* 修改用户密码

```sh
passwd (选项) (参数)
-d：删除密码，仅有系统管理者才能使用；
-f：强制执行；
-k：设置只有在密码过期失效后，方能更新；
-l：锁住密码；
-s：列出密码的相关信息，仅有系统管理者才能使用；
-u：解开已上锁的帐号。
```

##### `usermod`

* 修改用户属性

```sh
usermod (选项) (参数)
-c<备注>：修改用户帐号的备注文字；
-d<登入目录>：修改用户登入时的目录，只是修改/etc/passwd中用户的家目录配置信息，不会自动创建新的家目录，通常和-m一起使用；
-m<移动用户家目录>:移动用户家目录到新的位置，不能单独使用，一般与-d一起使用。
-e<有效期限>：修改帐号的有效期限；
-f<缓冲天数>：修改在密码过期后多少天即关闭该帐号；
-g<群组>：修改用户所属的群组；
-G<群组>；修改用户所属的附加群组；
-l<帐号名称>：修改用户帐号名称；
-L：锁定用户密码，使密码无效；
-s<shell>：修改用户登入后所使用的shell；
-u<uid>：修改用户ID；
-U:解除密码锁定。
```

```sh
# 将 newuser2 添加到组 staff 中
usermod -G staff newuser2

# 修改newuser的用户名为newuser1
usermod -l newuser1 newuser

# 锁定账号newuser1：
usermod -L newuser1

# 解除对newuser1的锁定
usermod -U newuser1
```

##### `chage` 

* 修改用户属性

```
chage [选项] 用户名
-m：密码可更改的最小天数。为零时代表任何时候都可以更改密码。
-M：密码保持有效的最大天数。
-w：用户密码到期前，提前收到警告信息的天数。
-E：帐号到期的日期。过了这天，此帐号将不可用。
-d：上一次更改的日期。
-i：停滞时期。如果一个密码已过期这些天，那么此帐号将不可用。
-l：例出当前的设置。由非特权用户来确定他们的密码或帐号何时过期。
```



##### `id 用户名`

* 查看用户是否存在
* 若不加参数,则显示当前登录用户

```sh
[root@holelin ~]# id root
uid=0(root) gid=0(root) groups=0(root)
[root@holelin ~]# id
uid=0(root) gid=0(root) groups=0(root)
```

##### `whoami/who am i`

* 查看当前用户

```sh
[root@holelin ~]# who am i
root     pts/0        2021-07-24 18:41 (192.168.12.114)
[root@holelin ~]# whoami
root
```

##### `su 用户名`

* 用于切换当前用户身份到其他用户身份

```sh
su - user1
```

```sh
-c<指令>或--command=<指令>：执行完指定的指令后，即恢复原来的身份；
-f或——fast：适用于csh与tsch，使shell不用去读取启动文件；
-l或——login：改变身份时，也同时变更工作目录，以及HOME,SHELL,USER,logname。此外，也会变更PATH变量；
-m,-p或--preserve-environment：变更身份时，不要变更环境变量；
-s<shell>或--shell=<shell>：指定要执行的shell；
--help：显示帮助；
--version；显示版本信息。
```

##### `sudo`

* **用来以其他身份来执行命令**，预设的身份为root。在`/etc/sudoers`中设置了可执行`sudo`指令的用户。若其未经授权的用户企图使用`sudo`，则会发出警告的邮件给管理员。用户使用`sudo`时，必须先输入密码，之后有5分钟的有效期限，超过期限则必须重新输入密码。

```sh
-b：在后台执行指令；
-h：显示帮助；
-H：将HOME环境变量设为新身份的HOME环境变量；
-k：结束密码的有效期限，也就是下次再执行sudo时便需要输入密码；。
-l：列出目前用户可执行与无法执行的指令；
-p：改变询问密码的提示符号；
-s<shell>：执行指定的shell；
-u<用户>：以指定的用户作为新的身份。若不加上此参数，则预设以root作为新的身份；
-v：延长密码有效期限5分钟；
-V ：显示版本信息。
```

##### `groupadd`

* 新增用户组

```sh
-g：指定新建工作组的id；
-r：创建系统工作组，系统工作组的组ID小于500；
-K：覆盖配置文件“/ect/login.defs”；
-o：允许添加组ID号不唯一的工作组。
```

##### `groupdel`

* 删除组

##### `groupmod`

* 修改组

```sh
-g<群组识别码>：设置欲使用的群组识别码；
-o：重复使用群组识别码；
-n<新群组名称>：设置欲使用的群组名称。
```





