---
layout: post
title: SLF4J
tags: 
    - log
    - 日志
category: git
description: SLF4J是用于日志记录系统的外观模块（facade），允许最终用户在部署时插入所需的日志记录系统。
---

# SLF4J

[TOC]

SLF4J是用于日志记录系统的外观模块（facade），允许最终用户在部署时插入所需的日志记录系统。

SLF4J有利于维护和各个类的日志处理方式统一

https://www.slf4j.org

## JCL与SLF4j

​	Jakarta Commons Logging (JCL)提供的是一个日志(Log)接口(interface)，同时兼顾轻量级和不依赖于具体的日志实现工具。 它提供给中间件/日志工具开发者一个简单的日志操作抽象，允许程序开发人员使用不同的具体日志实现工具。用户被假定已熟悉某种日志实现工具的更高级别的细节。JCL提供的接口，对其它一些日志工具，包括Log4J, Avalon LogKit, and JDK 1.4等，进行了简单的包装，此接口更接近于Log4J和LogKit的实现. 

### JCL的缺点：

**common-logging**通过动态查找的机制，在程序运行时自动找出真正使用的日志库。由于它使用了ClassLoader寻找和载入底层的日志库， 导致了象OSGI这样的框架无法正常工作，因为OSGI的不同的插件使用自己的ClassLoader。 OSGI的这种机制保证了插件互相独立，然而却使Apache Common-Logging无法工作。

[commons-logging classloader pain ](http://wrschneider.blogspot.com/2005/06/commons-logging-classloader-pain_18.html)

**slf4j**在编译时静态绑定真正的Log库,因此可以再OSGI中使用。另外，SLF4J 支持参数化的log字符串，提高性能和代码简洁。



## SLF4J绑定

https://stackoverflow.com/questions/42186919/how-slf4j-jpa-jax-rs-find-their-implementation

https://v4forums.wordpress.com/2008/12/27/slf4j-vs-jcl-dynamic-binding-vs-static-binding/

## Logger是否声明为静态字段？

 **we no longer recommend one approach over the other.**

Here is a summary of the pros and cons of each approach.

| Advantages for declaring loggers as static                   | Disadvantages for declaring loggers as static                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| common and well-established idiomless CPU overhead: loggers are retrieved and assigned only once, at hosting class initializationless memory overhead: logger declaration will consume one reference per class | For libraries shared between applications, not possible to take advantage of repository selectors. It should be noted that if the SLF4J binding and the underlying API ships with each application (not shared between applications), then each application will still have its own logging environment.not IOC-friendly |
| Advantages for declaring loggers as instance variables       | Disadvantages for declaring loggers as instance variables    |
| Possible to take advantage of repository selectors even for libraries shared between applications. However, repository selectors only work if the underlying logging system is logback-classic. Repository selectors do not work for the SLF4J+log4j combination.IOC-friendly | Less common idiom than declaring loggers as static variableshigher CPU overhead: loggers are retrieved and assigned for each instance of the hosting classhigher memory overhead: logger declaration will consume one reference per instance of the hosting class |

静态字段：内存和CPU消耗更少，但是IOC不好。

非静态字段：消耗更多内存和CPU，但是IOC友好。

从当前角度来看，更多的选择静态字段。

## 推荐声明 SLF4JLogger

The following is the recommended logger declaration idiom. For reasons [explained above](https://www.slf4j.org/faq.html#declared_static), it is left to the user to determine whether loggers are declared as static variables or not.

```
package some.package;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
      
public class MyClass {
  final (static) Logger logger = LoggerFactory.getLogger(MyClass.class);
  ... etc
}
```

Unfortunately, given that the name of the hosting class is part of the logger declaration, the above logger declaration idiom is *not* resistant to cut-and-pasting between classes.

Alternatively, you can use `MethodHandles.lookup()` introduced in JDK 7 to pass the caller class.

```
package some.package;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.lang.invoke.MethodHandles;
      
public class MyClass {
  final static Logger logger = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
  ... etc
}
```

This pattern can be cut and pasted across classes.

## MDC

　MDC（Mapped Diagnostic Context，映射调试上下文）是 log4j 和 logback 提供的一种方便在多线程条件下记录日志的功能。 

MDC 可以看成是一个与当前线程绑定的哈希表，可以往其中添加键值对。MDC 中包含的内容可以被同一线程中执行的代码所访问。当前线程的子线程会继承其父线程中的 MDC 的内容。当需要记录日志时，只需要从 MDC 中获取所需的信息即可。 

示例：

```
@Around(value = "execution(* com.xx.xx.facade.impl.*.*(..))", argNames="pjp")
  public Object validator(ProceedingJoinPoint pjp) throws Throwable {
      try {
          String traceId = TraceUtils.begin();
          MDC.put("mdc_trace_id", traceId);
          Object obj = pjp.proceed(args);
          return obj;
      } catch(Throwable e) {
          //TODO 处理错误
      } finally {
          TraceUtils.endTrace();
      }
  }   
```

代码通过AOP记录了每次请求的traceId并使用变量"mdc_trace_id"记录，在日志配置文件里需要设置变量才能将"mdc_trace_id"输出到日志文件中。

```xml
<appender name="ALL" class="ch.qos.logback.core.rolling.RollingFileAppender">
    ...
      <encoder charset="UTF-8">
          <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - traceId:[%X{mdc_trace_id}] - %msg%n</pattern>
      </encoder>
  </appender>
```



