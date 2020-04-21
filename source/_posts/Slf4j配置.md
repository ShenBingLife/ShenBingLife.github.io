---
layout: post
title: Slf4j配置
tags: 
    - log
    - 日志
category: 日志
description: Slf4j桥接关系、输出格式。 
---

# Slf4j

[TOC]

## 功能依赖
![image](https://tva1.sinaimg.cn/large/007S2YVMgy1gdzit7wo76j30w00homz4.jpg)



## 桥接包：

<http://www.slf4j.org/legacy.html>

1，从Jakarta Commons Logging (JCL)迁移到slf4j： 用“jcl-over-slf4j.jar”替换掉“commons-logging.jar”即可

​     如果在maven中：1，明确的标记exclusion掉common-logging.jar。2，声明common-logging.jar的依赖范围为“provided”（在IED里显示不太好，注意顺序）。

2，很少见的特殊情况，想要把slf4j日志转移到JCL上：用slf4j-jcl.jar包

​     ——上述两个包不能一起用，否则死循环（这很好理解）

3，从log4j迁移到slf4j（居然直接针对log4j编码，太2了）：用“log4j-over-slf4j.jar”替换掉“log4j.jar”即可。

​     ——上述包不能与“slf4j-log4j12.jar”一起用，否则死循环（一样的道理）。

4，从java.util.logging（JUL）迁移到slf4j——jvm自己的类不允许随便替换，所以这里比较复杂(SLF4JBridgeHandler+jul-to-slf4j.jar)，而且有性能问题，很少用到，用的时候再研究。

​      ——上述包不能与“slf4j-jdk14.jar”一起用，否则死循环（一样的道理）。

## 桥接流程

### Logback日志，桥接Log4j、JCL、JUL

### Log4j日志，桥接JCL、JUL

### JUL日志，桥接JCL、Log4j

![image](https://tvax2.sinaimg.cn/large/007S2YVMgy1gdziqa13ujj31830v70xc.jpg)

## 使用方法

在maven中声明包的实现：

对于普通项目（并不是一个工具包）：

>* logback：
>
>​          声明对logback-classic-1.0.13.jar的依赖即可，会自动关联依赖slf4j-api-1.7.7.jar和logback-core-1.0.13.jar（显式声明这两个关联依赖也可以）
>
>* log4j：
>
>​          声明对slf4j-log4j12-1.7.7.jar的依赖即可，会自动关联依赖slf4j-api-1.7.7.jar和log4j-1.2.17.jar
>
>* jdk log：
>
>​          声明对slf4j-jdk14-1.7.7.jar的依赖即可，会自动关联依赖slf4j-api-1.7.7.jar



对于工具包项目：

> 仅仅依赖slf4j-api即可，让用户自己决定底层用什么实现。

二进制兼容性：

> 各个api绑定包，与下面的日志实现包，其版本必须对应。比如slf4j-api-1.7.7.jar应该对应slf4j-simple-1.7.7.jar.
>
>  而上层的应用程序，与其依赖的slf4j-api是各个版本都完全兼容的。

支持MDC（Mapped Diagnostic Context）

> 同时需要底层的日志包也支持，例如logback或者log4j。



## Logback Pattern

https://logback.qos.ch/manual/layouts.html

| %m   | 输出代码中指定的消息                                         |
| ---- | ------------------------------------------------------------ |
| %p   | 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL                |
| %r   | 输出自应用启动到输出该log信息耗费的毫秒数                    |
| %c   | 输出所属的类目，通常就是所在类的全名                         |
| %t   | 输出产生该日志事件的线程名                                   |
| %n   | 输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n”      |
| %d   | 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 |
| %l   | 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10) |

%c{length} /%lo{length} /logger{length} 打印logger名称，并控制长度，比如；%logger{5}   mainPackage.sub.sample.Bar  m.s.s.Bar

%C{length} /class{length} 打印出产生这条log的调用方的完整名字，并控制长度。这个方法效率不高，慎用。(目测要调用堆栈才能查到调用方的类名或者方法)

%contextName / %cn 打印出logger的上下文信息。在Configuration一节中我们曾经看到过这个的使用

%d{pattern} / %date{pattern} /%d{pattern, timezone} /%date{pattern, timezone} 打印出当前时间，通过pattern来控制日期格式，比如：%date{HH:mm:ss.SSS}    14:06:49.812

%F / %file 打印出产生这条log的java源文件，这个方法效率不高，慎用。

1. %caller{depth}
2. %caller{depthStart..depthEnd}
3. %caller{depth, evaluator-1, ... evaluator-n}
4. %caller{depthStart..depthEnd, evaluator-1, ... evaluator-n}

%L / %line 打印出产生这条log的源码行号

%m / %msg / %message 打印loggger方法传入的消息，比如.loggger.info("hello"),指代这里的“hello”

%M / %method 打印出产生这条log的方法名，效率不高

%n 换行

%p / %le /%level 打印出日志等级

%r / %relative 打印出自动App启动至今流逝的时间（单位：毫秒）

%t / %thread 打印出产生这条log的线程名字

%X{key:-defaultVal} / mdc{key:-defaultVal} 打印出MDC中定义的上下文

1. %ex{depth}
2. %exception{depth}
3. %throwable{depth}
4. %ex{depth, evaluator-1, ..., evaluator-n}
5. %exception{depth, evaluator-1, ..., evaluator-n}
6. %throwable{depth, evaluator-1, ..., evaluator-n}

1. %xEx{depth}
2. %xException{depth}
3. %xThrowable{depth}
4. %xEx{depth, evaluator-1, ..., evaluator-n}
5. %xException{depth, evaluator-1, ..., evaluator-n}
6. %xThrowable{depth, evaluator-1, ..., evaluator-n}

%nopex /%nopexception 忽略异常信息

%marker 打印marker信息，格式：parentName [ child1, child2 ]

%property{key} 引用之前定义的变量（或属性）

%replace(p){r, t} 举例说明：%replace(%msg){'\s', ''}，会把消息中的所有空格移除

 

 

 

 

 