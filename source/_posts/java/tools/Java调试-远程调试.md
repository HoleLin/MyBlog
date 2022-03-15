---
title: Java调试-远程调试
date: 2021-05-26 22:51:33
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags: 
- 远程调试
categories: 
- Java
- Base
---

### Java 远程调试

 * 启动脚本中添加选项，并重启(JavaSE 5以后java -agentlib:jdwp=...)

   ```
    JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"
   ```

 * JavaSE 5之前(java -Xdebug -Xrunjdwp:...)

   ```java
    CATALINA_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,address=10086,suspend=n,server=y
   ```

 * 参数说明

   * transport
     指定运行的被调试应用和调试者之间的通信协议，有如下可选值：
     * dt_socket： 采用socket方式连接（常用）
     * dt_shmem：采用共享内存的方式连接，支持有限，仅仅支持windows平台
   * server
     * 指定当前应用作为调试服务端还是客户端，默认的值为n（客户端）。
     * 如果你想将当前应用作为被调试应用，设置该值为y;如果你想将当前应用作为客户端，作为调试的发起者，设置该值为n。
   * suspend
     * 当前应用启动后，是否阻塞应用直到被连接，默认值为y（阻塞）。
     * 大部分情况下这个值应该为n，即不需要阻塞等待连接。一个可能为y的应用场景是，你的程序在启动时出现了一个故障，为了调试，必须等到调试方连接上来后程序再启动。
   * address
     * 对外暴露的端口，默认值是8000
     * 注意：此端口不能和项目同一个端口，且未被占用以及对外开放。
   * onthrow
     * 这个参数的意思是当程序抛出指定异常时，则中断调试。
   * onuncaught
     * 当程序抛出未捕获异常时，是否中断调试，默认值为n。
   * launch
     * 当调试中断时，执行的程序。
   * timeout
     * 超时时间，单位ms（毫秒）
     * 当 suspend = y 时，该值表示等待连接的超时；当 suspend = n 时，该值表示连接后的使用超时。