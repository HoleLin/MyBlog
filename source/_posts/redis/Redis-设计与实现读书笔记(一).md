---
title: <<Redis设计与实现>>读书笔记(一)-数据结构与对象
date: 2021-12-17 10:46:07
index_img: /img/cover/Redis.jpg
cover: /img/cover/Redis.jpg
tags:
- 数据结构
categories: Redis
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

* Redis设计与实现-黄建宏

### 数据结构与对象

### 简单动态字符串(SDS,simple dynamic string)

#### SDS定义

```c
/*
 * 保存字符串对象的结构
 */
struct sdshdr {
    // buf 中已占用空间的长度
  	// 等于SDS所保存字符串的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 字节数组,用于保存字符串
    char buf[];
};
```

* SDS遵循C字符串以空字符结尾的管理,保存空字符的1字节空间不计算.在SDS的len属性里面,并且为空字符`'\0'`分配额外的1字节空间,以及添加空字符到字符串末尾等操作都是SDS函数自动完成的,所以这个空字符对SDS使用者来说是完全透明的.
* 遵循空字符结尾这一惯例的好处是,SDS可以直接重用C字符串函数库里面的函数.如可以直接使用`<stdio.h>/printf`函数.
* 根据传统,C语言使用长度为N+1的字符数组来表示长度为N的字符串,并且字符数组的最后一个元素总是空字符`'\0'`.

```c
sds sdsnewlen(const void *init, size_t initlen) {

    struct sdshdr *sh;

    // 根据是否有初始化内容，选择适当的内存分配方式
    // T = O(N)
    if (init) {
        // zmalloc 不初始化所分配的内存
        sh = zmalloc(sizeof(struct sdshdr)+initlen+1);
    } else {
        // zcalloc 将分配的内存全部初始化为 0
        sh = zcalloc(sizeof(struct sdshdr)+initlen+1);
    }

    // 内存分配失败，返回
    if (sh == NULL) return NULL;

    // 设置初始化长度
    sh->len = initlen;
    // 新 sds 不预留任何空间
    sh->free = 0;
    // 如果有指定初始化内容，将它们复制到 sdshdr 的 buf 中
    // T = O(N)
    if (initlen && init)
        memcpy(sh->buf, init, initlen);
    // 以 \0 结尾
    sh->buf[initlen] = '\0';

    // 返回 buf 部分，而不是整个 sdshdr
    return (char*)sh->buf;
}
```

#### SDS与C字符串的区别

##### 获取字符串长度的复杂度

* 因为C字符串不记录自身的长度信息,所以为了获取一个C字符串的长度,程序必须遍历整个字符串,对遇到的每个字符串进行计数,知道遇到代表字符串结尾的空字符为止,整个操作复杂度为O(n).
* SDS因为在len属性中记录了SDS本身的长度,所以获取一个SDS长度的复杂度为O(1).
* 通过使用SDS而不是C字符串,Redis将获取字符串长度所需的复杂度从O(N)降到了O(1),保证了获取字符串长度.

##### 避免缓冲区溢出

* 因为C字符串不记录自身长度,因此在使用的过程中容易出现缓冲区溢出(buffer overflow).

  * 例如:在使用`<string.h>/strcat`函数的时候,因为C字符串不记录自身长度,所以`strcat`假设用户在执行这个函数时,已经为`dest`分配了足够的内存.,可以容纳带拼接的字符串内容,而一旦这个假设不成立就会出现缓冲区溢出的现象.

    ```
    s1: Redis
    s2: Java
    假设s1,s2在内存中紧挨着
    'R' 'e' 'd' 'i' 's' '\0' 'J' 'a' 'v' 'a' '\0'
    当s1使用str函数 strcat(s1," Cluster")时没有给s1分配足够的内存空间,则会出现s1的数据溢出到s2的内存空间到,造成的后果是" Cluster"将"Java"内容篡改.
    ```

* 相对C字符串,SDS的空间分配完全避免了缓冲区溢出的出现.

  * 当SDSAPI需要对SDS进行修改,API会先检查SDS的空间是否满足修改所需要的要求,如果不满足的话,API会自动将SDS的空间扩展至执行所需要的大小,然后才执行实际的修改操作,所以使用SDS既不需要手动修改SDS的空间大小,也不会出现缓冲区溢出的问题.
  
  ```c
  /*
   * 将给定字符串 t 追加到 sds 的末尾
   * 
   * 返回值
   *  sds ：追加成功返回新 sds ，失败返回 NULL
   *
   * 复杂度
   *  T = O(N)
   */
  /* Append the specified null termianted C string to the sds string 's'.
   *
   * After the call, the passed sds string is no longer valid and all the
   * references must be substituted with the new pointer returned by the call. */
  sds sdscat(sds s, const char *t) {
      return sdscatlen(s, t, strlen(t));
  }
  ```
  
  ```c
  /*
   * 将长度为 len 的字符串 t 追加到 sds 的字符串末尾
   *
   * 返回值
   *  sds ：追加成功返回新 sds ，失败返回 NULL
   *
   * 复杂度
   *  T = O(N)
   */
  /* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
   * end of the specified sds string 's'.
   *
   * After the call, the passed sds string is no longer valid and all the
   * references must be substituted with the new pointer returned by the call. */
  sds sdscatlen(sds s, const void *t, size_t len) {
      
      struct sdshdr *sh;
      
      // 原有字符串长度
      size_t curlen = sdslen(s);
  
      // 扩展 sds 空间
      // T = O(N)
      s = sdsMakeRoomFor(s,len);
  
      // 内存不足？直接返回
      if (s == NULL) return NULL;
  
      // 复制 t 中的内容到字符串后部
      // T = O(N)
      sh = (void*) (s-(sizeof(struct sdshdr)));
      memcpy(s+curlen, t, len);
  
      // 更新属性
      sh->len = curlen+len;
      sh->free = sh->free-len;
  
      // 添加新结尾符号
      s[curlen+len] = '\0';
  
      // 返回新 sds
      return s;
  }
  ```
  
  ```c
  /*
   * 对 sds 中 buf 的长度进行扩展，确保在函数执行之后，
   * buf 至少会有 addlen + 1 长度的空余空间
   * （额外的 1 字节是为 \0 准备的）
   *
   * 返回值
   *  sds ：扩展成功返回扩展后的 sds
   *        扩展失败返回 NULL
   *
   * 复杂度
   *  T = O(N)
   */
  sds sdsMakeRoomFor(sds s, size_t addlen) {
  
      struct sdshdr *sh, *newsh;
  
      // 获取 s 目前的空余空间长度
      size_t free = sdsavail(s);
  
      size_t len, newlen;
  
      // s 目前的空余空间已经足够，无须再进行扩展，直接返回
      if (free >= addlen) return s;
  
      // 获取 s 目前已占用空间的长度
      len = sdslen(s);
      sh = (void*) (s-(sizeof(struct sdshdr)));
  
      // s 最少需要的长度
      newlen = (len+addlen);
  
      // 根据新长度，为 s 分配新空间所需的大小
      if (newlen < SDS_MAX_PREALLOC)
          // 如果新长度小于 SDS_MAX_PREALLOC 
          // 那么为它分配两倍于所需长度的空间
          newlen *= 2;
      else
          // 否则，分配长度为目前长度加上 SDS_MAX_PREALLOC
          newlen += SDS_MAX_PREALLOC;
      // T = O(N)
      newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);
  
      // 内存不足，分配失败，返回
      if (newsh == NULL) return NULL;
  
      // 更新 sds 的空余长度
      newsh->free = newlen - len;
  
      // 返回 sds
      return newsh->buf;
  }
  ```

##### 减少修改字符串时带来的内存重分配次数

* 
