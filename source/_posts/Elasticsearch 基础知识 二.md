---
title: Elasticsearch 基础知识 二
tags:
  - elasticsearch
  - es
id: 283
categories:
  - 笔记
date: 2017-07-16 18:16:12
---

## 1. 概念
* 节点：一个运行中的 Elasticsearch 实例
* 主节点：选举产生的。它负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。任何节点都可以成为主节点。
* 集群：由一个或者多个拥有相同 cluster.name 配置的节点组成。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。
* 集群健康：`status`字段表示集群健康度，通过`GET /_cluster/health`查看
* 索引：索引实际上是指向一个或者多个物理 分片 的 逻辑命名空间
* 分片：一个 分片 是一个底层的 工作单元 ，它仅保存了 全部数据中的一部分。一个分片是一个 Lucene 的实例。它本身就是一个完整的搜索引擎。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。
* 主分片：索引内任意一个文档都归属于一个主分片。
* 副本分片：一个副本分片只是一个主分片的拷贝。
* 元数据： 有关文档的信息。`_index`文档在哪存放,`_type`文档表示的对象类别,`_id`文档唯一标识
<!--more-->


## 2. 操作
### 2.1. 创建一个名为`blogs`的索引（索引在默认情况下会被分配5个主分片），该索引将分配3个主分片和1份副本：
```shell
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```
### 2.2. 修改`blogs`的默认副本数：
```shell
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```
### 2.3. 自定义ID
```shell
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```
### 2.4. 自动生成ID
```shell
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```
自动生成的唯一ID:`"_id": "AV1Kxv0cIHSGtXe9SZfO",`
### 2.5. 只返回文档的一部分
`GET /website/blog/123?_source=title,text`
### 2.6. 只获取_source
`GET /website/blog/123/_source`
### 2.7. 判断文档是否存在
`curl -i -XHEAD http://localhost:9200/website/blog/123`
如果文档存在， Elasticsearch 将返回一个 200 ok 的状态码
若文档不存在， Elasticsearch 将返回一个 404 Not Found 的状态码
### 2.8. 更新文档
在 Elasticsearch 中文档是 不可改变 的，不能修改它们。 相反，如果想要更新现有的文档，需要 重建索引 或者进行替换。
实际上 Elasticsearch 按前述完全相同方式执行以下过程：
1. 从旧文档构建 JSON
2. 更改该 JSON
3. 删除旧文档
4. 索引一个新文档

### 2.8. PUT if absent
```shell
PUT /website/blog/123?op_type=create
{ ... }
```
```shell
PUT /website/blog/123/_create
{ ... }
```
如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 201 Created 的 HTTP 响应码。
### 2.9. 删除文档
`DELETE /website/blog/123`
删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。
### 2.10. 指定版本号`_version`修改
```shell
PUT /website/blog/1?version=1
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
如果版本号已经变更，Elasticsearch 返回 409 Conflict HTTP 响应码
### 2.11. 使用外部版本号
```shell
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```
### 2.12. 更新指定字段
```shell
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```
该操作的内部实现和`PUT`请求的内部实现是一样的，也是*检索-修改-重建索引* 的处理过程，区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。
### 2.13. 使用脚本更新文档
```shell
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```
`_source`的字段内容， 它在更新脚本中称为 `ctx._source` 。
### 2.14. 批量获取文档
```shell
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```

### 2.15. 批量操作bulk
```shell
{ action: { metadata }}n
{ request body        }n
{ action: { metadata }}n
{ request body        }n
...
```
* 每行一定要以换行符(\n)结尾， 包括最后一行 。这些换行符被用作一个标记，可以有效分隔行。
* 这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON 不 能使用 pretty 参数打印。
例子：
```shell
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} }
```
每个子请求都是独立执行，因此某个子请求的失败不会对其他子请求的成功与否造成影响。
一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。