---
title: Java基础-BigDecimal使用
date: 2021-07-10 22:30:38
cover: /img/cover/Java.jpg
tags:
- BigDecimal
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

#### `BigDecimal`概述

* `BigDecimal`是Java在`java.match`包中提供的API类,主要用来对超过16位有效位的数进行精确的运算.
* 双精度浮点型变量`double`可以处理16位有效数,但在实际引用中,可能需要对更大或者更小的数进行运算和处理;
* 一般情况对于那些不需要精确计算精度的数字,可以直接使用`Float`和`Double`来处理,但是使用`Float.valueOf(String)`和`Float.valueOf(String)`会丢失经,所以面对需要精确计算的结果,则必须使用`BigDecimal`类来操作;
* `BigDecimal`所创建的是对象,所以不能使用传统的`+,-,*,/`等运算符直接对对象进行数学运算,而必须调用其对应的方法,方法的参数也必须是`BigDecimal`的对象.

#### `BigDecimal`常用的构造函数

* `BigDecimal(int)`: 创建一个具有参数所指定整数值的对象

* `BigDecimal(double)`: 创建一个具有参数所指定双精度值的对象

* `BigDecimal(long)`: 创建一个具有参数所指定长整型数值的对象

* `BigDecimal(String)`: 创建一个具有参数所指定以字符串表示的数值的对象

###### 使用中存在的问题

```java
@Slf4j
public class BigDecimalTest {
    public static void main(String[] args) {
        double doubleValue = 0.1D;
        float floatValue = 0.1F;
        String stringValue="0.1";

        BigDecimal a = new BigDecimal(1);
        BigDecimal b = new BigDecimal(stringValue);
        BigDecimal c = new BigDecimal(doubleValue);
        BigDecimal d = new BigDecimal(floatValue);
        BigDecimal newC = new BigDecimal(Double.toString(doubleValue));
        BigDecimal newD = new BigDecimal(Float.toString(floatValue));
        log.info("a value is : {}", a);
        log.info("b value is : {}", b);
        log.info("c value is : {}", c);
        log.info("d value is : {}", d);
        log.info("newC value is : {}", newC);
        log.info("newD value is : {}", newD);
    }
}
// 输出
22:50:15.629 [main] INFO com.holelin.sundry.demo.BigDecimalTest - a value is : 1
22:50:15.632 [main] INFO com.holelin.sundry.demo.BigDecimalTest - b value is : 0.1
22:50:15.632 [main] INFO com.holelin.sundry.demo.BigDecimalTest - c value is : 0.1000000000000000055511151231257827021181583404541015625
22:50:15.633 [main] INFO com.holelin.sundry.demo.BigDecimalTest - d value is : 0.100000001490116119384765625
22:50:15.633 [main] INFO com.holelin.sundry.demo.BigDecimalTest - newC value is : 0.1
22:50:15.633 [main] INFO com.holelin.sundry.demo.BigDecimalTest - newD value is : 0.1
```

#### `BigDecimal`常用方法

* `add(BigDecimal)`: `BigDecimal`对象中的值相加,返回`BigDecimal`对象

* `subtract(BigDecimal)`: `BigDecimal`对象中的值相减,返回`BigDecimal`对象

* `multiply(BigDecimal)`: `BigDecimal`对象中的值相乘,返回`BigDecimal`对象

* `divide(BigDecimal)`: `BigDecimal`对象中的值相除,返回`BigDecimal`对象

* `toString()`: `BigDecimal`对象中的值转换为字符串

* `doubleValue()`: `BigDecimal`对象中的值转换为双精度数

* `floatValue`: `BigDecimal`对象中的值转换为单精度数

* `longValue`: `BigDecimal`对象中的值转换为长整型数

* `intValue`: `BigDecimal`对象中的值转换为整数

* `compareTo(BigDecimal)`: 两个`BigDecimal`对象中的值比较大小: 

  * `int result = bigDecimal1.compareTo(bigDecimal2)`
    * `result = -1`: 表示`bigDecimal1`小于`bigDecimal2`
    * `result = 0`: 表示`bigDecimal1`等于`bigDecimal2`
    * `result = 1`: 表示`bigDecimal1`大于`bigDecimal2`

* 格式化

  * 可以使用`NumberFormat`类的`forma`方法对`BigDecimal`进行格式化

    ```
            // 货币格式化
            NumberFormat currency = NumberFormat.getCurrencyInstance();
            // 百分比格式化
            NumberFormat percent = NumberFormat.getPercentInstance();
            // 设置百分比小数点最多3位
            percent.setMaximumFractionDigits(3);
            NumberFormat number = NumberFormat.getNumberInstance();
            BigDecimal principal = new BigDecimal("50000.9875");
    
            BigDecimal loanAmount = new BigDecimal("20210.071");
            BigDecimal interestRate = new BigDecimal("0.008");
            BigDecimal interest = loanAmount.multiply(interestRate);
            log.info("贷款金额: {}",currency.format(loanAmount));
            log.info("利率: {}",percent.format(interestRate));
            log.info("利息: {}",currency.format(interest));
            log.info("本金: {}",number.format(principal));
    // 输出
    23:05:27.575 [main] INFO com.holelin.sundry.demo.BigDecimalTest - 贷款金额: ￥20,210.07
    23:05:27.575 [main] INFO com.holelin.sundry.demo.BigDecimalTest - 利率: 0.8%
    23:05:27.575 [main] INFO com.holelin.sundry.demo.BigDecimalTest - 利息: ￥161.68
    23:05:27.575 [main] INFO com.holelin.sundry.demo.BigDecimalTest - 本金: 50,000.988
    ```

#### `BigDecimal`常见异常

```
        BigDecimal testNum = new BigDecimal("10");
        BigDecimal three = new BigDecimal("3");
        BigDecimal divide = testNum.divide(three);
// 异常
java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
```

* 通过`BigDecimal#divide`方法进行除法不整除,出现循环小数是,就会抛出上方的异常
* 解决方法: 使用`divide`方法时设置精确的小数点: `divide(bigDecimal,2)`

  

