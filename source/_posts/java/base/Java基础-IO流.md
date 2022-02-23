---
title: Java基础-I/O流
date: 2022-02-09 22:28:07
cover: /img/cover/Java.jpg
tags:
- IO
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

* [IO 多路复用](https://mp.weixin.qq.com/s/CMWlDywI1zbgJSoeGTBmuw)
* Java语言设计-基础篇(第8版)

### `File`类

* `File`对象封装了文件或路径的属性,但是它既不包括创建文件,也不包括从(向)文件读(写)数据的方法.

### 文件输入和输出

#### `PrintWriter`

* `java.io.PrintWriter`类可以用来创建一个文件并向文本文件写入数据.首先必须为一个文本文件创建一个`PrintWriter`对象.

  ```java
          final PrintWriter printWriter = new PrintWriter(new File(""));
  				printWriter.println("");
  				printWriter.close();
  ```

#### `Scanner`

* `java.util.Scanner`类用来从控制台读取字符串和基本类型数值.`Scanner`可以将输入分为由空白字符串分隔的有用信息.

  ```java
          final Scanner scanner = new Scanner(System.in);
          while (scanner.hasNext()) {
              final String next = scanner.next();
          }
          scanner.close();
  ```

#### `FileInputStream`类和`FileOutputStream`类

* `FileInputStream`类和`FileOutputStream`类是为了从(向)文件读取(写入)字节.它们的所有方法都是从`InputStream`类和`OutputStream`类继承的.

  ```java
      		FileOutputStream outputStream = null;
          try {
              outputStream = new FileOutputStream(new File(""));
              outputStream.write(0);
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (Objects.nonNull(outputStream)) {
                      outputStream.close();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
  
          FileInputStream inputStream = null;
          try {
              inputStream = new FileInputStream(new File(""));
              int value;
              while ((value = inputStream.read()) != -1) {
                  System.out.println(value);
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (Objects.nonNull(inputStream)) {
                      inputStream.close();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
  ```

#### `FilterInputStream`类和`FilterOutputStream`类

* 过滤器数据流(filter stream)是为某种目的的过滤字节的数据流.基本字节输入流提供的读取方法`read`只能用来读取字节.如果要读取整数值,双精度值或字符串,那就需要一个过滤类器类来包装字节输入流.使用过滤器类就可以读取整数值,双精度值和字符串,而不是字节或字符.
* `FilterInputStream`类和`FilterOutputStream`类是过滤数据的基类,需要处理基本数据值类型时,就使用`DataInputStream`类和`DataOutputStream`类来过滤字节.

#### `DataInputStream`类和`DataOutputStream`类

* `DataInputStream`从数据流读取字节,并且将他们转换为正确的基本了死刑值或字符串.
* `DataOutputStream`将基本类型的值或字符串转换为字节,并且将字节输出到数据流.

```java
			  DataOutputStream dataOutputStream = null;
        try {
            dataOutputStream = new DataOutputStream(new FileOutputStream(""));
            dataOutputStream.writeUTF("123");

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (Objects.nonNull(dataOutputStream)) {
                    dataOutputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

#### `BufferedInputStream`类和`BufferedOutputStream`类

* `BufferedInputStream`类和`BufferedOutputStream`类可以通过减少读写次数来提高输入和输出的速度.
* `BufferedInputStream`类和`BufferedOutputStream`类没有包含新的方法.所有方法都是从`InputStream`类和`OutputStream`类继承而来.
* `BufferedInputStream`类和`BufferedOutputStream`类为存储字节在流中添加一个缓冲区,以提高处理效率.
* 如果没有指定缓冲区大小,默认为512个字节.缓冲区输入流会在每次读取调用中尽可能多地将数据读入缓冲区.相反地,只有当缓冲区已满或调用`flush()`方法时,缓冲输出流才会调用写入方法.

#### `DataInputStream`类和`DataOutpuStream`类

* `DataInputStream`类和`DataOutpuStream`类可以实现基本数据类型与字符串的输入和输出,还可以实现对象的输入输出.

#### 随机访问文件

* `RandomAccessFile`类允许在文件内的随机位置上进行读写.`RandomAccessFile`类实现了`DataInput`和`DataOutput`接口.

* 当创建一个`RandomAccessFile`数据流时,可以指定两种模式(`r`或`rw`)之一.

  ```java
          final RandomAccessFile raf = new RandomAccessFile("filename.txt", "rw");
  ```

  * 如果文件已存在,则创建raf访问这个文件;如果文件不存在,则创建一个名为''filename.txt"的新文件,再创建raf访问这个文件

* 随机访问文件是由字节序列组成的.一个称为文件指针(`file poniter`)的特殊标记定位这些字节中的某个字节的位置.文件读写操作就是在文件指针的位置进行的.打开文件时,文件指针置于文件的起始位置.在文件中进行读写数据后,文件指针就会向前移到下一个数据项.







