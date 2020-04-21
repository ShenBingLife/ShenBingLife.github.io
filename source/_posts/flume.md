---
layout: post
title: Flume
category: 日志采集
tags: 
    - 日志采集
    - Flume
    - Hadoop
description: Flume功能简介
---

# Flume

[TOC]



## 是什么？

flume是一个实时数据收集工具，hadoop的生态圈之一，主要用来在分布式环境下各服务器节点做数据收集，然后汇总到统一的数据存储平台，flume支持多种部署架构模式，单点agent部署，分层架构模式部署，如通过一个负载均衡agent将收集的数据分发到各个子agent，然后在汇总到同一个agent上，数据传输到统一的数据存储平台

![image](https://tvax2.sinaimg.cn/large/007S2YVMgy1gdzj3sal7hj30co05bt8u.jpg)

## 优点

1. Flume的管道是基于事务，保证了数据在传送和接收时的一致性.
2. Flume是可靠的，容错性高的，可升级的，易管理的,并且可定制的。
3. 扇入扇出，一个channel支持多个source和sink，可以实时将数据流向分析工具
4. 多种input和output类型支持

## 缺点

Flume的配置很繁琐，source，channel，sink的关系在配置文件里面交织在一起，不便于管理

弱的分区和拦截器功能

Java开发，受限JVM的性能。

## 类似框架

Facebook的Scribe，还有Apache新出的另一个明星项目chukwa，还有淘宝Time Tunnel

### [Logstash](https://www.elastic.co/products/logstash)

![img](https://tvax2.sinaimg.cn/large/007S2YVMgy1gdzj529p7fj30j30cnq3u.jpg)

**主要特征：**

- 从各种来源中提取数据：日志，指标，Web应用程序，数据存储，AWS，而不会失去并发性。
- 实时数据解析。
- 从非结构化数据创建结构。
- 用于数据安全的流水线加密。
- logstash基于JVM，支持跨平台；

**成本：**开源

#### 优点：

1. 多插件input和output
2. Logstash可以和ELK其他组件配合，开发、应用都会简单很多，技术成熟，使用场景广泛
3. 采集过程中，提供Filter负责对采集的日志进行分析和重写
4. SNMP支持
5. output支持zabbix

缺点：

如果做日志过滤和分析，需要学习grok正则，Grok内置了120多种的正则表达式库，地址:<https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns>。

### [Fluentd](http://www.fluentd.org/)

Fluentd的旗舰功能是一个广泛的插件库，它在简洁的开发环境中提供与日志和数据管理相关的任何扩展支持和功能。

**主要特征：**

- 统一日志记录层，可以将数据从多个来源中分离出来。
- 给非结构化日志提供结构。
- 灵活，但很简单。需要几分钟才能完成。
- 与大多数现代数据源兼容。
- 支持SNMP
- input，和output支持zabbix

**成本：**

- 免费：开源

**基本：**

Fluentd使用C/Ruby开发，使用JSON文件来统一日志数据。

Fluentd的部署和Flume非常相似：

![img](https://tva4.sinaimg.cn/large/007S2YVMgy1gdzj5jmvqaj30hs0kh7gx.jpg)

#### 优点：

采用JSON统一数据/日志格式是它的另一个特点。相对去Flumed，配置也相对简单一些

#### 缺点：

ruby开发，如果自定义插件需要学习成本

### Scribe

**Facebook已停止维护。

### chukwa

版本低，活跃度低

### TimeTunnel

文档已死

