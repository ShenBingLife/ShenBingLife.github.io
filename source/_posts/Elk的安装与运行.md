---
layout: post
title: ELK环境搭建
category: 日志采集
tags: 
    - 日志采集
    - ELK
description: ELK 是 ElasticSearch、 LogStash、 Kibana 三个开源工具的简称，现在还包括 Beats，本文介绍了ELK的安装流程
---

# ELK环境搭建

[TOC]



ELK 是 ElasticSearch、 LogStash、 Kibana 三个开源工具的简称，现在还包括 Beats，其分工如下:

- LogStash/Beats: 负责数据的收集与处理
- ElasticSearch: 一个开源的分布式搜索引擎，负责数据的存储、检索和分析
- Kibana: 提供了可视化的界面。负责数据的可视化操作

基于 ELK Stack 可以构建日志分析平台、数据分析搜索平台等非常有用的项目。

作为学习笔记的第一篇，简单介绍下 ELK 各个软件的安装与简单配置，快速的搭建一个日志的查询平台

### 一. ElasticSearch 的安装与运行

ES 是一个基于 Lucene 的使用 Java 开发的开源搜索引擎，因此其运行是基于 JVM 的，因此在安装之前需要保证已经安装了 Java 环境。ES 要求使用 Java8 或者更高版本的 Java 环境。

确定机器已经安装了 Java 环境后，就可以安装 ES 了。官网提供了压缩包可以直接下载，

```
# 下载压缩包
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.0.tar.gz
# 解压
tar -xzf elasticsearch-6.3.0.tar.gz
# 进入文件
cd elasticsearch-6.3.0/bin/
```

下载解压完成后进入目录，启动命令在 bin 目录下，直接运行命令就可以启动了

```
./elasticsearch
```

ElasticSearch 的默认启动端口是 9200。手动访问出现如下信息说明启动成功。

集群搭建：

**ip:172.20.9.23**

```

#cluster.name: my-application
 node.name: node-1 
 path.data: /data/es-data
# Path to log files:
 path.logs: /var/log/elasticsearch/
 network.host: 172.20.9.23
 http.port: 9200
 http.cors.enabled: true
 http.cors.allow-origin: "*"
# 允许HTTP请求的方法类型 
 http.cors.allow-methods: OPTIONS,HEAD,GET,POST,PUT,DELETE
# 允许HTTP请求头返回类型
 http.cors.allow-headers: X-Requested-With,Content-Type,Content-Length,Authorization,Content-Encoding,Accept-Encoding
# 支持HTTP访问API 总开关
 http.enabled: true
 discovery.zen.ping.unicast.hosts: ["172.20.9.23", "172.20.9.22", "172.20.9.24"]
# 节点角色
 node.master: true
 node.data: true
 node.ingest: false
 search.remote.connect: false
 discovery.zen.minimum_master_nodes: 2
 gateway.recover_after_nodes: 3
```

**ip:172.20.9.22**

```
#cluster.name: my-application
 node.name: node-1 
 path.data: /data/es-data
# Path to log files:
 path.logs: /var/log/elasticsearch/
 network.host: 172.20.9.22
 http.port: 9200
 http.enabled: true
 discovery.zen.ping.unicast.hosts: ["172.20.9.23", "172.20.9.22", "172.20.9.24"]
# 节点角色
 node.master: true
 node.data: true
 node.ingest: false
 search.remote.connect: false
 discovery.zen.minimum_master_nodes: 2
 gateway.recover_after_nodes: 3
```

**ip:172.20.9.24**

```
#cluster.name: my-application
 node.name: node-1 
 path.data: /data/es-data
# Path to log files:
 path.logs: /var/log/elasticsearch/
 network.host: 172.20.9.24
 http.port: 9200
 http.enabled: true
 discovery.zen.ping.unicast.hosts: ["172.20.9.23", "172.20.9.22", "172.20.9.24"]
# 节点角色
 node.master: false
 node.data: true
 node.ingest: false
 search.remote.connect: false
 discovery.zen.minimum_master_nodes: 2
 gateway.recover_after_nodes: 3
```

### 二.Head插件安装

**（1）安装NodeJS**

```
[root@node1 ~]# yum install -y nodejs
或者自己下载tar文件解压
```

wget https://npm.taobao.org/mirrors/node/latest-v4.x/node-v4.5.0-linux-x64.tar.gz

tar -zxvf node-v4.5.0-linux-x64.tar.gz

配置下环境变量,编辑/etc/profile添加

export NODE_HOME=/usr/local/node-v4.5.0-linux-x64
export PATH=$PATH:$NODE_HOME/bin/
export NODE_PATH=$NODE_HOME/lib/node_modules

执行 source /etc/profile

**（2）安装npm**

```
[root@node1 ~]#  npm install -g cnpm --registry=https://registry.npm.taobao.org
```

**（3）使用npm安装grunt**

```
[root@node1 ~]# npm install -g grunt
[root@node1 ~]# 
```

```
[root@node1 ~]# npm install -g grunt-cli --registry=https://registry.npm.taobao.org --no-proxy
[root@node1 ~]# 
```

**（4）版本确认**

```
[es@node1 ~]$ node -v
[es@node1 ~]$ npm -v
[es@node1 ~]$ grunt -version
[es@node1 ~]$
```

**（5）下载head插件源码**

```
[es@node1 ~]$ wget https://github.com/mobz/elasticsearch-head/archive/master.zip
[es@node1 ~]$ unzip master.zip 
```

**（6）下载依赖** 
进入elasticsearch-head-master目录，执行下面命令

```
[es@node1 elasticsearch-head-master]$ npm install
[es@node1 elasticsearch-head-master]$ 
```

如果上面命令安装较慢或失败，可以尝试国内镜像安装

```
[es@node1 elasticsearch-head-master]$ sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
[sudo] password for es: 
[es@node1 elasticsearch-head-master]$ cnpm install
[es@node1 elasticsearch-head-master]$ 
```

### 配置

**（0）停止ElasticSearch** 
如果ElasticSearch已经启动，需要先停止

```
[es@node1 ~]$ jps
3261 Elasticsearch
3375 Jps
[es@node1 ~]$ kill 3261
```



**（1）配置 ElasticSearch，使得HTTP对外提供服务**

```
[es@node1 elasticsearch-6.1.1]$ vi config/elasticsearch.yml
```

添加如下内容

```
# 增加新的参数，这样head插件可以访问es。设置参数的时候:后面要有空格
http.cors.enabled: true
http.cors.allow-origin: "*"
```

**（2）修改Head插件配置文件**

```
[es@node1 elasticsearch-head-master]$ vi Gruntfile.js
```

找到connect：server，添加hostname一项，如下

```
connect: {
                        server: {
                                options: {
                                        hostname: '0.0.0.0',
                                        port: 9100,
                                        base: '.',
                                        keepalive: true
                                }
                        }
                }
```

###  启动

**（1）启动elasticsearch** 
首先确认elasticsearch已经启动

```
[es@node1 elasticsearch-6.1.1]$ bin/elasticsearch -d
[es@node1 elasticsearch-6.1.1]$ jps
3451 Jps
3436 Elasticsearch
[es@node1 elasticsearch-6.1.1]$
```

**（2）启动head** 
通过命令`grunt server`启动head

```
[es@node1 elasticsearch-head-master]$ grunt server
需要在head的目录下运行
```

或者通过命令`npm run start`也可以启动head

```
[es@node1 elasticsearch-head-master]$ npm run start
```

**（3）访问9100端口** 
[http://172.20.9.23:9100/](http://node1:9100/)

### 三. FileBeats 与 LogStash 的安装

LogStash 可以用来对日志进行收集并进行过滤整理后输出到 ES 中，FileBeats 是一个更加轻量级的日志收集工具。 
现在最常用的方式是通过 FileBeats 收集目标日志，然后统一输出到 LogStash 做进一步的过滤，在由 LogStash 输出到 ES 中进行存储。

#### 1. LogStash 的安装运行

官方提供了压缩包下载， <https://www.elastic.co/downloads/logstash> 。 下载完成后解压即可。

```
tar zxvf logstash-6.3.0.tar.gz
# 进入目录
cd logstash-6.3.0
```

LogStash 的运行需要指定一个配置文件，来指定数据的流向，我们在当前目录下创建一个 first.conf 文件，其内容如下:

```
# 配置输入为 beats
input {
    beats {
            port => "5044"
    }
}
# 数据过滤
filter {
    grok {
            match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    geoip {
            source => "clientip"
    }
}
# 输出到本机的 ES
output {
    elasticsearch {
            hosts => [ "172.20.9.23:9200"  ]

    }

}
```

上面配置了 LogStash 输出日志到 ES 中，具体字段在后面的笔记中会详细介绍，这里先用起来再说。 
配置完成后就可以通过如下方式启动 LogStash 了

```
bin/logstash -f first.conf --config.reload.automatic1
```

可以看到命令行会打印出如下信息， 可以看到 LogStash 默认端口为 5044:

```
[2018-03-08T23:12:44,087][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2018-03-08T23:12:44,925][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.2.2"}
[2018-03-08T23:12:45,623][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
[2018-03-08T23:12:49,960][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50}
[2018-03-08T23:12:50,882][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://localhost:9200/]}}
[2018-03-08T23:12:50,894][INFO ][logstash.outputs.elasticsearch] Running health check to see if an Elasticsearch connection is working {:healthcheck_url=>http://localhost:9200/, :path=>"/"}
[2018-03-08T23:12:51,303][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://localhost:9200/"}
[2018-03-08T23:12:51,595][INFO ][logstash.outputs.elasticsearch] ES Output version determined {:es_version=>nil}
[2018-03-08T23:12:51,604][WARN ][logstash.outputs.elasticsearch] Detected a 6.x and above cluster: the `type` event field won't be used to determine the document _type {:es_version=>6}
[2018-03-08T23:12:51,641][INFO ][logstash.outputs.elasticsearch] Using mapping template from {:path=>nil}
[2018-03-08T23:12:51,676][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>60001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date"}, "@version"=>{"type"=>"keyword"}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2018-03-08T23:12:51,773][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["//localhost:9200"]}
[2018-03-08T23:12:52,176][INFO ][logstash.filters.geoip   ] Using geoip database {:path=>"/Users/zouyingjie/soft/study/ELK/logstash-6.2.2/vendor/bundle/jruby/2.3.0/gems/logstash-filter-geoip-5.0.3-java/vendor/GeoLite2-City.mmdb"}
[2018-03-08T23:12:53,026][INFO ][logstash.inputs.beats    ] Beats inputs: Starting input listener {:address=>"0.0.0.0:5044"}
[2018-03-08T23:12:53,195][INFO ][logstash.pipeline        ] Pipeline started succesfully {:pipeline_id=>"main", :thread=>"#<Thread:0x66461e40 run>"}
[2018-03-08T23:12:53,290][INFO ][org.logstash.beats.Server] Starting server on port: 5044
[2018-03-08T23:12:53,401][INFO ][logstash.agent           ] Pipelines running {:count=>1, :pipelines=>["main"]}
123456789101112131415161718
```

#### 2. 安装运行 FileBeats

FileBeats 也提供了下载包，地址为 <https://www.elastic.co/downloads/beats/filebeat> 。找到系统对应的包下载后解压即可。

```
tar zxvf filebeat-6.3.0-darwin-x86_64.tar.gz
cd filebeat-6.3.0-darwin-x86_6412
```

进入目录编辑 filebeat.yml 找到对应的配置项，配置如下

```
- type: log
   # Change to true to enable this prospector configuration.
    enabled: True

    # Paths that should be crawled and fetched. Glob based paths.
    # 读取 Nginx 的日志
    paths:
      - /usr/local/nginx/logs/*.log

#----------------------------- Logstash output --------------------------------
# 输出到本机的 LogStash
output.logstash:
  # The Logstash hosts
  hosts: ["172.20.9.23:5044"，"172.20.9.22:5044"，"172.20.9.24:5044"]
```

配置完成后执行如下命令，启动 FileBeat 即可

```
# FileBeat 需要以 root 身份启动，因此先更改配置文件的权限
sudo chown root filebeat.yml
sudo ./filebeat -e -c filebeat.yml -d "publish"
```

### 四. Kibana 的安装运行

Kibana 也提供了对应的安装包下载，链接为 <https://www.elastic.co/downloads/kibana> , Mac、Linux、Win 都有对应的安装包，直接下载解压即可

```
tar zxvf kibana-6.3.0-darwin-x86_64.tar.gz
cd kibana-6.3.0-darwin-x86_64
# 直接启动即可
bin/kibana
```

Kibana 默认链接了本机的 9200 端口，其绑定的端口为 5601，启动成功后直接访问 172.20.9.23:5601 端口即可,界面如下。我因为安装了 x-pack 插件因此显示的项可能会多一些，这个暂时忽略.
