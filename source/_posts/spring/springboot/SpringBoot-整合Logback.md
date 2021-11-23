---
title: SpringBoot-整合Logback
date: 2021-11-23 17:07:54
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 日志
categories:
- SpringBoot
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

* [SpringBoot配合logback达到日志切割管理通用配置](https://blog.csdn.net/mojiewangday/article/details/107538981)
* [Logback and Spring Boot's new springProperty lookup mechanism not working](https://stackoverflow.com/questions/33505624/logback-and-spring-boots-new-springproperty-lookup-mechanism-not-working)

### logback配置

* 配置文件名称: `logback-spring.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">

    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%thread]){faint} %clr(%-40.40logger{39}){cyan} %clr([%L]) %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

    <!-- 定义日志文件的存储地址，勿在logback配置文件中使用相对路径.通过springProperty可以在properties文件设置相关path,level,name信息
    application.properties中定义的变量，在logback的xml文件中无法直接读取，必须要增加springProperty属性中转一下，source属性里放的值是application.properties中定义的
    -->
    <springProperty name="log.path" source="logging.file.path" />
    <springProperty name="log.name" source="logging.file.name" />
    <springProperty name="log.level" source="logging.logback.level" />

    <!-- 定义日志文件名格式化,定义到哪个维度则按哪个维度切割日志，只支持每周，每天，每个小时，每分钟等创建一个文件 yyyy-MM-dd-HH-mm 分钟维度 -->
    <property name="log.timeFormat" value="yyyy-MM-dd" />
    <!-- 定义日志文件的输出格式。%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度，%logger{50} 表示logger名字最长50个字符，否则按照句点分割。%msg：日志消息，%n是换行符 -->
    <property name="log.pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" />
    <!-- 定义日志文件保留天数，单位和log.timeFormat定义的最小维度保持一致，上面定义到天，则单位默认为天 -->
    <property name="log.maxHistory" value="7" />
    <!-- 定义日志文件最大限制，超过限制则切割日志 -->
    <property name="log.maxFileSize" value="100MB" />

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件，并按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/${log.name}.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>${log.level}</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--    滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动
                    %i：当文件大小超过maxFileSize时，按照i进行文件滚动 为了保证不重复
            -->
            <FileNamePattern>${log.path}/${log.name}.%d{${log.timeFormat}}-%i.log</FileNamePattern>
            <MaxHistory>${log.maxHistory}</MaxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>${log.maxFileSize}</MaxFileSize><!-- 日志文件分割的条件为日志文件大小达到maxFileSize -->
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <!-- 按照每天生成日志文件。仅记录错误日志 -->
    <appender name="FILE-ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/error/${log.name}_error.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${log.path}/error/${log.name}_error.%d{${log.timeFormat}}.%i.log</FileNamePattern>
            <MaxHistory>${log.maxHistory}</MaxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>${log.maxFileSize}</MaxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <!-- 日志输出级别，root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
        要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
        <appender-ref ref="FILE-ERROR" />
    </root>
</configuration>
```

* SpringBoot `bootstrap.yml`配置

  ```yml
  logging:
    level:
      logback: info
    file:
      path: logs/xxx
      name: xxxx
  ```

  
