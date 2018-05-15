---
title: ELK入坑配置
date: 2015-11-12 18:03:26
tags: [code]
description: 记录了ELK日志分析系统的基本概念与配置
cover: /images/elastic.jpg
---

## Logstash
### nginx日志配置
- 进入conf目录,编辑nginx.conf文件

```bash
cd /usr/conf/
vim nginx.conf
```

- 配置nginx日志格式

```
//自定义分隔符，方便后续logstash的解析
log_format  main  "$remote_addr | $remote_user | $time_local | $request | $status | $body_bytes_sent | $http_referer | $http_user_agent | $http_x_forwarded_for ";
access_log  logs/access.log  main;
```

### logstash配置

- 进入logstash文件夹，新建配置文件test.conf

```bash
cd /opt/logstash
vim test.conf
```
```
input{
        file{
                path => "/usr/logs/access.log"
                start_position => "beginning"
                codec => json
        }
}
filter {
	ruby {
        init => "@kname = ['remote_addr','remote_user','time_local','request','status','body_bytes_sent','http_referer','http_user_agent','http_x_forwarded_for']"
        code =>"event.append(Hash[@kname.zip(event['message'].split(' | '))])"
    }
    if [request] {
        ruby {
            init => "@kname = ['method','uri','verb']"
            code => "event.append(Hash[@kname.zip(event['request'].split(' '))])"
        }
        if [uri] {
            ruby {
                init => "@kname = ['url_path','url_args']"
                code => "event.append(Hash[@kname.zip(event['request'].split('?'))])"
            }   
            kv {
                prefix => "url_"
                source => "url_args"
                field_split => "& "
                remove_field => [ "url_args","uri","request" ]
            }
        }
    }
    mutate {
        convert => [
            "body_bytes_sent" , "integer"
        ]
    }
    date {
        match => [ "time_local", "dd/MMM/yyyy:hh:mm:ss Z" ]
        locale => "en"
    }
}
output{
        //stdout用于测试环境输出
        stdout{
            codec => rubydebug
        }
        //假设elasticsearch在本地开启，若输出到远程服务器，添加配置
        elasticsearch{
            // host => 127.0.0.2
            // user => ***
            // password => ***
            codec => json
        }
} 
```

- 启动logstash

```bash
//检测conf是否通过
bin/logstash -f test.conf --configtest
configation OK
//启动
bin/logstash -f test.conf
/stdout记录输出,并输出到elasticsearch
{
                 "message" => "127.0.0.1 | - | 11/Nov/2015:13:14:53 +0800 | GET
/file/test.img?width=800&height=600 HTTP/1.1 | 404 | 570 | - | Mozilla/5.0 (Wind
ows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Sa
fari/537.36 | - \r",
                "@version" => "1",
              "@timestamp" => "2015-11-11T05:14:53.000Z",
                    "host" => "Jevirs-PC",
                    "path" => "D:\\nginx\\nginx\\logs\\access_test.log",
             "remote_addr" => "127.0.0.1",
             "remote_user" => "-",
              "time_local" => "11/Nov/2015:13:14:53 +0800",
                  "status" => "404",
         "body_bytes_sent" => 570,
            "http_referer" => "-",
         "http_user_agent" => "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/5
37.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36",
    "http_x_forwarded_for" => "- \r",
                  "method" => "GET",
                    "verb" => "HTTP/1.1",
                "url_path" => "GET /file/test.img",
               "url_width" => "800",
              "url_height" => "600"
}
```


## ElasticSearch
### 概念
- **节点 (node)**， **集群 (cluster)**
单台服务器，存储数据并参与集群的索引和搜索，服务器在启动时自动分配*ID* ，节点默认会加入*ID*为*ElasticSearch*的集群中，除非自己配置，如果只有一个节点，那么该节点也作为一个集群工作。
多个节点组成的一个集群，集群可跨节点，进行联合索引和搜索。集群默认*ID*为*ElasticSearch*

- **索引(index)** ，**类型(type)** ，**文档(document)**
例:
index可为blogSystem；
type可为blogs,comments,user；
document存储基本信息元；

- **分片 (shard)** 和 **副本(replica)**
可为一个索引指定n个分片，分片可以存储在集群中任一个节点上。
一个索引默认分配5个分片和一个副本

### 安装运行

```bash
//普通用户身份操作
curl -L -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.0.0/elasticsearch-2.0.0.tar.gz
...
...
tar -xvf elasticsearch-2.0.0.tar.gz
cd elasticsearch-2.0.0/bin
./elasticsearch   
...
...
```
服务占用9200端口，提供对文档操作RESTful API
```
curl -X<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>
//exp
curl -XGET 'localhost:9200/index/type/id?pretty'
curl -XPUT 'localhost:9200/index/type/1' -d 
'{
  "name": "John Doe"
}'
curl -XDELETE 'localhost:9200/customer'
```

### 查询语法

```bash
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match-all": { } },     //所有文档
  "query": { "match": { "address": "mill" } },   //特定文档
  "query": { "match": { "address": "aaa bbb" } }, //含有aaa或bbb
  "query": { "match_phrase": { "address": "aaa bbb" } }, //含有"aaa bbb"
  "size":10,   //记录条数
  "from":10,   //记录跳页
  "sort": { "balance": { "order": "desc" } },//排序
  
  //与
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  },
  //或
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  },
  //非
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  },
  //多条件
   "query": {
     "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  },
  //范围选择
   "query": {
     "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {"gte": 20000,"lte": 30000}
        }
      }
    }
  },

  //排序
    "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      }
    }
  }
  
}'
```

## Kibana
kibana运行于浏览器端，可对数据进行可视化分析
- 下载[Kibana][1]
- 链接远程elasticsearch:
  编辑kibana.yml，`elasticsearch.url:"http://elasticsearch:9200"`
- 解压并启动 `./bin/kibana`
- 在浏览器端输入http://localhost:5601


  [1]: https://www.elastic.co/downloads/kibana
