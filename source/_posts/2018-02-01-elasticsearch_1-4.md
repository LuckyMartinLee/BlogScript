---
title: Elasticsearch 查询
date: 2018-02-01 15:41:28
tags:
- Elasticsearch
---

1. ## 基本概念

   <br/>

   ### Cluster
   Elasticsearch是一个分布式的全文检索引擎，多个程序节点构成一个大的集群，集群中节点即是数据备份节点，也是数据查询分析负载均衡节点。

   <br/>

   ### Node
   节点，即是一个Elasicsearch的实例，一台机器可以运行一个或多个节点。

   <br/>

   ### Shard(primary shard)
   分片是为了解决海量数据存储问题，Elasticsearch 使用分片机制，将海量数据切分为多个分片存储在不同的节点上。Elasticseach分片就是它的主分片，Elasticsearch默认主分片数量是5。

   <br/>

   ### replica(replica shard)
   分片副本，意思显而易见，就是分片的备份，作用是提高集群数据安全，提升系统高可用性，同时副本节点在海量数据检索时也分担检索压力，具备提升 Elasticsearch 请求吞吐量和性能作用。

   <br/>

   ### Index
   这里的 Index 不是查询索引的概念，可以理解为同类型数据的库，相当于 MySQL 中的库，但又有不同，不同之处在于此处的Index中存储的都是字段类型基本一致的同类型数据。

   <br/>

   ### Type
   这里的 Type ，可以理解为 MySQL 中的表，一个 Index 中可以有多个 Type，且一个 Type存储同种数据。如，一个名为 animal 的 Index 中有 bird 和 fish 两个 Type, 他们有很多共同的属性。在 ES 早期版本，一个Index下是可以有多个 Type ，从 7.0 开始，一个Index只有一个 Type。

   <br/>

   ### Mapping
   Mapping 类似于数据库中的表结构定义 schema，它有以下几个作用：
   定义索引中的字段的名称
   定义字段的数据类型，比如字符串、数字、布尔
   字段，倒排索引的相关配置，比如设置某个字段为不被索引、记录 position 等

   <br/>

   ### Document
   Document，即是 Type 中具体的文档。每个文档都有自己唯一的 id.

   <br/>

   ### Field
   Field，是文档的属性或者叫做字段，不同字段，类型不同，分词方式也不同。

   <br/>

   ## 基本概念与Mysql 对比

   | MySql              | ES                            |
   | ------------------ | ----------------------------- |
   | table（表）        | index（索引）                 |
   | row （一条记录）   | document（文档）              |
   | field （列）       | field（一个字段）             |
   | schema（创建语句） | Mapping（字段和类型定义配置） |
   | sql （查询语言）   | DSL（查询语言）               |

   <br/>

   ## DSL查询
   ``` json
   GET /sunny/user/_search
   {
       "from":0, // 偏移量
       "size":1, // 返回 documents 梳理
       "query": { 
           "bool": { 
               "must": [
                   { "match": { "title":   "Search"        }}, 
                   { "match": { "content": "Elasticsearch" }}  
               ],
               "filter": [ 
                   { "term":  { "status": "published" }}, 
                   { "range": { "publish_date": { "gte": "2015-01-01" }}} 
                ]
           }
       },
       "highlight": { // 高亮
          "fields" : {
              "car" : {}
          }
       },
       "aggs" : { // 聚合查询语句的简写
           "popular_colors" : { // 给聚合查询取个名字，叫popular_colors
               "terms" : { // 聚合类型为，terms，terms是桶聚合的一种，类似SQL的group by的作用，根据字段分组，相同字段值的文档分为一组。
                 "field" : "color" // terms聚合类型的参数，这里需要设置分组的字段为color，根据color分组
               }
           }
       }
   
   }
   ```

   <br/>

   ### Query 查询

   <br/>

   #### term 与 terms 查询
   ``` json
   // term 会去倒排索引中寻找确切的term，它并不知道分词器的存在
   {
   	"query":{
   	    "term":{
   		    "preview":"elasticsearch"
   		}
   	}
   }
   
   {
   	"query": {
   	    "terms": {
   		    "preview": ["elasticsearch","book"]
   		    "miniumu_match":2
   		}
   	}
   }
   
   ```

   #### match 查询
   ``` json
   //  知道分词器的存在，会对field进行分词操作，然后再查询
   // match
   {
     "query": {
       "match": {
         "title":  "my ss"   //它和term区别可以理解为term是精确查询，这边match模糊查询；match会对my ss分词为两个单词，然后term对认为这是一个单词
       }
     }
   }
   
   
   // multi_match
   {
     "query": {
       "multi_match": {
         "query": "宝马发动机多少",
         "type": "best_fields", // 我们希望完全匹配的文档占的评分比较高，则需要使用best_fields,默认
         // "type": "most_fields", // 我们希望越多字段匹配的文档评分越高，就要使用most_fields
         // "type": "cross_fields", //我们会希望这个词条的分词词汇是分配到不同字段中的，那么就使用cross_fields
         "fields": [
           "tag",
           "content"
         ]
       }
     }
   }
   
   // match_phrase 必须匹配短语中的所有分词，并且保证各个分词的相对位置不变
   {
     "query": {
       "match_phrase": {
           "content" : {
               "query" : "宝马多少马力",
               "slop" : 4 // 分词之间的最大变换距离
           }
       }
     }
   }
   
   ```
   #### prefix 查询
   ``` json
   {
   	"query":{
   		"prefix":{
   		    "title":{
   		        "value":"r"      // 匹配title 字段 r开头的
   		    }
   	    }
       }
   }
   ```

   

   #### range 查询
   ``` json
   {
   	"query":{
   	    "range":{
   	        "publish_date":{      // 时间字段
   	            "from":"2015-01-01",
   	            "to":"2015-06-30"
   	        }
           }
       }
   }
   
   {
   	"query":{
   	    "range":{
   	        "price":{       // 价格字段
   	            "from":"10",
   	            "to":"20",
   	            "include_lower": true,   // 意思是包括 from
   	            "include_upper":false    // 不包括 to
   	        }
           }
       }
   }
   
   ```

   #### sort 排序
   ``` json
   {
       "query":{
           "match_all":{}
       },
       "sort":[
       {
           "comments":{
               "order":"desc"
           }
       }
       ]
   }
   ```


   #### bool 查询
   ``` json
   bool:{
   	"filter":[],   // 字段过滤
   	"must":[],     // 所有查询条件都满足
   	"should":[],   // 满足一个或多个
   	"must_not":{}  // 都不满足于must相反
   }
   
   {
       "bool" : {
           "must" : {
               "term" : { "user" : "kimchy" }
           },
           "filter": {
               "term" : { "tag" : "tech" }
           },
           "must_not" : {
               "range" : {
                   "age" : { "from" : 10, "to" : 20 }
               }
           },
           "should" : [
               {
                   "term" : { "tag" : "wow" }
               },
               {
                   "term" : { "tag" : "elasticsearch" }
               }
           ],
           "minimum_should_match" : 1,
           "boost" : 1.0
       }
   }
   
   // bool 查询嵌套
   {
       "query":{
           "bool":{
               "should":[
                   {"term":{"title":"python"}},
                   {
                   "bool":{
                       "must":[
                           {"term":{"title":"django"}},
                           {"term":{"salary":30}}
                       ]
                   }
                   }
               ]
           }
       }
   }
   
   ```

   ### Aggs 聚合统计
   ``` json
   // 基本结构
   {
       "aggregations" : {                                //定义聚合对象,也可用 "aggs"
         "<aggregation_name>" : {                    //聚合的名称,用户自定义
               "<aggregation_type>" : {                //聚合类型,比如 "histogram"
                   <aggregation_body>                  //每个聚合类型都有其自己的结构定义
               },
               "meta" : {  [<meta_data_body>] },
               "aggregations" : { [<sub_aggregation>]+ }   //可以定义多个 sub-aggregation
           },
           "<aggregation_name_2>" : { ... }        //定义额外的多个平级 aggregation,只有 Bucketing 类型才有意义
   }
   
   // Bucket：分桶类型，类似SQL中的GROUP BY语法
   // Metric：指标分析类型，如计算最大值、最小值、平均值等等
   // Pipeline：管道分析类型，基于上一级的聚合分析结果进行在分析
   // Matrix：矩阵分析类型，实验性质
   
   {
       "aggs" : {
           "sales_per_month" : {
               "date_histogram" : { // bucket 聚合，按照月份进行分桶，每个月的归属一个桶
                   "field" : "date",
                   "interval" : "month"
               },
               "aggs": {
                   "sales": {
                       "sum": { // metric 聚合，对每个桶类的 price 求和，即每月的销售额
                           "field": "price"
                       }
                   }
               }
           },
           "max_monthly_sales": {
               "max_bucket": { // pipeline 聚合，求所有桶中销售额 sales 最大的值
                   "buckets_path": "sales_per_month>sales" 
               }
           }
       }
   }
   ```


   #### Bucketing 桶分聚合

   Bucket：意为桶，即按照一定的规则将文档分配到不同的桶中，达到分类分析的目的
   Terms 
   <br/>
   Range
   <br/>
   Date Range
   <br/>
   Histogram
   <br/>
   Date Histogram


   ##### 基于 Terms 分桶
   ``` json
   // 最简单的分桶策略，直接按照term来分桶，如果是text类型，则按照分词后的结果分桶
   {
     "aggs": {
       "group_by_gender": {
         "terms": {
           "field": "keyword"
         }
       }
     }
   }    
   ```

   ##### 基于 Range、Date Range 分桶
   ``` json
   // Range: 通过制定数值的范围来设定分桶规则
   
   // ranges：配置区间，数组，每一个元素是一个区间。例如：[{from:0}, {from:50, to:100}, {to:200}],包括 from 值，不包括 to 值（区间前闭后开）
   // keyed：以一个关联的唯一字符串作为键，以 HASH 形式返回，而不是默认的数组
   // script：利用 script 执行结果替代普通的 field 值进行聚合。script可以用file给出，还可以对其它 field 进行求值计算。
   
   {
       "aggs" : {
           "price_ranges" : {
               "range" : {
                   "field" : "price",
                   "ranges" : [ //包含 3 个桶
                       { "to" : 50 },
                       { "from" : 50, "to" : 100 },
                       { "from" : 100 }
                   ],
                   "keyed" : true
               }
           }
       }
   }
   
   // Date Range: 通过指定日期的范围来设定分桶规则
   {
       "aggs": {
           "range": {
               "date_range": {
                   "field": "date",
                   "format": "MM-yyy",               
                   "ranges": [ //包含 3 个桶
                       { "to": "now-10M/M" }, 
                       { "from": "now-10M/M" },
                       {"from":"1970-1-1", "to":"2000-1-1"}
                   ]
               }
           }
       }
   }
   ```


   ##### 基于 Historgram、Date Histogram 分桶

   ``` json
   // Historgram 直方图，以固定间隔的策略来分割数据
   // field：字段，必须为数值类型
   // interval：分桶间距
   // min_doc_count：最少文档数桶过滤，只有不少于这么多文档的桶才会返回
   // extended_bounds：范围扩展
   // order：对桶排序，如果 histogram 聚合有一个权值聚合类型的"直接"子聚合，那么排序可以使用子聚合中的结果
   // offset：桶边界位移，默认从0开始
   // keyed：hash结构返回，默认以数组形式返回每一个桶
   // missing：配置缺省默认值
   
   {
       "aggs" : {
           "prices" : {
               "histogram" : {
                   "field" : "price",
                   "interval" : 50,
                   "min_doc_count" : 1,
                   "extended_bounds" : {
                       "min" : 0,
                       "max" : 500
                   },
                   "order" : { "_count" : "desc" },
                   "keyed":true,
                   "missing":0
               }
           }
       }
   }
   // Date Histogram 针对日期的直方图或者柱状图，是时序分析中常用的聚合分析类型
   // field：
   // interval：时间间隔类型为：year、quarter、month、week、day、hour、minute、second，其中，除了year、quarter 和 month，其余可用小数形式
   // format：定义日期的格式，配置后会返回一个 key_as_string 的字符串类型日期（默认只有key）
   // time_zone：定义时区，用作时间值的调整
   // offset：
   // missing：
   
   {
       "aggs" : {
           "articles_over_time" : {
               "date_histogram" : {
                   "field" : "date",
                   "interval" : "month",
                   "format" : "yyyy-MM-dd",
                   "time_zone": "+08:00"
               }
           }
       }
   }
   ```

   数值范围聚合——基于某个值（可以是 field 或 script），以【字段范围】来桶分聚合。


   #### Metric 指标聚合

   ``` json
   // 单值分析，只输出一个分析结果
   min,max,avg,sum
   cardinality //指不同数值的个数，类似SQL中的distinct count概念
   {
     "aggs": {
       "min_age": {
         "min": {
           "field": "age"
         }
       },
       "avg_age":{
         "avg":{
           "field":"age"
         }
       },
       "max_age":{
         "max":{
           "field":"age"
         }
       }
     }
   }  
   
   
   // 多值分析，输出多个分析结果
   stats, // 返回一系列数值类型的统计值，包含min、max、avg、sum和count
   extended stats, // 对stats的扩展，包含了更多的统计数据，比如方差、标准差等
   percentile,percentile rank
   top hits, // 一般用于分桶后获取该桶内匹配的顶部文档列表，即详情数据.例如，按照性别进行分组，并对每组中按照balance进行排序（子聚合）
   {
     "aggs": {
       "group_by_gender": {
         "terms": {
           "field": "gender"
         },
         "aggs": {
           "top_employee": {
             "top_hits": {
               "size": 2,
               "_source": ["gender","balance"], 
               "sort": [
                 {
                   "balance": {
                     "order": "desc"
                   }
                 }
                 ]
             }
           }
         }
       }
     }
   }
   
   ```

   #### Pipeline 管道聚合

   对其它聚合操作的输出及其关联指标进行聚合。
   此类聚合的作用对象往往是桶，而不是文档，是一种后期对每个分桶的一些计算操作。
   ``` json
   
   Max/Min/Avg/Sum Bucket
   Stats/Extended Stats Bucket
   Percentiles Bucket
   
   {
       "size": 0,
       "aggs": {
           "group_by_stats": {
               "terms": {
                   "field": "state.keyword"
                },
               "aggs": {
                   "average_balance": {
                       "avg": {
                           "field": "balance"
                           }
                       }
               }
           },
           "min_avg_by_balance":{
               "min_bucket": {
                   "buckets_path": "group_by_stats>average_balance"
               }
           }
       }
   }
   
   Derivative, //      导数求导
   Moving Average, //  移动平均
   Cumulative Sum, //  累计求和
   {
     "size":0,
     "aggs": {
       "hist_city": {
         "histogram": {
           "field": "age",
           "interval": 5
         },
         "aggs":{
           "avg_balance":{
             "avg": {
               "field": "balance"
             }
           },
           "derivative_avg_balance":{
             "derivative": {
               "buckets_path": "avg_balance"
             }
           }
         }
       }
     }
   }
   ```