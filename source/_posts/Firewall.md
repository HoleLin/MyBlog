---
title: Firewall
date: 2021-05-25 22:02:17
tags:
- Linux
- 防火墙
categories: 
- Linux
---
### firewalld

#### firewalld常用命令

* **启动firewalld服务**:

  ````shell
  systemctl start firewalld.service
  ````

* **关闭firewalld服务**:

  ```shell
  systemctl stop firewalld.service
  ```

* **重启firewalld服务**

  ```shell
  systemctl restart firewalld.service
  ```

* **查看firewalld状态**

  ```shell
  systemctl status firewalld.service
  ```

*  **开机自启firewalld**

  ```shell
  systemctl enable firewalld
  ```

* **查看版本**

  ```shell
  firewall-cmd --version
  ```

* **查看帮助**

  ```shell
  firewall-cmd --help
  ```

* **显示状态**

  ```shell
  firewall-cmd --state
  ```

*  **查看所有打开的端口**

  ```shell
  firewall-cmd --zone=public --list-ports
  ```

* **更新防火墙规则**

  ```shell
  firewall-cmd --reload
  * 每次更改firewall规则后需重新加载
  ```

* **添加开放端口**

  ```shell
  firewall-cmd --zone=public --add-port=80/tcp --permanent (permanent永久生效，没有此参数重启后失效)
  ```

* **查看端口是否开放**

  ```shell
  firewall-cmd --zone=public --query-port=80/tcp
  ```

* **删除开放端口**

  ```shell
  firewall-cmd --zone=public --remove-port=80/tcp --permanent
  ```

  
