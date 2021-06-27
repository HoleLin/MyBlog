# Ngnix笔记

## 目录

* Nginx的优点
* Nginx的三个应用场景
* Ngnix的组成
* Nginx版本
* 编译安装Nginx

### Nginx的优点

* 高并发,高性能
* 可扩展性好
* 高可用
* 热部署
* BSD许可证

### Nginx的三个应用场景

* 静态资源服务
  * 通过本地文件系统提供服务
* 反向代理服务
  * Nginx的强大性能
  * 缓存
  * 负载均衡
* API服务
  * OpenResty

### Ngnix的组成

* Nginx二进制可执行文件
  * 由各个模块源码编辑出的一个文件
* Ngnix.conf配置文件
  * 控制Nginx的行为
* access.log访问日志
  * 记录每一条HTTP请求信息
* error.log错误日志
  * 定位问题

### Nginx版本

* 开源免费的Nginx http://www.nginx.org
* 商业版的Nginx Plus http://www.nginx.com
* 阿巴巴巴的Tenginehttp://tengine.taobao.org/
* 免费OpenResty http://openresty.org
* 商业版OpenResty http://openresty.com

### 编译安装Nginx

* 下载Ngnix

* Nginx各个目录

  ![image-20210110205540681](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210110205540681.png)

  * 执行`cp -r contrib/vim/* ~/.vim/`,使得Vim识别nginx.conf的配置语法,使其高亮
  * 执行`./configure --help |more`,查看configure支持的配置

* Configure

* 中间文件(objs)

  ![image-20210110221350014](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210110221350014.png)

* 编译

  * `./configure --prefix=/home/holelin/nginx`
  * `make`

* 安装

  * `make install`

### Nginx配置语法

> * 配置文件由指令与指令块构成;
> * 每条指令以;分号结尾,指令与参数间以空格符号分隔;
> * 指令块以{}大括号将多条指令组织在一起;
> * include语法允许组合多个配置文件以提升可维护性;
> * 使用#符号添加注释,提高可读性;
> * 使用$符号使用变量;
> * 部分指令的参数支持正则表达式;

* 配置参数: 时间单位

  > * ms: milliseconds
  > * s: seconds
  > * m: minutes
  > * h: hours
  > * d: days
  > * w: weeks
  > * M: months,30days
  > * y: years,365days

* 配置参数: 空间单位

  > * bytes
  > * k/K: kilobytes
  > * m/M: megabytes
  > * g/G: gigabytes

### Nginx命令行

> 格式: nginx -s reload
>
> 帮助: -? /-h
>
> 使用指定的配置文件:  -c
>
> 指定配置指令: -g
>
> 指定运行目录: -p
>
> 发送信号: -s
>
> * 立即停止服务: stop
> * 优雅的停止服务: quit
> * 重载配置文件: reload
> * 重新开始记录日志文件: reopen
>
> 测试配置文件是否有语法: -t /-T
>
> 打印nginx的版本信息,编译信息等: -v -V

* 热部署升级
  * 复制一份最新的nginx(/sbin/nginx);
  * `kill -USR2  原PID` 平滑的启动新的进程(master和worker);
  * `kill -WINCH 原PID` 优雅的关闭老的worker进程;
  * 最终老的master进程会被保留,以备回滚版本时使用;
* 日志切割
  * 复制一份原有的日志文件 `mv xxx.log back.log`
  * `../sbin/nginx -s reopen`
  * 使用编写脚本,通过定时任务来进行日志切割