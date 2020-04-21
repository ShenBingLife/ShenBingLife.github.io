---
layout: post
title: Logstash插件
category: 日志采集
tags: 
    - 日志采集
    - ELK
description: Logstash插件介绍和配置信息
---

# Logstash插件

```
插件获取地址：
https://github.com/logstash-plugins 

在线安装:
/plugin install  logstash-input-jdbc

升级插件：
/plugin update logstash-input-jdbc
卸载插件：
/plugin uninstall  logstash-input-jdbc

淘宝源地址： https://ruby.taobao.org
```

### Logstash插件分类

- inputs 输入
- codecs 解码
- filters 过滤
- outputs 输出

### Logstash_input插件

**stdin标准输入**

可用参数

```
参数               Input Type                Required        Default Value
add_field            hash                      No                 {}
codec                codec                     NO                 "line"
tags                 array                     NO             
type                 string                    NO
```

**file文件输入**

可用参数

```
input {
    file {
        #file插件的参数
        codec=>...                        #可选项                codec,         默认是plain,可通过这个参数设置编码方式
        discover_interval=>...            #可选项                number,        logstash没隔多久去检查一次被监听的path下是否有新文件,默认值是15秒
        exclude=>...                      #可选项                array,         不想被监听的文件可以排除出去,这里跟path一样支持glob展开
        sincedb_path=>...                 #可选项                string,        如果你不想用默认的$HOME/.sincedb,可以通过这个配置定义sincedb文件到其他位置    
        stat_interval...                  #可选项                number,        logstash没隔多久检查一次被监听文件状态(是否有更新),默认是1秒
        start_position...                 #可选项                string,        logstash从什么位置开始读取文件数据,默认是结束位置,也就是说logstash进程会以类似tial -f的形式运行.如果你是要导入一个完整的文件,把这个设定可以改成"beginning",logstash进程就会从文件开头读取,有点类似cat
        path=>...                         #必选项                array,         处理的文件的路径,可以定义多个路径
        tags=>...                         #可选项                array,         在数据处理的过程中,由具体的插件来添加或者删除的标记
        type=>...                         #可选项                string,        自定义将要处理事件类型,可以自己随便定义,比如处理的是linux的系统日志,可以定义为"syslog"
    }
}
```

实例1:过滤nginx的访问日志和错误日志

配置文件

```
[root@ELK-STACK logstash-1.5.5]# cat conf/test1.conf
input {
    file {
        path => ["/var/log/nginx/access.log"]
        type => "accesslog"
        start_position => "beginning"
        }
    file {
        path => ["/var/log/nginx/error.log"]
        type => "errorlog"
        start_position => "beginning"
        }
    }
output {
    stdout{}
    }
```

测试

```
[root@ELK-STACK logstash-1.5.5]# ./bin/logstash -f conf/test1.conf
Logstash startup completed
2017-02-23T08:00:25.757Z 0.0.0.0 2017/02/23 12:14:01 [error] 2145#0: *47 open() "/usr/share/nginx/html/zabbix" failed (2: No such file or directory), client: 172.16.2.64, server: _, request: "GET /zabbix HTTP/1.1", host: "172.16.1.225"
2017-02-23T08:00:25.757Z 0.0.0.0 172.16.2.64 - - [23/Feb/2017:12:14:01 +0800] "GET /zabbix HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36" "-"
2017-02-23T08:01:09.785Z 0.0.0.0 172.16.2.64 - - [23/Feb/2017:16:01:09 +0800] "GET / HTTP/1.1" 200 3700 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50" "-"
```

**TCP/UDP输入**

TCP

语法格式

```
input {
    tcp {
        #tcp插件的参数                           类型            默认值
        add_field=>...            #可选项        hash           {}
        codec=>...                #可选项                       plain
        data_timeout=>...        #可选项         Number         1
        host=>...                #可选项                        0.0.0.0                
        mode=>...                #可选项                       值是["server","client"]其中之一,默认是server    
        port=>...                #必填项         number        端口号,需要和另一端匹配
        ssl_cacert=>...            #可选项
        ssl_cert=>...            #可选项
        ssl_enable...            #可选项
        ssk_key=>...            #可选项
        ssl_key_passphrase=>...    #可选项
        ssl_verify...            #可选项
        tags=>...                #可选项         array         
        type=>...                #可选项         string         
    }
}
```

UDP

语法格式

```
input {
    udp {
        #可用参数  
                                                                 #默认值
        add_field=>...            #hash类型            #可选项      默认{}
        host=>...                #string类型           #可选项      默认0.0.0.0 
        port=>...                #number类型           #必填项      端口号,需要和另一端匹配
        tags=>...                #array类型            #必填项
        type=>...                #string类型           #可选项
        workers=>...            #number类型            #默认2
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### Logstash_codec插件

介绍: codec:编码,解码(json,msgpack,edn)

放置位置: input{}和output()字段里的任何插件里(比如说input里file插件里 input{file{codec=>json}})

**codec插件之plain**

介绍:plain是一个空的解析器，它可以让用户自己指定格式

**codec之json**

介绍

- 如果输入到logstash的内容是json格式,可以在input字段加入codec=>json来进行解析
- 如果想让logstash输出为json格式,可以在output字段加入codec=>json

语法格式: codec=> json

实例1:输出json化

配置文件

```
[root@ELK-STACK logstash-1.5.5]# cat conf/json.conf
input {
    stdin {
        }
    }
output {
    stdout {
        codec => json
        }
    }
```

测试

```
[root@ELK-STACK logstash-1.5.5]# ./bin/logstash -f conf/json.conf
Logstash startup completed             #等待logstash启动成功       
hello.world                            #输入一个字符串的hello.world
{"message":"hello.world","@version":"1","@timestamp":"2017-02-24T09:01:11.425Z","host":"0.0.0.0"}  #可以看到输入是以json化的格式输出的
```

**codec之json_lines**

介绍: 如果你的json文件比较长,需要换行的话,那么久得用到json_lines了

**codec之rubydebug**

介绍: rubydebug将采用Ruby Awsome Print库来解析日志

实例1:通过rubydebug输出键值对

配置文件

```
 cat conf/rubydebug.conf
input {
    stdin {
        codec => json       #输入为json格式,所以用codec=>json来解析
        }

    }



output {
    stdout {
        codec =>rubydebug  #让输出的格式为rubydebug模式,就是把键值对输出
        }
    }
```

测试

```
[root@ELK-STACK logstash-1.5.5]# ./bin/logstash -f conf/rubydebug.conf
Logstash startup completed
{"bookname":"elk stack"}             #这是输入的键值对
{                                        
      "bookname" => "elk stack",    #这是以rubydebug的模式输出键值对
      "@version" => "1",            #这是以rubydebug的模式输出键值对
    "@timestamp" => "2017-02-24T09:29:50.777Z",
          "host" => "0.0.0.0"
}
{"bookname":"elk stack","price":29}           #这是输入的键值对
{
      "bookname" => "elk stack",               #这是以rubydebug的模式输出键值对
         "price" => 29,                        #这是以rubydebug的模式输出键值对
      "@version" => "1",
    "@timestamp" => "2017-02-24T09:31:08.224Z",
          "host" => "0.0.0.0"
}
```

**codec之multiline**

介绍: 有些时候，应用程序调试日志会包含非常丰富的内容，为一个事件打印出很多行内容。这种日志通常都很难通过命令行解析的方式做分析;而 logstash 正为此准备好了 codec/multiline 插件

语法格式


```
input {
    stdin {
        codec =>multiline {
            charset=>...          #可选                      字符编码         
            max_bytes=>...        #可选     bytes类型            设置最大的字节数
            max_lines=>...        #可选     number类型           设置最大的行数,默认是500行
            multiline_tag...      #可选     string类型           设置一个事件标签,默认是multiline
            pattern=>...          #必选     string类型           设置匹配的正则表达式
            patterns_dir=>...     #可选     array类型           可以设置多个正则表达式
            negate=>...           #可选     boolean类型         设置true是向前匹配,设置false向后匹配,默认是FALSE
            what=>...             #必选                        设置未匹配的内容是向前合并还是先后合并,previous,next两个值选择
        }
    }

}
```

### Logstash_filter插件

**json filter**

介绍:如果数据格式是json,那么可以通过它把数据解析成你想要的数据结构

语法格式

```
filter {
    json {
        add_field=>...            #可选项         #hash          添加属性,默认{}
        add_tag=>...              #可选项         #array          添加标识,默认{}
        remove_field=>...         #可选项         #array         删除属性,默认{}
        remove_tag=>...            #可选项         #array         删除标识,默认{}
        source=>...                #必选项         #string       指定来源数据
        target=>...                #可选项         #string        定义将要解析的目标字段
    }
}
```


实例1:解析json数据

配置文件


```
vim conf/filter_json.conf
 input {
     stdin {
         }
     }
 filter {
     json {
         source => "message"
         target => "content"
         }
     }
 output {
         stdout {
             codec=>rubydebug
             }
     }
```

测试

```
[root@ELK-STACK logstash-1.5.5]# ./bin/logstash -f conf/filter_json.conf
Logstash startup completed
{"name":"zhai","age":12}       #输入被解析的值
{
       "message" => "{\"name\":\"zhai\",\"age\":12}",
      "@version" => "1",
    "@timestamp" => "2017-02-27T08:31:13.193Z",
          "host" => "0.0.0.0",
       "content" => {
        "name" => "zhai",   #这里就是json字段里的name
         "age" => 12        #这里就是json字段里的age
    }
}
```

**grok filter**

介绍

- grok是目前logstash里最好的一种解析各种非结构化的日志数据的工具
- grok可以过滤日志中你想要的字段
- 官方patterns地址:https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns
- 测试grok匹配规则的网址:grokdebug.herokuapp.com

可用参数

```
filter {
    grok {
        match=>...        #可选项        写匹配规则
    }
}
```

**kv filter**

介绍: 通过指定一个分隔符,截取key,value

KV插件设置一定的格式，来读取字段中的键值对

其中有两个比较重要的参数，如下：

- field_split
- value_split

field_split为两个键值对间的分隔符，而value_split则是两个key,value之间的分隔符

```
若要切割的字段如下： 
message=”messagid=123&name=logstash” 
kv的配置如下： 
kv{ 
source=>”message” 
field_split =>”&” 
value_split =>”=” 
} 
则message会把切割成 
messageid=”123” 
name=”logstash”

```

### mutate插件

mutate插件可以把一些字符串替换成别的字符串，通过gsub来实现。或者通过split来切割不同的字符串

```
mutate { 
gsub=>[“message”, “\n”, “$”] 
split=>[“message”,”keyword”] 
}
这样配置，message会把message字段中的\n替换成$,并且根据keyword把messageqi切割字符串，通过message，message来获取字段。

```

### Logstash_output插件

介绍: logstash output就是如何让数据经过logstash的解析和处理后,把结果输出

**输出到file**

可用参数

```
path => "/root/access_result"                 #结果输出到文件里
path => "/root/%{+YYYY.MM.dd}-%{host}-access" #结果输出到文件里,以时间命名文件
message_format => "%{ip}"                      #只输出filter过滤出来的指定字段
```

**输出到elasticsearch**

- elasticsearch就是数据库
- 把logstash解析过滤的结果输出到elasticsearch里

实例1:输出到elasticsearch

```
output {
    elasticsearch {
        host => "172.16.1.225"             #elasticsearch的地址,或者cluster=>"ClusterName"
        protocol =>"http"            
        index=>"test_output-%{type}-%{+YYYY.MM.dd}"    #在elasticsearch里创建索引,索引名称设置,type就是document_type的值,test_output-nginx-2017-02-28
        document_type=>"nginx"
        worker=>5
    }
}
```

### Output插件WebHDFS介绍

webhdfs插件主要是把日志上传到HDFS文件系统中，其格式为：

```
webhdfs{ 
host => “127.0.0.1” 
port => 50070 
user => “hadoop” 
path => “/logstash/data” 
codec => plain 
{ 
charset=>”UTF-8” 
format=>”%{message}” 
} 
}

```

