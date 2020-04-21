---
layout: post
title: Yarn压测系统
category: Yarn
description: Yarn压测系统的开发与设计，Yarn源码阅读
---


# Yarn压测系统
背景： 为了验证在Hadoop集群扩容后（5k-10k-15k），保证性能的稳定性，设计该压测系统。 
 
验证的主要方向是NM的心跳机制对Container调度速率的影响；集群扩容后，放大现网的实际任务规模，验证调度速率。
## 设计模拟集群

## 性能采集与可视化

采集：基于Yarn JMX Restful接口采集数据，每隔10s采集一次，存储到InfluxDB。 

可视化：基于Grafana定制图表，主要分为4块Dashboard
1. RM性能指标（包括调度速率、APP提交速率、RM内存和GC、集群队列的资源、Apps、Container数量）
2. 调度器指标：包括调度速率、调度步骤长度、每次调度跳过的节点数等
3. EventMetric： RM的AsyncDispatcher处理的事件数量
4. 队列状态表格：从root队列开始，列出所有子队列的资源状态和AppsPending数量，直观判断是否资源不足引起的调度阻塞，支持递归对子队列钻取

## 设计负载工具
负载工具目前共有4种：
1. 基于概率统计的负载工具：支持从现网的jhist分析作业的时间长度、个数的随机范围，并生成一系列的模拟任务提交到Yarn集群
2. 基于现网负载回放：将现网的jhist分析后，获得作业的执行时间和作业长度、作业的task的平均时间，提交模拟任务
3. 基于现网负载回放（MR版本）：同上面一样，获得task信息后，创建向Yarn提交MR模型版本的模拟作业
4. 指定container速率压测的负载工具：为了达到指定的ContainerAllocationRateAvgTime，不断的向集群提交作业，并且具有启发式的修改作业提交的规模和时间长度

### 基于现网负载回放
优化步骤：
1. NM本地预存模拟作业的jar包，消除HDFS读取步骤
2. 对现网作业分析后，支持每个模拟Container执行该现网作业Container的平均时间，或者将Container的运行时间分成2组，同时执行长作业Container和短作业Container
3. jhist中TaskAttempt的时间长度是资源的实际使用时间，而Task的时间长度包含了更多其他的时间
4. 自定义SleepContainerExecutor，通过NM执行sleep线程来模拟作业对资源的占用
5. 模拟作业的AM注销时，从NMClient中直接移除关闭的Container，而不是等待与RM通信关闭Container

### 指定container速率压测的负载工具
优化步骤： 
1. 读取JMX指标时，创建JVM缓存
2. 每个客户端创建RateLimiter限制提交速度
3. 在客户端刚启动时，delay一段时间，预提交一些作业，防止调度速率太低引起的大量作业提交
4. 在调度速率过低时，如果有空闲队列，则调大作业的Container数量，如果没有空闲队列，则调小Container的资源使用量和运行时间