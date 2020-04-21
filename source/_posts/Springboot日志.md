---
layout: post
title: Springboot 日志
tags: 
    - log
    - 日志
    - springboot
category: 日志
description: Springboot 日志的各种配置
---

# Springboot 日志

[TOC]

## 默认日志 `Logback`：

默认情况下，Spring Boot会用Logback来记录日志，并用**INFO**级别输出到控制台。在运行应用程序和其他例子时，你应该已经看到很多INFO级别的日志了。

`spring-boot-starter`其中包含了 `spring-boot-starter-logging`，该依赖内容就是 `Spring Boot` 默认的日志框架 `logback`。 

Spring Boot默认配置只会输出到控制台，并不会记录到文件中 。

## 配置文件

application.properties

- `logging.file`，设置文件，可以是绝对路径，也可以是相对路径。如：`logging.file=my.log`
- `logging.path`，设置目录，会在该目录下创建`spring.log`文件，并写入日志内容，如：`logging.path=/var/log`
- 二者不能同时使用，如若同时使用，则只有`logging.file`生效 默认情况下，日志文件的大小达到`10MB`时会切分一次，产生新的日志文件，默认级别为：`ERROR、WARN、INFO` 
- `logging.level`：日志级别控制前缀，*为包名或Logger名 
- `logging.level.root`：配置默认的日志级别
- logging.pattern.console：定义输出到控制台的样式（不支持JDK Logger）
- logging.pattern.file：定义输出到文件的样式（不支持JDK Logger）

### 多彩输出

如果你的终端支持ANSI，设置彩色输出会让日志更具可读性。通过在`application.properties`中设置`spring.output.ansi.enabled`参数来支持。

- NEVER：禁用ANSI-colored输出（默认项）
- DETECT：会检查终端是否支持ANSI，是的话就采用彩色输出（推荐项）
- ALWAYS：总是使用ANSI-colored格式输出，若终端不支持的时候，会有很多干扰信息，不推荐使用

## 自定义日志配置

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：

- Logback：`logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy` 

- Log4j：`log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml` 

- Log4j2：`log4j2-spring.xml, log4j2.xml` 

- JDK (Java Util Logging)：`logging.properties`

- 自定义日志配置文件位置

  ```
  logging.config=classpath:logging-config.xml
  ```

 `Spring Boot`官方推荐优先使用带有`-spring`的文件名作为你的日志配置（如使用`logback-spring.xml`，而不是`logback.xml`），命名为`logback-spring.xml`的日志配置文件，`spring boot`可以为它添加一些`spring boot`特有的配置项。

 

##  常用logbck-spring.xml配置

### **根节点\<configuration>包含的属性**

- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

###  contextName

每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改,可以通过%contextName来打印日志上下文名称。

```
<contextName>logback</contextName>
```

### property

用来定义变量值的标签， 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

```
<property name="log.path" value="/Users/tengjun/Documents/log" />
```

 

###  appender

appender用来格式化日志输出节点，有俩个属性name和class，class用来指定哪种输出策略，常用就是控制台输出策略和文件输出策略。

\#####控制台输出ConsoleAppender：

```
<!--输出到控制台-->
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>ERROR</level>
    </filter>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

`<encoder>`表示对日志进行编码：

- `%d{HH: mm:ss.SSS}`——日志输出时间
- `%thread`——输出日志的进程名字，这在Web应用以及异步任务处理中很有用
- `%-5level`——日志级别，并且使用5个字符靠左对齐
- `%logger{36}`——日志输出者的名字
- `%msg`——日志消息
- `%n`——平台的换行符

ThresholdFilter为系统定义的拦截器，例如我们用ThresholdFilter来过滤掉ERROR级别以下的日志不输出到文件中。如果不用记得注释掉，不然你控制台会发现没日志~

####  RollingFileAppender

另一种常见的日志输出到文件，随着应用的运行时间越来越长，日志也会增长的越来越多，将他们输出到同一个文件并非一个好办法。`RollingFileAppender`用于切分文件日志：

```
<!--输出到文件-->
<appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>${log.path}/logback.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>30</maxHistory>
		<totalSizeCap>1GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

其中重要的是`rollingPolicy`的定义，上例中`<fileNamePattern>${log.path}/logback.%d{yyyy-MM-dd}.log</fileNamePattern>`定义了日志的切分方式——把每一天的日志归档到一个文件中，`<maxHistory>30</maxHistory>`表示只保留最近30天的日志，以防止日志填满整个磁盘空间。同理，可以使用`%d{yyyy-MM-dd_HH-mm}`来定义精确到分的日志切分方式。`<totalSizeCap>1GB</totalSizeCap>`用来指定日志文件的上限大小，例如设置为1GB的话，那么到了这个值，就会删除旧的日志。

补:如果你想把日志直接放到当前项目下，把`${log.path}/`去掉即可。

logback 每天生成和大小生成冲突的问题可以看这个解答：[传送门](http://blog.csdn.net/wujianmin577/article/details/68922545) 

##  多环境日志输出

据不同环境（prod:生产环境，test:测试环境，dev:开发环境）来定义不同的日志输出，在 logback-spring.xml中使用 springProfile 节点来定义，方法如下：

> 文件名称不是logback.xml，想使用spring扩展profile支持，要以logback-spring.xml命名

```
<!-- 测试环境+开发环境. 多个使用逗号隔开. -->
<springProfile name="test,dev">
    <logger name="com.dudu.controller" level="info" />
</springProfile>
<!-- 生产环境. -->
<springProfile name="prod">
    <logger name="com.dudu.controller" level="ERROR" />
</springProfile>
```

可以启动服务的时候指定 profile （如不指定使用默认），如指定prod 的方式为：

`java -jar xxx.jar –spring.profiles.active=prod `

## 从logback中获取配置文件中的信息

 配置文件要以logback-spring.xml命名

```
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```



