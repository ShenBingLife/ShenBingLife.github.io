---
layout: post
title: Logback
tags: 
    - log 
    - 日志
category: 日志采集
description: Logback基本知识
---

# Logback

[TOC]

## 结构

LogBack分为3个组件，logback-core, logback-classic 和 logback-access。 

* logback-core提供了LogBack的核心功能，是另外两个组件的基础。 

* logback-classic则实现了Slf4j的API，所以当想配合Slf4j使用时，则需要引入这个包。

* logback-access是为了集成Servlet环境而准备的，可提供HTTP-access的日志接口 

## 日志级别

**OFF**、 **FATAL**、 **ERROR**、 **WARN**、 **INFO**、 **DEBUG**、 **ALL** 

## 核心对象

Logback整体流程：Logger 产生日志信息；Layout修饰这条msg的显示格式；Filter过滤显示的内容；Appender具体的显示 。

### Logger 

日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别 

### Appender

用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、 PostreSQL、Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 

### Layout

修饰日志的格式

### logger context

各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

### Filter

Logback的过滤器基于三值逻辑（ternary logic），允许把它们组装或成链，从而组成任意的复合过滤策略。过滤器很大程度上受到Linux的iptables启发。这里的所谓三值逻辑是说，过滤器的返回值只能是ACCEPT、DENY和NEUTRAL的其中一个。

如果返回DENY，那么记录事件立即被抛弃，不再经过剩余过滤器；

如果返回NEUTRAL，那么有序列表里的下一个过滤器会接着处理记录事件；

如果返回ACCEPT，那么记录事件被立即处理，不再经过剩余过滤器。

https://blog.csdn.net/d8111/article/details/45249555

### Marker

## 配置

如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG。 

https://www.cnblogs.com/reason-cai/p/6763108.html

https://blog.csdn.net/yingxiake/article/details/51274426

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
<property name="LOG_HOME" value="/home" />
<!-- 控制台输出 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
</encoder>
</appender>
<!-- 按照每天生成日志文件 -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
<!--日志文件输出的文件名-->
<FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
<!--日志文件保留天数-->
<MaxHistory>30</MaxHistory>
</rollingPolicy>
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
</encoder>
<!--日志文件最大的大小-->
<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
<MaxFileSize>10MB</MaxFileSize>
</triggeringPolicy>
</appender>

<!-- 日志输出级别 -->
<root level="INFO">
<appender-ref ref="STDOUT" />
</root>
</configuration>
```



