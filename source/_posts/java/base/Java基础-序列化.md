---
title: Java基础-序列化
date: 2021-07-05 22:07:30
cover: /img/cover/Java.jpg
tags:
- 序列化
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

* [面试官：Java序列化为什么要实现Serializable接口？我懵了](https://www.cnblogs.com/chengxy-nds/p/12347324.html)

### 什么是序列化

* **序列化**:  `Java`中的序列化机制能够将一个实例化对象信息写入到一个字节流中(**只序列号对象的属性值,而不会去序列化方法**),序列化后的对象可用于网络传输,或者持久化到数据库,磁盘中.
* **反序列化**: 需要对象的时候,在通过字节流中的信息来重构一个相同的对象.

#### Java中具体实现

* Java中要使一个类可以序列化,实现`java.io.Serializable`接口是最简单的.

  ```java
  public class User implements Serializable {
  
      private static final long serialVersionUID = 1L;
  }
  ```

  ```java
  @Data
  public class User implements Serializable {
  
      private static final long serialVersionUID = 1L;
  
      private String name;
  
      private String age;
  }
  ```

  ```java
  @Slf4j
  public class SerializeTest {
  
      public static void main(String[] args) throws Exception {
          User user = new User();
          user.setName("HoleLin");
          user.setAge("18");
  
          serialize(user);
          log.info("Java序列化前的结果:{} ", user);
  
          User deserializeUser = deserialize();
          log.info("Java反序列化的结果:{} ", deserializeUser);
      }
  
      /**
       * 序列化
       */
      private static void serialize(User user) throws Exception {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:\\UserSerialize.txt")));
          oos.writeObject(user);
          oos.close();
      }
  
      /**
       * 反序列化
       */
      private static User deserialize() throws Exception {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:\\UserSerialize.txt")));
          return (User) ois.readObject();
      }
  }
  ```


#### 为什么要实现`Serializable`接口

  * 打开`writeObject`方法的源码看一下，发现方法中有这么一个逻辑，当要写入的对象是`String`、`Array`、`Enum`、`Serializable`类型的对象则可以正常序列化，否则会抛出`NotSerializableException`异常。

  ```java
      /**
       * Underlying writeObject/writeUnshared implementation.
       */
      private void writeObject0(Object obj, boolean unshared)
          throws IOException
      {
          boolean oldMode = bout.setBlockDataMode(false);
          depth++;
          try {
              // handle previously written and non-replaceable objects
              int h;
              if ((obj = subs.lookup(obj)) == null) {
                  writeNull();
                  return;
              } else if (!unshared && (h = handles.lookup(obj)) != -1) {
                  writeHandle(h);
                  return;
              } else if (obj instanceof Class) {
                  writeClass((Class) obj, unshared);
                  return;
              } else if (obj instanceof ObjectStreamClass) {
                  writeClassDesc((ObjectStreamClass) obj, unshared);
                  return;
              }
  
              // check for replacement object
              Object orig = obj;
              Class<?> cl = obj.getClass();
              ObjectStreamClass desc;
              for (;;) {
                  // REMIND: skip this check for strings/arrays?
                  Class<?> repCl;
                  desc = ObjectStreamClass.lookup(cl, true);
                  if (!desc.hasWriteReplaceMethod() ||
                      (obj = desc.invokeWriteReplace(obj)) == null ||
                      (repCl = obj.getClass()) == cl)
                  {
                      break;
                  }
                  cl = repCl;
              }
              if (enableReplace) {
                  Object rep = replaceObject(obj);
                  if (rep != obj && rep != null) {
                      cl = rep.getClass();
                      desc = ObjectStreamClass.lookup(cl, true);
                  }
                  obj = rep;
              }
  
              // if object replaced, run through original checks a second time
              if (obj != orig) {
                  subs.assign(orig, obj);
                  if (obj == null) {
                      writeNull();
                      return;
                  } else if (!unshared && (h = handles.lookup(obj)) != -1) {
                      writeHandle(h);
                      return;
                  } else if (obj instanceof Class) {
                      writeClass((Class) obj, unshared);
                      return;
                  } else if (obj instanceof ObjectStreamClass) {
                      writeClassDesc((ObjectStreamClass) obj, unshared);
                      return;
                  }
              }
  
              // remaining cases
              if (obj instanceof String) {
                  writeString((String) obj, unshared);
              } else if (cl.isArray()) {
                  writeArray(obj, desc, unshared);
              } else if (obj instanceof Enum) {
                  writeEnum((Enum<?>) obj, desc, unshared);
              } else if (obj instanceof Serializable) {
                  writeOrdinaryObject(obj, desc, unshared);
              } else {
                  if (extendedDebugInfo) {
                      throw new NotSerializableException(
                          cl.getName() + "\n" + debugInfoStack.toString());
                  } else {
                      throw new NotSerializableException(cl.getName());
                  }
              }
          } finally {
              depth--;
              bout.setBlockDataMode(oldMode);
          }
      }
  ```

#### 为什么要显示指定`serialVersionUID`

* 因为序列化对象时,如果不显示的设置`serialVersionUID`,Java在序列化会根据对象属性自动的生成一个`serialVersionUID`,在进行存储或用作网络传输.
* 在反序列化时,会根据对象属性自动再生成一个新的`serialVersionUID`,和序列化是生成的`serialVersionUID`进行对比,两个`serialVersionUID`相同则反序列化成功,否则就会抛出异常.
* 故当显示的设置`serialVersionUID`后,Java序列化和反序列化对象时,生成的`serialVersionUID`都是我们设定的`serialVersionUID`,这样就保证了反序列化的成功.

