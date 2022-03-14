---
title: Spring(十二)-Validation校验
mermaid: true
date: 2021-06-27 21:02:16
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- Validation 
categories:
- Spring
- Spring Framework
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

### Spring校验使用场景

* Spring常规校验(`Validator`)
* Spring数据绑定(`DataBinder`)
* Spring Web参数绑定(`WebDataBinder`)
* `Spring Web MVC`/`Spring WebFlux`处理方法参数校验

### `Validator`接口设计

* 接口职责

  * Spring内部校验器接口,通过编程的方式校验目标对象

* 核心方法

  * `support(Class)`: 校验目标类能否校验;
  * `validate(Object,Errors)`: 校验目标对象,并将校验失败的内容输出至`Errors`对象

* 配套组件

  * 错误收集器: `org.springframework.validation.Errors `

  * `Validator` 工具类：`org.springframework.validation.ValidationUtils `

### Errors接口设计

* 接口职责
  * 数据绑定和校验错误收集接口，与 Java Bean 和其属性有强关联性
* 核心方法

     * `reject` 方法（重载）：收集错误文案

     * `rejectValue` 方法（重载）：收集对象字段中的错误文案
* 配套组件
     * Java Bean 错误描述：`org.springframework.validation.ObjectError`
     * Java Bean 属性错误描述：`org.springframework.validation.FieldError`  

 ### Errors 文案来源  

* Errors 文案生成步骤
  * 选择 `Errors`实现（如：`org.springframework.validation.BeanPropertyBindingResult`）
  * 调用 `reject` 或 `rejectValue` 方法
  * 获取 `Errors` 对象中 `ObjectError` 或 `FieldError`
  * 将 `ObjectError` 或 `FieldError` 中的 `code` 和 `args`，关联 `MessageSource` 实现（如:`ResourceBundleMessageSource`)

### 自定义 `Validator ` 

* 实现 `org.springframework.validation.Validator` 接口
  * 实现 `supports` 方法
  * 实现 `validate` 方法
    * 通过 `Errors` 对象收集错误
      * `ObjectError`：对象（Bean）错误：
      * `FieldError`：对象（Bean）属性（Property）错误
    * 通过 `ObjectError` 和 `FieldError` 关联 `MessageSource` 实现获取最终文案  

### `Validator` 的救赎  

* Bean Validation 与 `Validator` 适配
  * 核心组件 - `org.springframework.validation.beanvalidation.LocalValidatorFactoryBean`
  * 依赖 `Bean Validation` - `JSR-303 or JSR-349 provider`
  * Bean 方法参数校验 - `org.springframework.validation.beanvalidation.MethodValidationPostProcessor`  

### Hibernate Validator

#### 注解

| 注解名                                       | 作用                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| `@Valid`                                     | 被注解的元素是一个对象,需要检查此对象的所有字段              |
| `@Null`                                      | 被注解的元素必须为null                                       |
| `@NotNull`                                   | 被注解的元素必须不为null                                     |
| `@AssertTrue`                                | 被注解的元素必须为true                                       |
| `@AsserFalse`                                | 被注解的元素必须为false                                      |
| `@Min(value)`                                | 被注解的元素必须是一个数字,其值必须大于等于指定的最小值      |
| `@Max(value)`                                | 被注解的元素必须是一个数字,其值必须小于等于指定的最大值      |
| `@DecimalMin(value)`                         | 被注解的元素必须是一个数字,其值必须小于等于指定的最大值      |
| `@DecimalMax(value)`                         | 被注解的元素必须是一个数字,其值必须小于等于指定的最大值      |
| `@Size(max,min)`                             | 被注解的元素的大小必须在指定的范围内                         |
| `@Digits(integer,fraction)`                  | 被注解的元素必须是一个数字,其值必须在可接受的范围内          |
| `@Past`                                      | 被注解的元素必须是一个过去的日期                             |
| `@Future`                                    | 被注解的元素必须是一个将来的日期                             |
| `@Pattern(value)`                            | 被注解的元素必须符合指定的正则表达式                         |
| `@Email`                                     | 被注解的元素必须是电子邮箱地址                               |
| `@Length(min=,max=)`                         | 被注解的字符串的大小必须在指定的范围内                       |
| `@NotEmpty`                                  | 被注解的元素必须非空                                         |
| `@Range(min=,max=)`                          | 被注解的元素必须在可接受的范围内                             |
| `@NotBlank`                                  | 被注解的元素必须非空                                         |
| `@URL(protocol=,host=,port=,regexp=,flags=)` | 被注解的元素必须是一个有效的URL                              |
| `@CreditCardNumber`                          | 被注解的字符串必须通过Luhn校验算法,银行卡,信用卡等号码一般都用Luhn计算合法性 |
| `@ScriptAssert(lang=,script=,alias=)`        | 要有Java Scripting API                                       |
| `@SafeHtml(whitelistType,additionTags=)`     | classpath中要有jsoup包                                       |

* `@NotNull`,`@NotEmpty`,`@NotBlank`三个注解的区别
  * `@NotNull`:任何对象的value不能为null.
  * `@NotEmpty`:集合对象元素不为0,即集合不为空,也可以用于字符串不为null.
  * `@NotBlank`: 只能用于字符串不为null,并且字符串`trim()`以后length要大于0.
