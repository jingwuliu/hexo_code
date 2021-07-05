---
title: Elasticsearch原理介绍
date: 2021-03-23 17:33:26
tags: [Elasticsearch,倒排索引]
---

由于工作中项目需要更换database，所以也开始去了解一些elasticsearch的基础知识。

总的来说，Elasticsearch是一个分布式的存储和搜索引擎，适用于包括文本、数字、结构化和非结构化数据等所有类型的数据，[官网速递](https://www.elastic.co/cn/)。
首先我们要了解一种索引方式，叫倒排索引。顾名思义，和我们平时生活中常用的索引方式不太一样。就拿背古诗来举例，对于普通的索引方式，往往是以(key-诗名 value-诗文)的方式进行存储和索引的。 
![常规key-value示意图](key-value.png)
但是当我们想要通过关键词去寻找相关诗句时，如寻找带有"前"的诗句，由于缺少相关的索引，所以我们只能遍历记忆中的所有诗词，难以在短时间内得到结果，这时就需要我们的倒排索引了。倒排索引又叫反向索引。倒排索引往往会以类似于诗句中的每个字作为索引的key，而诗句内容作为value。即如下方法。
![倒排索引key-value示意图](new-key-value.png)
针对这样的一句诗句，我们就会建立10个索引(诗句中10个字)。相比较而言，原来的正向索引只需要一个索引，而反向需要10个，这样可能会导致我们的存储量爆炸增长。所以我们考虑将数据进行压缩。将key-value的对应更改为(key-关键词 value-诗名)的形式。
![倒排索引key-value新示意图](new-key-value3.jpg)
这样的方式就可以保证减少数据量。相应的，我们也需要形成(key-诗名 value-诗句)的索引矩阵帮助我们寻找。
![倒排索引key-value新示意图4](new-key-value4.png)
可以看到，静夜思和望庐山瀑布中都包含“前”字，而静夜思、月下独酌中都有“月”字，所以我们在对于多首诗建立索引时也可以这样。
![倒排索引key-value新示意图5](new-key-value5.png)
我们日常生活中使用的搜索引擎的原理也是这样的，最核心的都是建立倒排索引。只不过它们的流程稍微复杂一些，还包括网页爬取、停顿词过滤等。停顿词过滤会帮助引擎过滤掉一些诸如“的”、“而”等本身无意义的停顿词，即分词，然后再根据剩余的关键词进行索引。
在很早以前，业界有一个叫做**lucene**的库，用来实现倒排索引，后来人们对于lucene进行再次封装，写出了**Elasticsearch**。
Elasticsearch将对搜索引擎的操作都封装成了restful的api，通过http请求就可以对其进行操作。想要了解Elasticsearch，我们需要先了解几个专有名词。
#### 索引
这里说的索引和刚刚说的key索引不是一个概念，elasticsearch中的索引是存放数据的地方，可以理解成mysql中的数据库。
#### 类型
类型就是用来定义数据结构的，可以认为是mysql中的一张表。
#### 文档
文档可以被认为就是最终的数据了，可以理解成我们每次需要上传的数据都是文档，即一条记录。
![elasticsearch-mysql](elasticsearch-mysql.png)
#### 数据存储方式
比如一首诗，有诗题、作者、朝代、字数、诗内容等，首先我们可以建立一个名为poems的索引，诗题、作者、朝代都是keyword类型，而诗内容是text类型，字数是interger类型，最后把数据组织成json格式存放。
```
index
poems

type
“poem”: { 
    "properties": {
        "title": {
            "type": "keyword",
        }
    },
    "author": {
        "type": "keyword",
    },
    "dynasty": {
        "type": "keyword"
    },
    "words": {
        "type": "interger"
    },
    "content": {
        "type": "text"
    }
}

document
{
    "title": "静夜思",
    "author": "李白",
    "dynasty": "唐",
    "words": "20",
    "content": "床前明月光，疑是地上霜，举头望明月，低头思故乡。"
}
```

类型相当于表结构的描述，描述每个字段的类型，文档以json形式描述数据。针对上述数据类型，keyword类型是不会分词的，直接根据字符串内容建立反向索引，而text类型存入elasticsearch前会先分词，再根据分词后内容建立倒排索引。而我们只需要给elasticsearch发送http请求就可以成功建立索引。


#### Elasticsearch分布式原理
elasticsearch也会对数据进行切分，同时每一个分片会保存多个副本，从而保证分布式环境下的高可用。
![elastic-store](elastic-store.png)
图中绿色的表示数据块，也可以成为分片(shard)，它保存了索引中所有数据的一部分，即部分数据的容器。文档存放在分片中，分片会被分配到集群的节点上，当集群扩容或缩小，Elasticsearch也会自动在节点间迁移分片，以使集群保持平衡。分片也会被备份生成复制分片存储到多个节点中，每个节点是对等的，节点会通过一些规则选取集群的Master，Master会负责集群状态信息的改变，并同步给其它节点。


#### Elasticsearch实战搜索
假设我们有这样一个集群，我们将数据切分为两块，分别为P0、P1，将P0、P1分别生成两个复制分片并存放于不同的节点上，如图所示。
![elastic-search](elastic-search.png)
假设我们想要去对文档进行索引，索引步骤一般如下：
- 客户端给集群中的Master(假设为Node1)发送GET请求。
- 节点使用文档提供的_id确定需要被索引的文档包含在分片0中，分片0对应的复制分片在三个节点上都有。此时，它转发请求到Node2。
- Node2返回文档给Node1后将检索结果返回给客户端。

##### 索引结果字段说明
最基本的索引就是空搜索，这里不会只当任何的查询条件，这样会返回集群索引中的所有文档。
```
    GET /_search
```
响应内容往往如下：
```
    {
        "hits" : {
            "total" :       14,
            "hits" : [
            {
                "_index":   "us",
                "_type":    "tweet",
                "_id":      "7",
                "_score":   1,
                "_source": {
                    "date":    "2014-09-17",
                    "name":    "John Smith",
                    "tweet":   "The Query DSL is really powerful and flexible",
                    "user_id": 2
                }
            },
            ... 9 RESULTS REMOVED ...
            ],
            "max_score" :   1
        },
        "took" :           4,
        "_shards" : {
            "failed" :      0,
            "successful" :  10,
            "total" :       10
        },
        "timed_out" :      false
    }
```

- **hits**:hits包含了total字段来表示匹配到的文档总数，hits数组还会包含匹配到的前10条数据。
hits数组中的每个结果都包含_index、_type和文档的_id字段，被加入到_source字段中这意味着在搜索结果中我们将可以直接使用全部文档。这不像其他搜索引擎只返回文档ID，需要你单独去获取文档。
每个节点都有一个_score字段作为相关性得分，它衡量了文档与索引的匹配程度。默认的，返回的结果中关联性最大的文档排在首位；由于我们没有指定任何查询，所有文档的相关性是一样的，因此所有的_score都是取得中间值1。
max_score指的是所有文档匹配查询中_score的最大值。

- **took**:索引请求花费的毫秒数。

- **shards**:参与索引的分片数量，有多少是成功和失败的。

- **timeout**:返回查询超时与否。timeout不会停止执行索引，仅会告诉在指定timeout内顺利返回结果的节点然后关闭连接。在后台依然可以执行查询。


#### 搜索引擎原理总结

- 反向索引又叫倒排索引，是根据文章内容中的关键字建立索引。
- 搜索引擎原理就是建立反向索引。
- Elasticsearch 在 Lucene 的基础上进行封装，实现了分布式搜索引擎。
- Elasticsearch 中的索引、类型和文档的概念比较重要，类似于 MySQL 中的数据库、表和行。
- Elasticsearch 也是 Master-slave 架构，也实现了数据的分片和备份。

开发视图用于描述系统的模块划分和组成，以及细化到内部包的组成设计，服务于开发人员，反映系统开发实施过程。

本文内容参考于[终于有人把Elasticsearch原理讲透了！](https://zhuanlan.zhihu.com/p/62892586), [Elasticsearch系列(六)ES数据搜索之基本流程](https://blog.csdn.net/u012834750/article/details/88045861)