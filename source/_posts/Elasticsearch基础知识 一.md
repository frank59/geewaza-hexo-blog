---
title: Elasticsearch基础知识 一
tags:
  - elasticsearch
  - es
id: 279
categories:
  - 笔记
date: 2017-07-16 17:11:30
---

## 1. 安装Elasticsearch
[Elasticsearch2.4.2下载地址](https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/zip/elasticsearch/2.4.2/elasticsearch-2.4.2.zip
)
```shell
unzip elasticsearch-2.4.2.zip
cd elasticsearch-2.4.2
./bin/elasticsearch
```
在新的控制台访问：
```shell
curl 'http://localhost:9200/?pretty'
```
<!--more-->


[下载Kibana4.5.4](https://download.elastic.co/kibana/kibana/kibana-4.5.4-darwin-x64.tar.gz
)
```shell
tar -xvf kibana-4.5.4-darwin-x64.tar.gz
cd kibana-4.5.4-darwin-x64
./bin/kibana plugin --install elastic/sense
./bin/kibana
```
在浏览器访问地址
[http://localhost:5601/app/sense](http://localhost:5601/app/sense)

## 2. 测试elasticsearch
在保持es和kibana都启动的情况下，访问es：
```shell
curl -i -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```

## 3. 知识点
1. 面向文档：对文档进行索引、检索、排序和过滤
2. 使用JSON格式化文档
3. 集群：一个 Elasticsearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型

## 4. 索引操作
### 4.1. 插入文档
```shell
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```
说明：`/megacorp/employee/1` --> `索引名称\类型名称\该条doc的id`

### 4.2. 检索
指定检索某个id的文档
```shell
GET /megacorp/employee/1
```
获取全部记录文档
```shell
GET /megacorp/employee/_search
```
条件检索：
```shell
GET /megacorp/employee/_search?q=last_name:Smith
```
### 4.3. 其他操作
`DELETE` 删除
`HEAD` 查看是否存在某条文档
## 5. 查询表达式
### 5.1. JSON格式的查询表达式
```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```
这个查询与`GET /megacorp/employee/_search?q=last_name:Smith`查询效果一致。
### 5.2. 复杂的查询语句：
```shell
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            }
        }
    }
}
```
这条语句翻译成sql：`select * from employee where last_name='smith' and age > 40`
### 5.3. 全文搜索
指定字段模糊查询（默认按照相关性得分排序）：
```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
将查询条件视为一个完整的短语进行模糊匹配（不再分词匹配）：
```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```
增加高亮词的处理：
```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```
查询结果会多一个数据块：
```javascript
...
"highlight": {
   "about": [
      "I love to go <em>rock</em> <em>climbing</em>"
   ]
}
...
```
### 6. 分析结果
分析统计雇员的爱好（interests）字段的值：
```shell
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```
分级汇总（查询特定兴趣爱好员工的平均年龄）
```shell
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```
