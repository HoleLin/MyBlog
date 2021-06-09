---
title: pm2
date: 2021-05-25 22:33:12
index_img: /img/cover/Node.jpg
cover: /img/cover/Node.jpg
tags:
- pm2 
- node
categories: Node
---
### pm2

#### 参考文献

* [pm2 QuickStart](https://pm2.keymetrics.io/docs/usage/quick-start/)

#### 常用命令

* 安装

  ```
  $ npm install pm2@latest -g
  # or
  $ yarn global add pm2
  ```

* 启动app

  ```
  $ pm2 start app.js
  # Specify an app name
  --name <app_name>
  
  # Watch and Restart app when files change
  --watch
  
  # Set memory threshold for app reload
  --max-memory-restart <200MB>
  
  # Specify log file
  --log <log_path>
  
  # Pass extra arguments to the script
  -- arg1 arg2 arg3
  
  # Delay between automatic restarts
  --restart-delay <delay in ms>
  
  # Prefix logs with time
  --time
  
  # Do not auto restart app
  --no-autorestart
  
  # Specify cron for forced restart
  --cron <cron_pattern>
  
  # Attach to application log
  --no-daemon
  ```

* 管理

  ```
  // 重启
  $ pm2 restart app_name
  // 重新加载
  $ pm2 reload app_name
  // 停止
  $ pm2 stop app_name
  // 删除
  $ pm2 delete app_name
  ```

  
