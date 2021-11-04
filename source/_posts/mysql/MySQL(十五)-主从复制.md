---
title: MySQL(十五)-主从复制
date: 2021-08-16 23:11:45
index_img: /img/cover/MySQL.jpg
cover: /img/cover/MySQL.jpg
tags:
- 主从复制
categories:
- MySQL
---

### 参考文献

* [MySQL 主从复制](https://juejin.cn/post/6972194988473974821?utm_source=gold_browser_extension)

### 主从复制简介

> 复制是 MySQL 的一项功能，允许服务器将更改从一个实例复制到另一个实例。
>
> 1）主服务器将所有数据和结构更改记录到二进制日志中。
> 2）从属服务器从主服务器请求该二进制日志并在本地应用其内容。
> 3）IO：请求主库，获取上一次执行过的新的事件，并存放到relaylog
> 4）SQL：从relaylog中将sql语句翻译给从库执行

### 主从复制的方式

* 目前MySQL支持两种复制方式:
  * **传统方式**
    * 基于主库的binlong将日志事件和事件位置复制到从库,从库加以应用来达到主从同步的目的;
  * **`GTID`方式(MySQL>=5.7推荐使用)**
    * 基于`GTID`的复制中,从库会告知主库已经执行的事务的`GTID`的值,然后主库会将所有未执行的事务的`GTID`的列表返回给从库,并且可以保证同一个事务只在指定的从库执行一次;

### 多种复制类型

* 异步复制

  >一个主库，一个或多个从库，数据异步同步到从库。

* 同步复制

  > 在MySQL cluster中特有的复制方式。

* 半同步复制

  > 在异步复制的基础上，确保任何一个主库上的事物在提交之前至少有一个从库已经收到该事物并日志记录下来。

* 延迟复制

  > 在异步复制的基础上，人为设定主库和从库的数据同步延迟时间，即保证数据延迟至少是这个参数。

### 主从复制原理

#### 主从复制的前提

> 1）两台或两台以上的数据库实例
> 2）主库要开启二进制日志
> 3）主库要有复制用户
> 4）主库的server_id和从库不同
> 5）从库需要在开启复制功能前，要获取到主库之前的数据（主库备份，并且记录binlog当时位置）
> 6）从库在第一次开启主从复制时，时必须获知主库：ip，port，user，password，logfile，pos
> 7）从库要开启相关线程：IO、SQL
> 8）从库需要记录复制相关用户信息，还应该记录到上次已经从主库请求到哪个二进制日志
> 9）从库请求过来的binlog，首先要存下来，并且执行binlog，执行过的信息保存下来

#### 主从复制涉及到的文件和线程

* 主库
  * 主库`binlog`：记录主库发生过的修改事件
  * `dump thread`：给从库传送（TP）二进制日志线程
* 从库
  * `relay-log`（中继日志）：存储所有主库TP过来的binlog事件
  * `master.info`：存储复制用户信息，上次请求到的主库binlog位置点
  * `IO thread`：接收主库发来的binlog日志，也是从库请求主库的线程
  * `SQL thread`：执行主库TP过来的日志

####  MySQL主从复制的流程

1. 主库db的更新事件(update、insert、delete)被写到binlog

2. 主库创建一个binlog dump thread，把binlog的内容发送到从库

3. 从库启动并发起连接，连接到主库

   * 通过change master to语句告诉从库主库的ip，port，user，password，file，pos

4. 从库通过start slave命令开启复制必要的IO线程和SQL线程

   * 从库启动之后，创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log

   * 从库启动之后，创建一个SQL线程，从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db

5. 从库通过IO线程拿着change master to用户密码相关信息，连接主库，验证合法性

6. 从库连接成功后，会根据binlog的pos问主库，有没有比这个更新的

7. 主库接收到从库请求后，比较一下binlog信息，如果有就将最新数据通过dump线程给从库IO线程从库通过IO线程接收到主库发来的binlog事件，存储到TCP/IP缓存中，并返回ACK更新master.info

8. 将TCP/IP缓存中的内容存到relay-log中

9. SQL线程读取relay-log.info，读取到上次已经执行过的relay-log位置点，继续执行后续的relay-log日志，执行完成后，更新relay-log.info

#### MySQL主从复制的原理

> MySQL主从复制是一个异步的复制过程，主库发送更新事件到从库，从库读取更新记录，并执行更新记录，使得从库的内容与主库保持一致。

* **主库**

  * **binlog**

    > binary log，主库中保存所有更新事件日志的二进制文件。`binlog`是数据库服务启动的一刻起，保存数据库所有变更记录（数据库结构和内容）的文件。在主库中，只要有更新事件出现，就会被依次地写入到`binlog`中，之后会推送到从库中作为从库进行复制的数据源

  * **binlog输出线程**

    > 每当有从库连接到主库的时候，主库都会创建一个线程然后发送binlog内容到从库。 对于每一个即将发送给从库的sql事件，binlog输出线程会将其锁住。一旦该事件被线程读取完之后，该锁会被释放，即使在该事件完全发送到从库的时候，该锁也会被释放。

* 从库

  > 在从库中，当复制开始时，从库就会创建从库I/O线程和从库的SQL线程进行复制处理。

  * **从库I/O线程**

    > 当`START SLAVE`语句在从库开始执行之后，从库创建一个I/O线程，该线程连接到主库并请求主库发送`binlog`里面的更新记录到从库上。 从库I/O线程读取主库的`binlog`输出线程发送的更新并拷贝这些更新到本地文件，其中包括`relay log`文件。

  * **从库的SQL线程**

    > 从库创建一个SQL线程，这个线程读取从库I/O线程写到relay log的更新事件并执行。

### 主从复制实战

#### 配置主从数据库服务器参数

##### Master操作

* Master 服务器参数：

  ```toml
  [mysqld]
  log-bin = /home/mysql-master/data/mysql-bin
  binlog_format = mixed
  #开启binlog日志
  log_bin=mysql-bin
  server-id = 100
  #expire_logs_days = 10 #日志过期时间
  #max_binlog_size = 200M #日志最大容量，可以不设置，有默认值，
  binlog_do_db = test
  #binlog_do_db 指定记录二进制日志的数据库，即需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可
  ```

* 在 Master 服务器上建立复制账号

  * 需要设置 REPLICATION SLAVE 权限：

    ```sql
    CREATE USER '账号'@'master主机IP' IDENTIFIED BY '密码';
    GRANT REPLICATION SLAVE ON *.* TO 'repl'@'slavez主机IP';   
    flush privileges; #刷新权限
    ```
  
* 查看 Master 的` binlog `的文件名和 `binlog` 偏移量

  ```sql
  # 查看 Master 的 binlog 的文件名和 binlog 偏移量
  mysql> show master status;
  +---------------+----------+--------------+------------------+-------------------+
  | File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +---------------+----------+--------------+------------------+-------------------+
  | binlog.000004 |      376 |              |                  |                   |
  +---------------+----------+--------------+------------------+-------------------+
  1 row in set (0.06 sec)
  ```

##### Slave操作

* Slave 服务器参数：

  ```toml
  [mysqld]
  log-bin = /home/mysql-slave/data/mysql-bin
  binlog_format = mixed
  server-id = 200
  #expire_logs_days = 10 #日志过期时间
  #max_binlog_size = 200M #日志最大容量，可以不设置，有默认值，设置后MySQL无法重启，我遇到情况
  
  relay_log = /home/mysql-slave/data/relay-bin
  #指定relay_log日志的存放路径和文件前缀 ，不指定的话默认以主机名作为前缀
  
  read_only = on
  skip_slave_start = on
  
  #下面两个参数是把主从复制信息存储到innodb表中，默认情况下主从复制信息是存储到文件系统中的，如果从服务器宕机，很容易出现文件记录和实际同步信息不同的情况，存储到表中则可以通过innodb的崩溃恢复机制来保证数据记录的一致性
  master_info_repository = TABLE
  relay_log_info_repository = TABLE
  ```

* 在 Slave 服务器的操作

  * 配置slave服务器：

    ```toml
    # 通过change master to语句告诉从库主库的ip，port，user，password，file，position
    CHANGE MASTER TO
    MASTER_HOST='slave主机IP',
    MASTER_USER='之前创建的主从复制账号',
    MASTER_PASSWORD='之前创建的主从复制账号',
    # MASTER_LOG_FILE和MASTER_LOG_POS填入的值是在主库中执行`show master status`返回的File,Position的值
    MASTER_LOG_FILE='binlog.000004',
    MASTER_LOG_POS=376;
    ```

#### Master 和 Salve 数据库的数据保持一致

> 主从数据库的数据要保持一致，不然主从同步会出现 bug

##### 主库已经有数据的解决方案

* 第一种方案是选择忽略主库之前的数据，不做处理。这种方案只适用于不重要的可有可无的数据，并且业务上能够容忍主从库数据不一致的场景。

* 第二种方案是对主库的数据进行备份，然后将主数据库中导出的数据导入到从数据库，然后再开启主从复制，以此来保证主从数据库数据一致。

  * 锁定主数据库，只允许读取不允许写入，这样做的目的是防止备份过程中或备份完成之后有新数据插入，导致备份数据和主数据数据不一致。

    ```sql
    flush tables with read lock;
    ```

  * 通过 MySQL 主服务器上的全备初始化从服务器上数据：

    ```sh
    cd /data/db_backup/
    mysqldump -uroot -p --master-data=1 --single-transaction --routines --triggers --events  --all-databases > all.sql
    ```

  * 解锁主数据库,然后把数据全量导入 Slave 数据库，保证主从数据一致

    ```sql
    unlock tables;
    ```

  * 开始主从同步

    ```sql
    start slave; # 开启从同步
    
    show slave status; # 查看从节点状态
    ```

* **注意事项**

  * 如果出现IO线程一直在Connecting状态，可以看看是不是俩台机器无法相互连接，如果可以相互连接，那么有可能是Slave账号密码写错了，重新关闭Slave然后输入上面的配置命令再打开Slave即可。

  * 如果出现SQL线程为NO状态，那么有可能是从数据库和主数据库的数据不一致造成的，或者事务回滚，如果是后者，先关闭`stop slave`，然后先查看master的binlog和position，然后输入配置命令，再输入`set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;`，再重新`start slave;`即可，如通过是前者，那么就排查一下是不是存在哪张表没有被同步，是否存在主库存在而从库不存在的表，自己同步一下再重新配置一遍即可。

  * `Could not find first log file name in binary log index file`

    * 如果查看从库状态发现此问题，请查看主库状态，将其中的File和Position字段通过在从库中执行以下SQL语句写入从库配置中。

      ```sql
      change master to master_log_file='mysql-bin.000001', master_log_pos=3726;
      ```

  * `ERROR 1872 (HY000): Slave failed to initialize relay log info structure from the repository`

    * 如果启动slave时出现此错误，主要可能是因为保存着以前slave用的relay-log，可以执行以下语句来启动slave。

      ```sql
      reset slave;
      start slave;
      ```
  
  #### **主从复制基本故障处理**
  
  ##### **IO线程报错解决思路**
  
  ```toml
  # IO线程报错：
  解决思路：
  1.网络
  [root@db02 ~]# ping 10.0.0.51
  1）硬件层，路由，交换机，网络设备
  2）网线
  3）安全组规则
  4）插错网线口
  
  2.端口
  [root@db02 ~]# telnet 10.0.0.51 3306
  #关闭防火墙
  systemctl stop firewalld
  #防火墙添加允许mysql端口
  firewalld-cmd --add-service=mysql 
  firewalld-cmd --add-port=3306/tcp
  
  3.用户名
  mysql> grant replication slave on *.* to rep@'%' identified by '123';
  
  4.密码，先登录测试
  [root@db03 data]# mysql -urep -p123 -h10.0.0.51
  
  如果报错  #rep@'db03'，需在参数，跳过反向解析
  vim /etc/my.cnf
  skip_name_resolve
  
  #搭建主从时，用户名、密码、主机域、端口一定要一致。
   change master to
   master_host='10.0.0.51',#1
   master_user='rep',#2
   master_password='123',#3
   master_log_file='mysql-bin.000004',
   master_log_pos=376,
   master_port=3306;
  ```
  
  ##### SQL线程报错
  
  * **处理方法一**
  
    ```sql
    #临时停止同步
    mysql> stop slave;
    #将同步指针向下移动一个（可重复操作）
    mysql> set global sql_slave_skip_counter=1;
    #开启同步
    mysql> start slave;
    ```
  
  * **处理方法二：**
  
    ```sh
    #编辑配置文件
    [root@db01 ~]# vim /etc/my.cnf
    #在[mysqld]标签下添加以下参数，把线程号添加到配置文件
    slave-skip-errors=1032,1062,1007
    ```
  
  * **但是方法一、方法二都是有风险存在的，只是跳过错误，不能从根本上解决问题**
  
  * **处理方法三**
  
    * 重新备份数据库，恢复到从库
  
    * 给从库设置为只读
  
      ```toml
      #在命令行临时设置
      set global read_only=1;
      #在配置文件中永久生效
      read_only=1
      ```
  
      ```toml
      #设置配置文件永久生效
      [root@db03 ~]# vim /etc/my.cnf
      read_only=1
      #重启
      [root@db03 ~]# /etc/init.d/mysqld  restart
      Shutting down MySQL.. SUCCESS! 
      Starting MySQL. SUCCESS!
      #查看
      mysql> show variables like 'read_only';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | read_only     | ON    |
      +---------------+-------+
      1 row in set (0.00 sec)
      ```
  
      
  
    * ***注意：登录用户如果是all权限，包含了super超级权限，还是可以进行操作的***
  
      * all 权限，即使配置文件设置了只读，还是都可以操作的。
  
        ```sh
        [root@db03 ~]# mysql
        mysql> grant all on *.* to rea@'%' identified by '123';
        Query OK, 0 rows affected (0.00 sec)
        
        [root@db03 ~]# mysql -urea -p123 -h 10.0.0.53
        mysql> create database aaa;
        Query OK, 1 row affected (0.01 sec)
        ```
  
      * 不加all权限。哪怕给他指定select,insert, delete ,create 权限，都是不能操作，只能只读的
  
        ```sh
        mysql> grant select,create,delete,insert on *.* to rea1@'%' identified by '123';
        Query OK, 0 rows affected (0.00 sec)
        
        [root@db03 ~]# mysql -urea1 -p123 -h10.0.0.53
        mysql> create database bbb;
        ERROR 1290 (HY000): The MySQL server is running with the --read-only option so it cannot execute this statement
        mysql> drop database test;
        ERROR 1290 (HY000): The MySQL server is running with the --read-only option so it cannot execute this statement
        ...
        ```
  
        
  
      

​    

​    

