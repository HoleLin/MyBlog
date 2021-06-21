---
title: Java多线程-ReetrantLock
mermaid: true
date: 2021-06-18 00:06:08
tags:
categories:
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

### 特点

* 可中断; `reentrantLock.lockInterruptibly();`
* 可设置超时时间;`reentrantLock.ryLock(long timeout, TimeUnit unit);`
* 可以设置为公平锁;`new ReentrantLock(true);`
* 支持多个条件变量;
* 与`synchronized`一样都支持可重入;

#### 基本用法

```java
		ReentrantLock reentrantLock = new ReentrantLock();
        reentrantLock.lock();
        try {
            // 临界区
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 释放锁
            reentrantLock.unlock();
        }
```

#### 条件变量

> `synchronized`中也有条件变量,就是那个`WaitSet`休息室,当条件不满足进入`WaitSet`等待`ReentrantLock`的条件变量比`synchronized`强大之处在于,它是支持多个条件变量的,这就好比:
>
> * `synchronized`是哪些不满足的条件都在一个休息室等消息;
> * 而`ReentrantLock`支持多间休息室,有专门等烟的休息室,专门等早餐的休息室,唤醒也是按休息室来唤醒;

* 使用要点
  * `await`前需要获得锁;
  * `await`执行后,会释放锁,进入`conditionObject`等待;
  * `await`的线程被唤醒(或打断,或超时),会重新竞争lock锁;
  * 竞争lock锁成功后,从`await`后继续执行

```java
package com.holelin.sundry.test.thread;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j
public class ReentrantLockTest {
    private static ReentrantLock reentrantLock = new ReentrantLock();
    private static Condition waitCigaretteQueue = reentrantLock.newCondition();
    private static Condition waitBreakfastQueue = reentrantLock.newCondition();
    private static volatile boolean hasCigarette = false;
    private static volatile boolean hasBreakfast = false;

    public static void main(String[] args) {
        new Thread(() -> {
            reentrantLock.lock();
            try {
                while (!hasCigarette) {
                    try {
                        waitCigaretteQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.info("获取到烟");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                reentrantLock.unlock();
            }
        }).start();

        new Thread(() -> {
            reentrantLock.lock();
            try {
                while (!hasBreakfast) {
                    try {
                        waitBreakfastQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.info("获取到早饭");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                reentrantLock.unlock();
            }
        }).start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        sendBreakfast();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        sendCigarette();
    }

    private static void sendBreakfast() {
        reentrantLock.lock();
        try {
            log.info("送早餐");
            hasBreakfast = true;
            waitBreakfastQueue.signal();
        } finally {
            reentrantLock.unlock();
        }
    }

    private static void sendCigarette() {
        reentrantLock.lock();
        try {
            log.info("送烟");
            hasCigarette = true;
            waitCigaretteQueue.signal();
        } finally {
            reentrantLock.unlock();
        }
    }
}

22:40:43.703 [main] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 送早餐
22:40:43.704 [Thread-1] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 获取到早饭
22:40:44.712 [main] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 送烟
22:40:44.712 [Thread-0] INFO com.holelin.sundry.test.thread.ReentrantLockTest - 获取到烟
```



