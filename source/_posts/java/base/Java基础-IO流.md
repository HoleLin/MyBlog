---
title: Java基础-I/O流
date: 2022-02-09 22:28:07
cover: /img/cover/Java.jpg
tags:
- IO
categories:
- Java
- Base
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
* Java核心技术 卷2 高级特性

### `File`类

* `File`对象封装了文件或路径的属性,但是它既不包括创建文件,也不包括从(向)文件读(写)数据的方法.

![img](https://www.holelin.cn/img/java/io/InputStream%E5%92%8COutputStream%E7%B1%BB%E4%BB%A5%E5%8F%8A%E5%AD%90%E7%B1%BB.png)

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

​		![img](https://www.holelin.cn/img/java/io/DataInputStream%E7%B1%BB%E5%9B%BE.png)

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

#### 随机访问文件-`RandomAccessFile`

* `RandomAccessFile`类允许在文件内的随机位置上进行读写.`RandomAccessFile`类实现了`DataInput`和`DataOutput`接口.

* 当创建一个`RandomAccessFile`数据流时,可以指定两种模式(`r`或`rw`)之一.

  ```java
          final RandomAccessFile raf = new RandomAccessFile("filename.txt", "rw");
  ```

  * 如果文件已存在,则创建raf访问这个文件;如果文件不存在,则创建一个名为''filename.txt"的新文件,再创建raf访问这个文件

* 随机访问文件是由字节序列组成的.一个称为文件指针(`file poniter`)的特殊标记定位这些字节中的某个字节的位置.文件读写操作就是在文件指针的位置进行的.打开文件时,文件指针置于文件的起始位置.在文件中进行读写数据后,文件指针就会向前移到下一个数据项.

#### ZIP文档

* ZIP文档通常以压缩格式存储了一个或多个文件,每个ZIP文档都有一个头,包含诸如每个文件名和所使用的压缩方法等信息.

* 在Java中可以使用`ZipInputStream`来读入ZIP文档.若需要浏览文档中每个单独的项,`getNextEntry`方法就可以返回一个描述这些项的`ZipEntry`类型的对象.

* `ZipInputStream`的read方法被修改为在碰到当前项的结尾是返回-1(而不是碰到ZIP文件的末尾),然后必须调用`closeEntry`来读入下一项.

  ```java
  			final ZipInputStream zipInputStream = new ZipInputStream(new FileInputStream(""));
        ZipEntry zipEntry;
        while ((zipEntry=zipInputStream.getNextEntry())!=null){
              // analyze entry
              // read the content of zin
              zipInputStream.closeEntry();
        }
        zipInputStream.close();
  ```

  * **注意:** 在读入单个ZIP项后,不要关闭ZIP输入流,也不要将其传递给可能会关闭它的方法,否则就不能再读入后续的项了.

* 写出到ZIP文件,可以使用`ZipOutputStream`,若希望放入到ZIP文件中每一项,都应该创建一个`ZipEntry`对象,并将文件名传递给`ZipEntity`的构造器,它将设置其他诸如文件日期和解压缩方法等参数.然后需要调用`ZipOutputStream`的`putNextEntry`方法来开始写出新文件,并将文件数据发送到ZIP流中,最后完成时,需要调用`closeEntry`.

  ```java
  			final ZipOutputStream zipOutputStream = new ZipOutputStream(new FileOutputStream("test.zip"));
          // deal all files
          {
              ZipEntry ze=new ZipEntry("filename");
              zipOutputStream.putNextEntry(ze);
              // send data to zipOutputStream
              zipOutputStream.closeEntry();
          }
          zipOutputStream.close();
  ```

  * JAR文件只是带有一个特殊项的ZIP文件,这个项被称为清单.可以使用`JarInputStream`和`JarOutputStream`来处理.

### 操作文件

#### `Path`

* Path表示的是一个目录名序列,其后还可以跟着一个文件名.路径中的第一个部件可以是根部件,如`/`或`D:\`.而允许访问的根部件取决于文件系统.
* 以根部件开始的路径是绝对路径;否则,就是相对路径.
* 静态的`Paths.get`方法接收一个或多个字符串,并将它们用默认文件系统的路径分隔符连接起来.然后解析连接起来的结果,如果其表示不是给定文件系统中的合法路径,那么就抛出`InvalidPathException`异常.

##### 组合和解析

* `p.resolve(q)`将按照下列规则返回一个路径:

  * 如果q是绝对路径,则结果就是q;
  * 否则,根据文件系统的规则,将"p后面跟着q"作为结果;

* `resolve`方法是一种快捷方式,它接收一个字符串而不是路径:

  ```java
  	Path workPath=basePath.resolve("work");
  ```

* `resolveSibling`它通过解析指定路径的父路径产生其兄弟路径.

  * 例如,若workPath是"/opt/myapp/work"那么下面的调用将创建"/opt/myapp/temp"

  ```java
  Path tempPath=workPath.resolveSibling("temp");
  ```

* `resolve`的对立是`relative`即调用`p.relative(r)`将产生路径q,对q进行解析会产生r.

  * 例如"/home/cay"为目标对"/home/fred/myprog"进行相对化操作,会产生"../fred/myprog",其中假设".."为文件系统的父目录.

* `normalize`方法将移除所有冗余的"."和".."部件(或者文件系统认为冗余的所有部件).例如规范化"/home/cay/../fred/./myprog"将产生"/home/cay/fred/myprog"

#### `Files`

* `Files`类可以使得普通文件操作变得快捷.

```java
				// 读取文件
        final byte[] bytes = Files.readAllBytes(Paths.get(""));
        // 将文件当做行序列读入
        final List<String> strings = Files.readAllLines(Paths.get(""), Charset.defaultCharset());
        // 写出一个字符串到文件中
        final Path write = Files.write(Paths.get(""), "".getBytes(Charset.defaultCharset()));
        // 想指定的文件中追加内容
        final Path append = Files.write(Paths.get(""), "".getBytes(Charset.defaultCharset()), StandardOpenOption.APPEND);

			  // 复制文件
        Files.copy(Paths.get(""),Paths.get(""));
        // 移动文件
        Files.move(Paths.get(""),Paths.get(""));

        Files.copy(Paths.get(""),Paths.get(""), StandardCopyOption.REPLACE_EXISTING,StandardCopyOption.COPY_ATTRIBUTES);
				// 保持移动的操作为原子性
        Files.move(Paths.get(""),Paths.get(""),StandardCopyOption.ATOMIC_MOVE);

				// 删除文件 可以用来移除空目录
 				Files.delete(Paths.get(""));
        Files.deleteIfExists(Paths.get(""));
```

* 若目标路径已存在,那么复制或移动将失败,如果想要覆盖已有的目标路径,可以使用`REPLACE_EXISTING`选项,如果想要复制所有文件属性,可以使用`COPY_ATTRIBUTES`,也可以同时选择两个选项.

##### 创建文件和目录

```java
    		// 创建目录
        Files.createDirectory(Paths.get(""));
        Files.createDirectories(Paths.get(""));
				// 创建临时文件
        Files.createTempFile("prefix","suffix");

        // 创建文件创建文件
        Files.createFile(Paths.get(""));
```

* 若文件已存在了,这个创建文件的操作就会抛出异常.检查文件是否存在和创建文件是原子的.

##### 迭代目录中的文件

```java
 		 try (DirectoryStream<Path> entries = Files.newDirectoryStream(Paths.get(""))) {
            for (Path entry : entries) {
                // handle entry
            }
        }
```

* 若想要访问某个目录的所有子孙成员可以使用`walkFileTree`方法,并想起传递一个`FileVisitor`类型的对象,这个对象会得到下列通知:
  * 在遇到一个文件或目录时:`FileVisitResult visitFile(Path file, BasicFileAttributes attrs)`
  * 在一个目录被处理前: `FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) `
  * 在一个目录被处理后:`FileVisitResult postVisitDirectory(Path dir, IOException exc)`
  * 在试图访问文件或目录时发生错误,例如没有权限打开目录:`FileVisitResult visitFileFailed(Path file, IOException exc)`
  * 对于上述每种情况,都可以指定是否希望执行下面的操作:
    * 继续访问下一个文件: `FileVisitResult.CONTINUE`
    * 继续访问,但不再访问这个目录下的任何项: `FileVisitResult.SKIP_SUBTREE`
    * 继续访问,但不再访问这个文件的兄弟文件(和该文件在同一个目录下的文件):`FileVisitResult.`SKIP_SIBLINGS
    * 终止访问: `FileVisitResult.TERMINATE`

```java
 				Files.walkFileTree(Paths.get(""),new SimpleFileVisitor<Path>(){
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                if (attrs.isDirectory()){
                    System.out.println(file);
                }
                return FileVisitResult.CONTINUE;
            }
          
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                return super.preVisitDirectory(dir, attrs);
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                return super.postVisitDirectory(dir, exc);
            }

            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
                return FileVisitResult.CONTINUE;
            }
        });
```

### 内存文件映射

* 大多数操作系统都可以利用虚拟内存实现来将一个文件或文件的一部分"映射"到内存中.然后,这个文件就可以当做是内存数组一样地访问,这比传统文件操作要快得多.

* 在`java.nio`包使内存映射变得十分简单

  * 首先从文件中获得一个通道(channel),通道是用于磁盘文件的一种抽象,它使我们可以访问注入内存映射,文件加锁机制以及文件快速数据传递等操作系统特性.

    ```java
    	final FileChannel fileChannel = FileChannel.open(Paths.get(""), StandardOpenOption.READ);
    ```

  * 然后通过调用`FileChannel`类的`map`方法从这个通道中获得一个`ByteBuffer`.可以指定想要映射的文件区域与映射模式,支持的模式有三种:

    * `java.nio.channels.FileChannel.MapMode#READ_ONLY`: 所产生的缓冲区是只读的,任何对该缓冲区写入的尝试都会导致`ReadOnlyBufferException`异常;
    * `java.nio.channels.FileChannel.MapMode#READ_WRITE`:所产生的缓冲区是可写的,任何修改都会在某个时刻写会到文件中.注意,其他映射同一个文件的程序不能立即看到这些修改,多个程序同时进行文件映射的确切行为是依赖操作系统的;
    * `java.nio.channels.FileChannel.MapMode#PRIVATE`: 所产生的缓存区是可写的,但是任何修改对这个缓冲区来说都是私有的,不会传播到文件中.
    
  * 一但有了缓冲区,就可以使用`ByteBuffer`类和`Buffer`超类的方法读写数据了.
  
* 缓冲区支持顺序和随机数据访问,它有一个可以通过`get`和`put`操作类移动的位置.

  ```java
   				while (mappedByteBuffer.hasRemaining()) {
              final byte b = mappedByteBuffer.get();
          }
          
          for (int i = 0; i < mappedByteBuffer.limit(); i++) {
              final byte b = mappedByteBuffer.get(i);
          }
  ```

#### 缓冲区数据结构

* `Buffer`类是一个抽象类,它有很多具体子类,包括`ByteBuffer`,`CharBuffer`,`DoubleBuffer`,`IntBuffer`,`LongBuffer`和`ShortBuffer`.

  * `StringBuffer`类与这些缓冲区没有关系.

* 最常用的是`ByteBuffer`和`CharBuffer`,每个缓冲区都具有:

  * 一个容量,它永远不能改变.

  * 一个读写位置,下一个值将在此进行读写.

  * 一个界限,超过它进行读写是没有意义的.

  * 一个可选的标记,用于重复一个读入或写出操作.

  * 这些值满足下面的条件

    ```
    	0<=标记<=位置<=界限<=容量
    ```

* 使用缓冲区的主要目的是执行**"写,然后读入"**循环.假设有一个缓冲区,在一开始,它的位置是0,界限等于容量.不断地调用`put`将值添加到这个缓冲区中,当耗尽所有的数据或者写出的数据量达到容量大小时,就应该切换到读入操作了.

* 此时调用`filp`方法将界限设置到当前位置,并把位置复位到0.现在在`remaining`方法返回正数时(它返回的值是"界限-位置"),不断地调用`get`.在将所有缓冲区的内容读入之后,调用`clear`使缓冲区为下一次写循环做准备.`clear`方法将位置复位到0,并将界限复位到容量.

