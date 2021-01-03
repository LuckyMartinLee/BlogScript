---
title: Elasticsearch从入门到放弃(一) -- 基本概念
date: 2018-01-26 15:23:08
tags:
- Elasticsearch
---

## 相关概念

### 通用搜索 与 垂直搜索
&ensp;&ensp;通用搜索，通俗点说就是在网络上的所有可以获取的信息中进行搜索，涉及所有领域，所有展现形式，包括音频、视频、文字、图片。
一句话就是只要信息内容匹配你的搜索条件，就返回给你，知名站点有 Baidu, Google, Bing.
&ensp;&ensp;垂直搜索，相对于通用搜索，它是针对特定行业信息的搜索。比如，用于图标搜索的[EasyIcon](http://www.easyicon.cn/)、
小说搜索[Owllook ](https://www.owllook.net/)等。

### 全文检索 与 倒排索引
&ensp;&ensp;全文检索，概念就是通过计算机实现，你的搜索词出现在文章当中，甚至搜索词与文章有相关性，那么这篇文章就应该被搜索出来。计算机实现这个功能整个过程叫做全文检索。
&ensp;&ensp;倒排索引，是实现全文检索的技术手段之一。简单的说就是把文章拆解成一个个简单的词语（此过程叫分词），将文章与这些词语建立起关联关系（即是索引）。用户查询时，也讲搜索词进行分词，再在前面建立的关联关系中查找文章，以此实现全文检索。想更进一步知道什么倒排索引的实现，请看我的另一篇文章《[什么是倒排索引](/2017/04/05/inverted_index1/)》

### Lucene
&ensp;&ensp;Lucene，是一个开源免费的成熟的Java系的信息检索程序库，全文检索引擎工具包，由Apache软件基金会支持和提供。严格的说，Lucene 不是全文检索引擎或者搜索引擎，它提供了查询组件，索引组件以及文本分析组件，它是一个全文检索引擎的框架。 

## Elasticsearch
### 何为 Elasticsearch
&ensp;&ensp;Elasticsearch，是一款基于Lucene的开源的高扩展的分布式全文检索引擎。Elastic 是 Lucene 的封装，提供了 RESTFull API 的操作接口，开箱即用。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。
[![](/post_imgs/elastic-logo.png)](http://www.elastic.co/products/elasticsearch)

### Elasticsearch 核心概念
#### Near Realtime (NRT)
&ensp;&ensp;近实时概念包含两层含义，一是数据从写入到可搜索到，时间达到秒级别；二是Elasticsearch 查询、聚合、分析，时间到达秒级别。
#### Cluster
&ensp;&ensp;Elasticsearch是一个分布式的全文检索引擎，多个程序节点构成一个大的集群，集群中节点即是数据备份节点，也是数据查询分析负载均衡节点。
#### Node
&ensp;&ensp;节点，即是一个Elasicsearch的实例，一台机器可以运行一个或多个节点。
#### Shard(primary shard)
&ensp;&ensp;分片是为了解决海量数据存储问题，Elasticsearch 使用分片机制，将海量数据切分为多个分片存储在不同的节点上。Elasticseach分片就是它的主分片，Elasticsearch默认主分片数量是5。
#### replica(replica shard)
&ensp;&ensp;分片副本，意思显而易见，就是分片的备份，作用是提高集群数据安全，提升系统高可用性，同时副本节点在海量数据检索时也分担检索压力，具备提升 Elasticsearch 请求吞吐量和性能作用。
#### Index
&ensp;&ensp;这里的 Index 不是查询索引的概念，可以理解为同类型数据的库，相当于 MySQL 中的库，但又有不同，不同之处在于此处的Index中存储的都是字段类型基本一致的同类型数据。
#### Type
&ensp;&ensp;这里的 Type ，可以理解为 MySQL 中的表，一个 Index 中可以有多个 Type，且一个 Type存储同种数据。如，一个名为 animal 的 Index 中有 bird 和 fish 两个 Type, 他们有很多共同的属性。
#### Document
&ensp;&ensp;Document，即是 Type 中具体的文档。每个文档都有自己唯一的 id.
#### Field
&ensp;&ensp;Field，是文档的属性或者叫做字段，不同字段，类型不同，分词方式也不同。



























