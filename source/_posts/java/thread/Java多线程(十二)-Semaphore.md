---
title: Java多线程(十二)-Semaphore
date: 2021-06-30 23:02:30
cover: /img/cover/Java.jpg
tags:
- Semaphore
- 多线程
categories:
- Java
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

#### Semaphore概念

* Semaphore 通常我们叫它信号量， 可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

> 可以把它简单的理解成我们停车场入口立着的那个显示屏，每有一辆车进入停车场显示屏就会显示剩余车位减1，每有一辆车从停车场出去，显示屏上显示的剩余车辆就会加1，当显示屏上的剩余车位为0时，停车场入口的栏杆就不会再打开，车辆就无法进入停车场了，直到有一辆车从停车场出去为止。

#### Semaphore使用场景

* 通常用于那些资源有明确访问数量限制的场景，常用于限流 ;

#### Semaphore常见方法

```java

// 获取一个令牌，在获取到令牌、或者被其他线程调用中断之前线程一直处于阻塞状态。
acquire()  

// 获取一个令牌，在获取到令牌、或者被其他线程调用中断、或超时之前线程一直处于阻塞状态。
acquire(int permits)  

// 获取一个令牌，在获取到令牌之前线程一直处于阻塞状态（忽略中断）。
acquireUninterruptibly() 

// 尝试获得令牌，返回获取令牌成功或失败，不阻塞线程。 
tryAcquire()

// 尝试获得令牌，在超时时间内循环尝试获取，直到尝试获取成功或超时返回，不阻塞线程。
tryAcquire(long timeout, TimeUnit unit)

// 释放一个令牌，唤醒一个获取令牌不成功的阻塞线程。
release()

// 等待队列里是否还存在等待线程。
hasQueuedThreads()

// 获取等待队列里阻塞的线程数。
getQueueLength()

// 清空令牌把可用令牌数置为0，返回清空令牌的数量。
drainPermits()

// 返回可用的令牌数量。
availablePermits()
```

#### Semaphore示例

```java
package com.holelin.sundry.demo;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;
import java.util.concurrent.ThreadLocalRandom;

public class SemaphoreTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();

        //信号量，只允许 3个线程同时访问
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 10; i++) {
            final long num = i;
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    try {
                        //获取许可
                        semaphore.acquire();
                        //执行
                        System.out.println("Accessing: " + num);
                        Thread.sleep(ThreadLocalRandom.current().nextInt(1000));
                        //释放
                        semaphore.release();
                        System.out.println("Release..." + num);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        executorService.shutdown();
    }

}
```

