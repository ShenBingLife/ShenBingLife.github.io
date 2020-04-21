---
layout: post
title: 优先于log4j进行logback的原因
tags: 
    - log
    - 日志
category: git
description: Logback对log4j进行了大量改进，无论大小。 它们太多了，无法详尽地列举。 然而，这里是从log4j切换到logback的原因的非详尽列表。 请记住，logback在概念上与log4j非常相似，因为两个项目都是由同一个开发人员创建的。 
---



## 优先于log4j进行logback的原因

Logback对log4j进行了大量改进，无论大小。 它们太多了，无法详尽地列举。 然而，这里是从log4j切换到logback的原因的非详尽列表。 请记住，**logback在概念上与log4j非常相似**，因为两个项目都是由同一个开发人员创建的。 如果您已熟悉log4j，您将很快感到宾至如归。 如果你喜欢log4j，你可能会喜欢logback。

### 更快的实施

基于我们之前关于log4j的工作，已经重写了logback内部，以便在某些关键执行路径上执行大约十倍的速度。 不仅logback组件更快，而且内存占用更少。

### 大量的测试

Logback包含了在几年和无数小时工作中开发的大量测试。 虽然log4j也经过测试，但logback将测试带到了完全不同的水平。 在我们看来，这是优先于log4j进行logback的最重要原因。 您希望您的日志框架即使在不利条件下也能够坚如磐石且可靠。

### logback-classic本身就说SLF4J

由于logback-classic中的`Logger`类本身实现了SLF4J API，因此在调用带有logback-classic作为底层实现的SLF4J记录器时，会产生零开销。 此外，由于logback-classic强烈鼓励使用SLF4J作为其客户端API，如果需要切换到log4j或jul，可以通过将一个jar文件替换为另一个来实现。 您无需通过SLF4J API触摸代码记录。 这可以大大减少切换日志框架所涉及的工作。

### 广泛的文档

Logback随附详细且不断更新的文档。

### XML或Groovy中的配置文件

配置logback的传统方法是通过XML文件。 文档中的大多数示例都使用此XML语法。 但是，从logback版本0.9.22开始，也支持[用Groovy编写的配置文件](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/groovy.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhhBWBsoeVYtx0g7jfK3fKP6d6GtYw) 。 与XML相比，Groovy风格的配置更直观，更一致，并且语法更短。

还有一个[工具可以自动将logback.xml文件迁移到logback.groovy](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=http://logback.qos.ch/translator/asGroovy.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhidtWK-Nyj80UQRSXcZmCWOCnUWNQ) 。

### 自动重新加载配置文件

Logback-classic可以[在修改后自动重新加载其配置文件](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/configuration.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhgRA0DKswZ_dsz-GTjGf9lqf_6q1g#autoScan) 。 扫描过程快速，无争用，并且动态扩展到数百个线程上每秒数百万次调用。 它在应用程序服务器中也能很好地运行，更常见的是在JEE环境中，因为它不涉及创建单独的扫描线程。

### 从I / O故障中顺利恢复

Logback的`FileAppender`及其所有子类（包括`RollingFileAppender` ）可以从I / O故障中正常恢复。 因此，如果文件服务器暂时失败，则不再需要重新启动应用程序以使日志记录再次运行。 一旦文件服务器恢复，相关的logback appender将从之前的错误状态透明地快速恢复。

### 自动删除旧的日志存档

通过设置[TimeBasedRollingPolicy](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/appenders.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjeKCV5-S5P_a0hUMWGkus6yEudeA#TimeBasedRollingPolicy)或[SizeAndTimeBasedFNATP](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/appenders.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjeKCV5-S5P_a0hUMWGkus6yEudeA#SizeAndTimeBasedFNATP)的maxHistory属性，可以控制最大归档文件数。 如果您的滚动策略要求每月滚动并且您希望保留一年的日志，只需将maxHistory属性设置为12.将自动删除超过12个月的存档日志文件。

### 自动压缩存档的日志文件

[RollingFileAppender](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/appenders.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjeKCV5-S5P_a0hUMWGkus6yEudeA#RollingFileAppender)可以在翻转期间自动压缩归档日志文件。 压缩始终以异步方式发生，因此即使对于大型日志文件，也不会在压缩期间阻止应用程序。

### 谨慎的模式

在[谨慎模式下](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/appenders.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjeKCV5-S5P_a0hUMWGkus6yEudeA#prudent) ，在多个JVM上运行的多个`FileAppender`实例可以安全地写入同一个日志文件。 由于某些限制，谨慎模式扩展到`RollingFileAppender` 。

### 莉莉丝

[Lilith](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=http://lilith.huxhorn.de/&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjVyV4il5u8LBsQiPvWATidvYsK4w)是一个用于logback的日志记录和访问事件查看器。 它与log4j的电锯相当，只不过Lilith设计用于处理大量的测井数据而不会退缩。

### 条件处理配置文件

开发人员经常需要在针对不同环境（如开发，测试和生产）的多个logback配置文件之间进行操作。 这些配置文件有很多共同之处，仅在少数几个地方有所不同。 为了避免重复，logback支持在`<if>` ， `<then>`和`<else>`元素的帮助[下对配置文件](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/configuration.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhgRA0DKswZ_dsz-GTjGf9lqf_6q1g#conditional)进行[条件处理，](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/configuration.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhgRA0DKswZ_dsz-GTjGf9lqf_6q1g#conditional)以便单个配置文件可以充分地针对多个环境。

### 过滤器

Logback提供了大量的[过滤功能](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/filters.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjXLXyblQW4EUGOh2SEJYGGva8UpQ) ，远远超出了log4j所提供的功能。 例如，假设您在生产服务器上部署了业务关键型应用程序。 在处理大量事务的情况下，将日志记录级别设置为WARN，以便仅记录警告和错误。 现在想象一下，您遇到了一个可以在生产系统上重现的错误，但由于这两个环境（生产/测试）之间存在未指定的差异，因此在测试平台上仍然难以捉摸。

使用log4j，您唯一的选择是将生产系统上的日志记录级别降低到DEBUG以尝试识别问题。 不幸的是，这将产生大量的记录数据，使分析变得困难。 更重要的是，广泛的日志记录会影响应用程序在生产系统上的性能。

使用logback，您可以选择将所有用户的日志记录保持在WARN级别，除了负责识别问题的一个用户Alice。 当Alice登录时，她将以DEBUG级别登录，而其他用户可以继续登录WARN级别。 这个专长可以通过在配置文件中添加4行XML来实现。 在本手册的[相关部分](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/filters.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjXLXyblQW4EUGOh2SEJYGGva8UpQ#TurboFilter)中搜索`MDCFilter` 。

### SiftingAppender

[SiftingAppender](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/appenders.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjeKCV5-S5P_a0hUMWGkus6yEudeA#SiftingAppender)是一个非常多才多艺的appender。 它可用于根据**任何**给定的运行时属性分离（或筛选）日志记录。 例如， `SiftingAppender`可以根据用户会话分离日志记录事件，以便每个用户生成的日志进入不同的日志文件，每个用户一个日志文件。

### 堆栈跟踪包装数据

当logback打印异常时，堆栈跟踪将包含打包数据。 以下是[logback-demo](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/demo.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhgu7r6nIUu77LTwIf20ZZjzjfiFWA) Web应用程序生成的示例堆栈跟踪。

```
14:28:48.835 [btpool0-7] INFO  c.q.l.demo.prime.PrimeAction - 99 is not a valid value
java.lang.Exception: 99 is invalid
  at ch.qos.logback.demo.prime.PrimeAction.execute(PrimeAction.java:28) [classes/:na]
  at org.apache.struts.action.RequestProcessor.processActionPerform(RequestProcessor.java:431) [struts-1.2.9.jar:1.2.9]
  at org.apache.struts.action.RequestProcessor.process(RequestProcessor.java:236) [struts-1.2.9.jar:1.2.9]
  at org.apache.struts.action.ActionServlet.doPost(ActionServlet.java:432) [struts-1.2.9.jar:1.2.9]
  at javax.servlet.http.HttpServlet.service(HttpServlet.java:820) [servlet-api-2.5-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.servlet.ServletHolder.handle(ServletHolder.java:502) [jetty-6.1.12.jar:6.1.12]
  at ch.qos.logback.demo.UserServletFilter.doFilter(UserServletFilter.java:44) [classes/:na]
  at org.mortbay.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1115) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.servlet.ServletHandler.handle(ServletHandler.java:361) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.webapp.WebAppContext.handle(WebAppContext.java:417) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:230) [jetty-6.1.12.jar:6.1.12]
```

从上面可以看出，应用程序正在使用Struts 1.2.9版，并在jetty版本6.1.12下部署。 因此，堆栈跟踪将快速告知读者介入异常的类以及它们所属的包和包版本。 当您的客户向您发送堆栈跟踪时，作为开发人员，您将不再需要让他们向您发送有关他们正在使用的软件包版本的信息。 该信息将成为堆栈跟踪的一部分。 有关详细信息，请参阅[“％xThrowable”转换字](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://logback.qos.ch/manual/layouts.html&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjkuT4YZQccDzgjVm9Kjb6H7iZ0Gg#xThrowable) 。

对于某些用户错误地认为它是[IDE的](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=http://www.jetbrains.net/devnet/message/5259058&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjwlFSVrE_jL1Ln-DJY7gYOFyHG3A)一个[功能，](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=http://www.jetbrains.net/devnet/message/5259058&xid=17259,1500004,15700021,15700124,15700149,15700168,15700186,15700190,15700201,15700208&usg=ALkJrhjwlFSVrE_jL1Ln-DJY7gYOFyHG3A)这个功能非常有用。

### Logback-access，即带有大脑的HTTP访问日志记录，是logback的一个组成部分

最后但并非最不重要的是，logback-access模块是logback发行版的一部分，它与Servlet容器（如Jetty或Tomcat）集成，以提供丰富而强大的HTTP访问日志功能。 由于logback-access是初始设计的一部分，因此您喜欢的所有logback-classic功能也可用于logback-access。

### 综上所述

我们列出了一些优先考虑log4j的logback的原因。 鉴于logback建立在我们之前关于log4j的工作上，简单地说，logback只是一个更好的log4j。