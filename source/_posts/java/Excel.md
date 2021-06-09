---
title: Excel处理遇到的问题
date: 2021-06-04 15:25:54
index_img: /img/cover/Java.jpg
cover: /img/cover/Java.jpg
tags: 
- Excel
categories:
- Java
---

#### Excel导出设置标题格填充颜色setFillForegroundColor无效问题

```java
cellStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
加上下面的属性就可以了
cellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
```

