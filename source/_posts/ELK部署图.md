---
layout: post
title: ELK部署图
category: 日志采集
description: ELK 通用的部署架构图
---

# ELK部署图

ELK 常用架构及使用场景介绍
在这个章节中，我们将介绍几种常用架构及使用场景。

最简单架构
在这种架构中，只有一个 Logstash、Elasticsearch 和 Kibana 实例。Logstash 通过输入插件从多种数据源（比如日志文件、标准输入 Stdin 等）获取数据，再经过滤插件加工数据，然后经 Elasticsearch 输出插件输出到 Elasticsearch，通过 Kibana 展示。详见图 1。

### 最简单架构

这种架构非常简单，使用场景也有限。初学者可以搭建这个架构，了解 ELK 如何工作。

Logstash 作为日志搜集器
这种架构是对上面架构的扩展，把一个 Logstash 数据搜集节点扩展到多个，分布于多台机器，将解析好的数据发送到 Elasticsearch server 进行存储，最后在 Kibana 查询、生成日志报表等。详见图 2。

![image](https://tvax3.sinaimg.cn/large/007S2YVMgy1gdzje8vav1j30jm05iweq.jpg)

### Logstash 作为日志搜索器

这种结构因为需要在各个服务器上部署 Logstash，而它比较消耗 CPU 和内存资源，所以比较适合计算资源丰富的服务器，否则容易造成服务器性能下降，甚至可能导致无法正常工作。

![image](https://tva4.sinaimg.cn/large/007S2YVMgy1gdzjdvduhwj30db09bq38.jpg)

### Beats 作为日志搜集器
这种架构引入 Beats 作为日志搜集器。目前 Beats 包括四种：

Packetbeat（搜集网络流量数据）；
Topbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）；
Filebeat（搜集文件数据）；
Winlogbeat（搜集 Windows 事件日志数据）。
Beats 将搜集到的数据发送到 Logstash，经 Logstash 解析、过滤后，将其发送到 Elasticsearch 存储，并由 Kibana 呈现给用户。详见图 3。

图 3. Beats 作为日志搜集器
![image](https://tva4.sinaimg.cn/large/007S2YVMgy1gdzjd84rztj30gl0bx755.jpg)

这种架构解决了 Logstash 在各服务器节点上占用系统资源高的问题。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。另外，Beats 和 Logstash 之间支持 SSL/TLS 加密传输，客户端和服务器双向认证，保证了通信安全。

因此这种架构适合对数据安全性要求较高，同时各服务器性能比较敏感的场景。

引入消息队列机制的架构
到笔者整理本文时，Beats 还不支持输出到消息队列，所以在消息队列前后两端只能是 Logstash 实例。这种架构使用 Logstash 从各个数据源搜集数据，然后经消息队列输出插件输出到消息队列中。目前 Logstash 支持 Kafka、Redis、RabbitMQ 等常见消息队列。然后 Logstash 通过消息队列输入插件从队列中获取数据，分析过滤后经输出插件发送到 Elasticsearch，最后通过 Kibana 展示。详见图 4。



### 引入消息队列机制的架构

这种架构适合于日志规模比较庞大的情况。但由于 Logstash 日志解析节点和 Elasticsearch 的负荷比较重，可将他们配置为集群模式，以分担负荷。引入消息队列，均衡了网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性，但依然存在 Logstash 占用系统资源过多的问题。

基于 Filebeat 架构的配置部署详解
前面提到 Filebeat 已经完全替代了 Logstash-Forwarder 成为新一代的日志采集器，同时鉴于它轻量、安全等特点，越来越多人开始使用它。这个章节将详细讲解如何部署基于 Filebeat 的 ELK 集中式日志解决方案，具体架构见图 5。

![image](https://tva3.sinaimg.cn/large/007S2YVMgy1gdzjcajurcj30jp0960tf.jpg)

### 基于 Filebeat 的 ELK 集群架构

因为免费的 ELK 没有任何安全机制，所以这里使用了 Nginx 作反向代理，避免用户直接访问 Kibana 服务器。加上配置 Nginx 实现简单的用户认证，一定程度上提高安全性。另外，Nginx 本身具有负载均衡的作用，能够提高系统访问性能。

![image](https://tva1.sinaimg.cn/large/007S2YVMgy1gdzj9s0ovfj30n508wt9l.jpg)

转载自 https://www.ibm.com/developerworks/cn/opensource/os-cn-elk-filebeat/index.html

