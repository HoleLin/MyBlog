---
title: MySQL(二)-运维操作(二)
date: 2021-06-13 16:07:22
cover: /img/cover/MySQL.jpg
tags:
- 运维操作
categories:
- MySQL
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

### 慢查询

### 命令行接口

* `mysql --column-type-info` 显示结果集元数据

  ```sh
  -- 字段名
  Field  11:  `produced_time`
  -- 目录名称
  Catalog:    `def`
  -- 数据库名称
  Database:   `test`
  -- 数据表名称, 当使用select field_name from table_name as alias_name语法时,这里显示的是表的别名
  Table:      `orders`
  -- 原始表名,当前面行显示的别名,知道这个就非常有用了
  Org_table:  `orders`
  -- 前面的行显示字段类型
  Type:       DATETIME
  -- 排序规则
  Collation:  binary (63)
  -- 表定义中定义的字段长度
  Length:     26
  -- 返回结果集字段长度的最大值
  Max_length: 0
  -- 如果是一个整数类型,则表示该字段中小数点后的位数
  Decimals:   6
  Flags:      BINARY 
  ```

### MySQL沙箱

* [参考文献](https://www.cnblogs.com/gomysql/p/3767445.html)

### MySQL通用日志

* 开启通用日志

  ```mysql
  set global general_log=1;
  set global log_output='table';
  ```

* 查看通用日志的内容

  ```mysql
  SELECT argument FROM mysql.general_log ORDER BY	event_time desc
  ```

### 故障排除的一般步骤

* 尝试确定造成问题的实际查询;
* 检查以确保查询的语句正确;
* 确认查询里有问题;
* 如果查询返回错误数据时,请尝试重写它以得到正确的结果;
* 如果重写没用,可以检查服务器选项并尝试确定它们是否影响结果;
* 如果问题不能再MySQL CLI重现,请检查它是否有并发问题;
* 如果该问题会导致系统崩溃或挂起,首先检查错误日志;

### MySQL基准工具

#### `mysqlslap`

* `mysqlslap`是MySQL发布版中自带的一个负载模拟客户端.

#### [`SysBench`](https://github.com/akopytov/sysbench)

* 它可以用来测试整个系统的性能,包括测试CPU,文件I/O,操作系统调度程序,POSIX线程性能,内存分配,传输速度与数据库服务性能.

### 转换表的引擎的处理方式

#### `ALTER TABLE`

```mysql
ALTER TABLE mytable ENGINE=InnoDB;
```

* **注意点:**需要执行很长时间.MySQL会按行将数据从原表复制到一张新的表中,在复制期间可能会消耗系统所有的I/O能力,同时原表上会加上读锁.所以在繁忙的表上执行此操作要特别小心.

#### 导出与导入

* 使用`mysqldump`工具将数据导出到文件,然后修改文件中的`CREATE TABLE`语句的存储引擎选项,注意同时修改表名.同时注意`mysqldump`默认会自动在`CREATE TABLE`语句之前加上`DROP TABLE`语句.

#### 创建与查询

```mysql
CREATE TABLE innodb_table LIKE myisam_table;
ALTER TABLE innodb_table ENGINE=InnoDB;
START TRANSACTION;
INSERT INTO innodb_table SELECT * FROM myisam_table WHERE id BETWEEN x AND y;
COMMIT;
```

### 大表修改工具

* [`pt-online-schema-change`](https://www.percona.com/doc/percona-toolkit/3.0/pt-online-schema-change.html)

* [Online Schema Changes](https://www.cockroachlabs.com/docs/stable/online-schema-changes.html)

* [openark kit](https://code.openark.org/forge/openark-kit)
* [Percona Toolkit](https://www.percona.com/software/database-tools/percona-toolkit)

### MySQL服务器配置

* MySQL获取配置信息路径
  * 命令行参数 `mysqld_safe --datadir=/data/sql_data`
  * 配置文件

* 确定配置的文件路径

  ```shell
  root@2dd900ce424a:/# which mysqld
  /usr/sbin/mysqld
  root@2dd900ce424a:/# /usr/sbin/mysqld --verbose --help |grep -A 1 'Default options'
  Default options are read from the following files in the given order:
  /etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
  ```

* 配置文件通常分成多个部分,每个部分的开头是一个方括号括起来的分段名称.MySQL程序通常读取跟它同名的分段部分,很多客户端程序还会读取client部分,这是一个存放共用设置的地方.服务器通常读取mysqld这一段.一定要确认配置项放在了文件正确的分段中,否则配置是不会生效的.

* MySQL配置参数的作用域

  * 全局参数

    ```properties
    set global 参数名=参数值;
    set @@global.参数名:=参数值; 
    ```

  * 会话参数

    ```properties
    set [session] 参数名=参数值;
    set @@session.参数名:=参数值; 
    ```

* MySQL配置

  ```properties
  # 内存相关
  # 确定可以使用的内存的上限
  # 确定MySQL的每个连接使用的内存
  
  # 针对每个连接分配内存的
  # 排序缓冲区
  sort_buffer_size
  # 连接缓冲区 
  join_buffer_sizes
  read_buffer_size
  read_rnd_buffer_size
  
  # 缓存池分配内存 若服务器只有MySQL: 总内存-(每个线程所需要的线程*连接数)-系统保留内存
  Innodb_buffer_pool_size
  key_buffer_size
  
  # I/O相关
  Innodb_log_file_size
  Innodb_log_files_in_group
  # 事务日志总大小=Innodb_log_files_in_group * Innodb_log_file_size
  Inondb_log_buffer_size
  # 0: 每秒进行一次log写入cache,并flush log到磁盘.设置该值当MySQL崩溃时,会丢失至少一秒的数据
  # 1: 默认,在每次事务提交执行log写入cache,并flush log到磁盘
  # 2: 每次事务提交,执行到log数据写入到cache,每秒执行一次flush log到磁盘
  Innodb_flush_log_at_trx_commit
  
  Innodb_flush_method = O_DIRECT
  Innodb_file_per_table = 1
  # 双写缓存
  Innodb_doublewrite = 1
  
  # 安全相关
  # 指定自动清理binlog的天数
  expire_logs_days
  # 控制MySQL可以接收的包的大小
  max_allowed_packet
  # 禁止DNS查找
  skip_name_resolve
  # 确保sysdate()返回确定性日志
  sysdate_is_now
  # 禁止非super权限的用户写权限
  read_only
  # 禁止Slave自动恢复
  skip_slave_start
  # 设置MySQL所使用的SQL模式
  sql_mode
  #①ONLY_FULL_GROUP_BY
  # 对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中
  #②NO_AUTO_VALUE_ON_ZERO
  # 该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。
  #③STRICT_TRANS_TABLES
  # 如果一个值不能插入到一个事务中，则中断当前的操作，对非事务表不做限制
  #④NO_ZERO_IN_DATE
  # 不允许日期和月份为零
  #⑤NO_ZERO_DATE
  # mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告
  #⑥ERROR_FOR_DIVISION_BY_ZERO
  # 在insert或update过程中，如果数据被零除，则产生错误而非警告。如果未给出该模式，那么数据被零除时Mysql返回NULL
  #⑦NO_AUTO_CREATE_USER
  # 禁止GRANT创建密码为空的用户
  #⑧NO_ENGINE_SUBSTITUTION
  # 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常
  #⑨PIPES_AS_CONCAT
  # 将"||"视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样是，也和字符串的拼接函数Concat想类似
  #⑩ANSI_QUOTES
  # 不能用双引号来引用字符串，因为它被解释为识别符
  
  # 控制MySQL如何向磁盘刷新binlog
  sync_binlog
  # 控制内存表临时大小,两个参数应该保持一致
  tmp_table_size
  max_heap_table_size
  # 控制允许的最大连接数
  max_connections
  ```

#### 内核相关参数

* `/etc/sysctl.conf`

```properties
# 网络
net.core.somaxconn=65535
net.core.netdev_max_backlog=65535
net.ipv4.tcp_max_syn_backlog=65535

net.ipv4.tcp_fin_timeout=10
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_tw_recycle=1

net.core.wmen_default=87380
net.core.wmen_max=16777216
net.core.rmem_default=87380
net.core.rmem_max=16777216

net.ipv4.tcp_keepalive_time=120
net.ipv4.tcp_keepalive_intvl=30
net.ipv4.tcp_keepalive_probes

# 内存
# 用于定义单个共享内存段的最大值
# 这个参数应该设置的足够大,以便能在一个共享内存段下容纳整个的InnoDB缓冲池的大小
# 这个值的大小对于64位Linux系统,可取的最大值为物理内存值-1byte,建议值为大于物理内存的一半,一般取值大于InnoDB缓冲池的大小即可,可以取物理内存-1byte.
kernel.shmmax=4294967295
# 这个参数当内存不足时会对性能产生比较明显的影响,表示除非内存被占满了,否则不会使用交换分区
vm.swappiness=0
# Linux系统内存交换区:在Linux系统安装时都会有一个特殊的磁盘分区,称之为系统交换分区
# 交换分区的作用:当操作系统因为没有足够的内存时就将一些虚拟内存写到磁盘的交换区中,这样就会发生内存交换.
```

#### 增加资源限制

* `/etc/security/limit.conf`
* 这个文件实际上是Linux PAM也即是插入式认证模块的配置文件.打开文件数的限制

```properties
# * 表示对所有用户有效
# soft 指的是当前系统生效的设置
# hard 表明系统中所能设定的最大值
# nofile 表示锁限制的资源是打开文件的最大数目
# 65535 表示限制的数据量
* soft nofile 65535
* hard nofile 65535
```

#### 磁盘调度策略

* `/sys/block/devname/queue/scheduler`

```properties
noop 
anticipatory 
deadline 
# 默认
cfq
# 修改
# echo deadline > /sys/block/sda/queue/scheduler
```

##### noop(电梯式调度策略)

* NOOP实现了一个FIFO队列,他像电梯的工作方法一样对I/O请求进行组织,当有一个新的请求到来时,它将请求合并到最近的请求之后,以此来保证请求同一介质.NOOP倾向饿死读而利于写,因此NOOP对于闪存设备,RAM以及嵌入式系统是最好的选择.

##### deadline(截止时间调度策略)

* deadline确保了在一个截止时间内服务请求,这个截止时间是可以调整的,而默认读期限短于写期限.这样就防止了写操作因为不能被读取而饿死的现象,deadline对数据库类应用是最好的选择.

##### antiipatory(预料I/O调度策略)

* 本质上与deadline一样,但在最后一次读操作后,要等待6ms,才能继续进行其他I/O请求进行调度.它会在每个6ms中插入新的I/O操作,而会将一些小写入流合并成一个大写入流,用写入延时换取最大的写入吞入量.AS适合写入较多的环境,比如文件服务器,AS对数据库环境表现很差.

#### 文件系统对服务器的影响

* Windows

  * FAT
  * NTFS

* Linux

  * EXT3

  * EXT4

  * XFS

##### EXT3/4系统的挂载参数

* `/etc/fstab`

```properties
# data = writeback | ordered | journal
# noattime,nodiratime
# 示例
/dev/sda1/ext4 noatime,nodiratime,data=writebck 1 1
```

#### 检查MySQL服务器状态变量

* 可以使用`SHOW GLOBAL STATUS`,也可以使用下面的命令每隔60秒来查看状态变量的增量变化

  ```shell
  mysqladmin extended-status -ri60
  ```


### MySQL磁盘选择与配置

#### 使用传统机械硬盘

* 特点: 
  * 最常见
  * 使用最多
  * 价格低
  * 存储空间大
  * 读写速度较慢
* 传统机器硬盘读取数据的过程:(访问时间[1,2]+传输速度[3])
  * 移动磁头到磁盘表面上的正确位置;
  * 等待磁盘旋转,使的所需的数据在磁头之下;
  * 等待磁盘旋转过去,所有所需的数据都被磁头读出;

### 使用RAID增强传统机械硬盘的性能

* RAID: 磁盘冗余队列的简称(Redundant Arrays of Independent Disks)
  * 作用就是可以把多个容量较小的磁盘组成一组容量更大的磁盘,并提供数据冗余来保证数据完整性的技术.

#### RAID 0

* 是最早出现的RAID模式,也称为数据条带.是组成磁盘阵列中最简单的一种形式,只需要两块以上的磁盘即可,成本低,可以提高整个磁盘的性能和吞吐量.**RAID 0没有提供冗余或者错误修复的能力**,但是实现成本是最低的.

#### RAID 1

* 又被称为磁盘镜像,原理是把一个磁盘的数据镜像到另外一个磁盘上,也就是说数据在写入一块一块磁盘的同事,会在另一块闲置的磁盘上生成镜像文件,在不影响性能情况下**最大限度的保证系统的可靠性和可修复性**.

#### RAID 5

* 分布式奇偶校验磁盘阵列
* 通过分布式奇偶校验块把数据分散到多个磁盘上,这样如果任何一个盘数据失效,都可以从奇偶校验块中重建.但是如果两块磁盘失效,则整个卷的数据都无法恢复.

#### RAID 10

* 分片的镜像
* 它是磁盘先做RAID 1之后对两组RAID 1的磁盘再做RAID 0,所以对读写都有良好的性能,相对RAID 5重建起来更简单,速度也更快.

### 使用固态存储SSD和PCIe卡

* 相对机械磁盘固态磁盘有更好的随机读写性能;
* 相对机械磁盘固态磁盘能更好的支持并发;
* 相对磁盘固态磁盘更容易损坏;

### 使用网络存储NAS和SAN

