---
layout: post
title: Java日志规范简约
tags: 
    - log 
    - 日志
category: 日志采集
description: 参考阿里的Java手册编写的Java日志规范简约
---

# Java日志规范简约

[TOC]

## 版本号

| 版本  | 日期       | 修订人 | 备注 |
| ----- | ---------- | ------ | ---- |
| 1.0.0 | 2018-07-10 | 沈兵   |      |

## 异常

1. 自定义公共异常模块[**trendy-commons-base**](http://172.20.15.8/trendy-commons/trendy-commons)，所有的自定义异常必须实现`ITrendytechException`接口，所有的运行时异常继承`UncheckedException`
2. 对外API的异常，尽量使用带自定义返回码的异常`UncheckedCodeException`，并根据异常码转换成用户可以理解的信息并反馈
3. 业务异常、或大部分受检异常，必须使用运行时异常，除非你是有经验的并决定这是一个受检异常，那么你可以自己抛出并提供try-catch处理
4. 在方法上注释上尽量写出方法内可能抛出的异常，及抛出的原因、解决方式，尤其是自定义的业务异常。
5. 明确定义的自定义异常名称
6. 方法出错，按具体情况决定返回错误码还是抛出异常，不要一味以为返回错误码或抛异常总是最好的
7. SpringWeb项目必须统一处理异常，并且返回统一的数据格式到前端
8. 异常捕获的日志，不要重复打印log，如果有统一的异常处理，那么仅在统一处理的地方打印日志
9. 异常的日志记录必须符合规范`logger.error("error msg", e)`，必须打印异常日志，并详细说明日志的发生原因，有必要的情况下必须记录方法参数、错误的运行结果
10. 必须清楚的认识`IllegalArgumentException`、IllegalStateException、NullPointException等基本异常，并且在符合条件的情况下使用
11. catch 时请分清稳定代码和非稳 定代码，稳定代码指的是无论如何不会出错的代码。   对于非稳定代码的 catch 尽可能进行区分 异常类型，再做对应的异常处理。 
12. 有 try 块放到了事务代码中，catch 异常后，如果需要回滚事务，一定要注意手动回滚事务。 
13. finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。 
14. 不能在 finally 块中使用 return，finally 块中的 return 返回后方法结束执行，不会再执行 try 块中的 return 语句。 
15. 方法的返回值可以为 null，不强制返回空集合，或者空对象等，必须添加注释充分 
16. 异常不要用来做流程控制，条件控制，因为异常的处理效率比条件分支低。 

## 日志

1. 公司提供统一的业务日志的切面框架[trendy-commons-log](http://172.20.15.8/trendy-commons/trendy-commons)
2. 一般情况下的异常日志必须用`logger.error("msg", e)`记录。msg必须记录有效的异常信息，e打印异常堆栈
3. 特殊情况下，有的异常被捕获后不处理，例如：用try-catch尝试返回`null`，那么也要打印`logger.warn("msg")`或`log.info("msg")`
4. Java异常的日志必须仅打印一次，重复打印会产生过多的冗余
5. 统一采用`SLF4J`做统一的日志操作框架，底层可以采用Logback或Log4J
6. SLF4J统一用`static final`修饰日志对象
7. Springboot建议采用默认的Logback框架
8. 所有的jar包中不推荐包含log4j.xml、log4j.properties、logback.xml文件，避免干扰实际的业务系统
9. 日志存储周期必须定义，防止写满服务器，常用1到3个月
10. 必须使用`SLF4J`的日志占位符拼接日志信息
11. 生产环境禁用debug日志
12. 可以使用 warn 日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从
13. 对`InvocationTargetException`,必须打印TargetException信息
14. 禁止敏感业务信息记录入日志文件，例如密码
15. 多业务的情况可以根据业务拆成多个日志文件
16. 公司提供统一的日志采集的配置文件，其他项目必须基于该配置文件进行修订
17. 日志文件的名称必须规范：日志等级.主机.项目名.业务名.log，示例：`info.app.log`，`info.app_name.business.log`，`info.app_name.business.date.log`，`info.host.app_name.business.log`