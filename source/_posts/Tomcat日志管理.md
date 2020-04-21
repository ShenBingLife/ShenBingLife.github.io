---
layout: post
title: Tomcat日志
tags: 
    - log
    - 日志
    - tomcat
category: 日志
description: tomcat的日志内容
---

# Tomcat日志
[TOC]

Tomcat 日志信息分为两类：

- 访问日志信息，记录访问的时间,IP,访问的资料等相关信息。
- 运行中的日志，主要记录运行的一些信息，尤其是一些异常错误日志信息。

## 访问日志：

Tomcat访问日志的配置在TOMCAT_HOME/conf/server.xml中（注：注释以下内容即可关闭访问日志）

```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />123
```

参数详解：

className：官方说明必须按照默认配置不可更改。 
directory：日志文件位置。 
prefix：日志文件前缀。 
suffix：日志文件后缀。 
pattern：日志模式参数，设置参数很丰富，参数说明见下表。 
resolveHosts：如果这个值是true的话，tomcat会将这个服务器IP地址通过DNS转换为主机名，如果是false，就直接写服务器IP地址。

pattern 参数： 
%a - 远端IP地址 
%A - 本地IP地址 
%b - 发送的字节数，不包括HTTP头，如果为0，使用”－” 
%B - 发送的字节数，不包括HTTP头 
%h - 远端主机名(如果resolveHost=false，远端的IP地址） 
%H - 请求协议 
%l - 从identd返回的远端逻辑用户名（总是返回 ‘-‘） 
%m - 请求的方法（GET，POST，等） 
%p - 收到请求的本地端口号 
%q - 查询字符串(如果存在，以 ‘?’开始) 
%r - 请求的第一行，包含了请求的方法和URI 
%s - 响应的状态码 
%S - 用户的session ID 
%t - 日志和时间，使用通常的Log格式 
%u - 认证以后的远端用户（如果存在的话，否则为’-‘） 
%U - 请求的URI路径 
%v - 本地服务器的名称 
%D - 处理请求的时间，以毫秒为单位 
%T - 处理请求的时间，以秒为单位

## 运行日志：

- 运行日志分为5类：catalina 、localhost 、manager 、admin 、host-manager

- 日志的级别分为7种：SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value) 
  除此之外，还可以使用OFF关闭相关日志，使用ALL输出所有级别的日志

- 修改 TOMCAT_HOME/conf/logging.properties 中的内容，设定某类日志的级别

  - 设置 catalina 日志的级别为： FINE

    ```
    1catalina.org.apache.juli.FileHandler.level = FINE1
    ```

  - 禁用 catalina 日志的输出：

    ```
    1catalina.org.apache.juli.FileHandler.level = OFF1
    ```

  - 输出 catalina 所有的日志消息：

    ```
    1catalina.org.apache.juli.FileHandler.level = ALL
    ```

https://blog.csdn.net/loyachen/article/details/47302143

# Linux下Tomcat日志定期清理(其他日志方法相同)

https://blog.csdn.net/oSayMissyou0/article/details/70209920