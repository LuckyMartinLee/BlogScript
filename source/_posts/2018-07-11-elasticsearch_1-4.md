---
title: Elasticsearch从入门到放弃(三) -- 查询一
date: 2018-07-11 15:11:21
tags:
- Elasticsearch
---

## 查询方式
1 使用Search Lite API，并将所有的搜索参数都通过URL传递
``` bash
// 下面是在所有的字段中搜索带有"John"的结果，相当于指定 _all 字段
curl -XGET 'localhost:9200/megacorp/employee/_search?q=John'

// 指定地段查询， 如指定 interests 字段
curl -XGET 'localhost:9200/megacorp/employee/_search?q=interests:music'
```

2 使用Elasticsearch DSL，其可以通过传递一个JSON请求来获取结果。
DSL方式提供了更加灵活的方式来构建更加复杂的查询（我们将在后面看到），甚至指定你想要的返回结果
``` bash
// 等价于上面的 在所有的字段中搜索带有"John"的结果
curl -XGET 'localhost:9200/megacorp/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "John",
            "fields" : ["_all"]
        }
    }
}'
```


## 基本查询

term查询和terms查询
不进行分词查询，直接去倒排索引中匹配确切的term。
term:查询某个字段里含有某个关键词的文档
terms:查询某个字段里含有多个关键词的文档，关键词之间是或关系
``` bash
get /lib3/user/_search/
{
  "query":{"term":{ "interests":"youyong"}}
}
 
get lib3/user/_search/
{
  "query":{"terms":{"interests":["shufa","youyong"]}}
}
```

match查询
对查询关键字进行查询分词
match:先进行分词操作，然后再查询

match_all:查询所有文档

match_phrase:短语匹配查询，可以指定slop分词间隔多远。相隔多远的意思是，你需要移动一个词条多少次来让查询和文档匹配。

match_phrase_prefix: 前缀查询，根据短语中最后一个分词来做前缀匹配，注意和max_expanions搭配。其实默认是50

multi_match：多字段查询，使用相当的灵活，可以完成match_phrase和match_phrase_prefix的工作。
``` bash
GET lib3/user/_search
{
  "query":{"match":{"age": 20}}
}

// match_all的值为空，表示没有查询条件，那就是查询全部。就像select * from table_name一样
GET lib3/user/_search
{
  "query":{
    "match_all": {}
  }
}
 
 
get lib3/user/_search
{
  "query":{
    "match_phrase":{"interests": "youyong shufa","slop": 2}
  }
}


get lib3/user/_search
{
  "query":{
    "match_phrase_prefix":{"interests": "you","max_expansions": 1}
  }
}

GET lib3/user/_search
{
  "query":{
    "multi_match": {
      "query": "youyong",
      "fields":["interests","name"]
    }
  }
}
// multi_match 实现 match_phrase_prefix 功能
GET lib3/user/_search
{
  "query": {
    "multi_match": {
      "query": "gi",
      "fields": ["title"],
      "type": "phrase_prefix"
    }
  }
}

// multi_match 实现 match_phrase功能
GET lib3/user/_search
{
  "query": {
    "multi_match": {
      "query": "girl",
      "fields": ["title"],
      "type": "phrase"
    }
  }
}

GET lib3/user/_search
{
   "multi_match" : {
      "query" : "北京天安门",
      // type 默认是best_field，词条匹配度越高，得分越高，此外还有
      // most_fields, 词条命中数量越多，得分越高
      // cross_fields, 字段命中的越多，得分越高
      "type" : "best_fields", 
      // 词条之间 逻辑关系，默认是 or
      "operator": "and",
      //
      "tie_breaker" : 0.3,
      "fields" : [ "title", "body" ],
      "minimun_should_match" : "30%" // 至少要有30%的词条被搜索到，才命中,对most_fields 无效
   }
}

```

Boosting 字段权重调整
如搜索请求在多个field中查询，想提高某个field的查询权重,下面的例子中，我们把interests的权重调成3，这样就提高了其在结果中的权重。
Boosting不仅仅意味着计算出来的分数(calculated score)直接乘以boost factor，最终的boost value会经过归一化以及其他一些内部的优化。
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "rock",
            "fields": ["about", "interests^3"]
        }
    }
}'
```

Bool Query  布尔查询
布尔查询可以接受一个must参数(等价于AND)，一个must_not参数(等价于NOT)，以及一个should参数(等价于OR)。比如，我想查询about中出现music或者climb关键字的员工，员工的名字是John，但姓氏不是smith，我们可以这么来查询：
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "bool": {
                "must": {
                    "bool" : { 
                        "should": [
                            { "match": { "about": "music" }},
                            { "match": { "about": "climb" }} ] 
                    }
                },
                "must": {
                    "match": { "first_nale": "John" }
                },
                "must_not": {
                    "match": {"last_name": "Smith" }
                }
            }
    }
}'
```

Fuzzy Queries 模糊查询
模糊查询可以在Match和 Multi-Match查询中使用以便解决拼写的错误，模糊度是基于 Levenshtein Edit Distance 计算与原单词的距离。使用如下：
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "rock climb",
            "fields": ["about", "interests"],
            "fuzziness": "AUTO"
        }
    },
    "_source": ["about", "interests", "first_name"],
    "size": 1
}'
```
fuzziness 参数的取值如下:
1) 0,1,2
表示最大可允许的莱文斯坦距离

2) AUTO
会根据词项的长度来产生可编辑距离，它还有两个可选参数，形式为AUTO:[low],[high]， 分别表示短距离参数和长距离参数；如果没有指定，默认值是 AUTO:3,6 表示的意义如下

	2.1) 0..2
	单词长度为 0 到 2 之间时必须要精确匹配，这其实很好理解，单词长度太短是没有相似度可言的，例如 'a' 和 'b'。

	2.2) 3..5
	单词长度 3 到 5 个字母时，最大编辑距离为 1

	2.3) >5
	单词长度大于 5 个字母时，最大编辑距离为 2

如果不设置 fuziness 参数，查询是精确匹配的。
fuzziness 在绝大多数场合都应该设置成 AUTO




Wildcard Query 通配符查询
通配符查询允许我们指定一个模式来匹配，而不需要指定完整的trem。
? 将会匹配如何字符；
* 将会匹配零个或者多个字符。
如想查找所有名字中以J字符开始的记录，我们可以如下使用：
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
            "wildcard" : {
                "first_name" : "s*"
            }
        },
        "_source": ["first_name", "last_name"],
    "highlight": {
            "fields" : {
                "first_name" : {}
            }
        }
}'
```
 
Regexp Query 正则表达式查询
如查找作者名字以J字符开头，中间是若干个a-z之间的字符，并且以字符n结束的记录，可以如下查询：
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "regexp" : {
            "first_name" : "J[a-z]*n"
        }
    },
    "_source": ["first_name", "age"],
    "highlight": {
        "fields" : {
            "first_name" : {}
        }
    }
}'
```
Match Phrase Query 匹配短语查询
匹配短语查询要求查询字符串中的trems要么都出现Document中、要么trems按照输入顺序依次出现在结果中。在默认情况下，查询输入的trems必须在搜索字符串紧挨着出现，否则将查询不到。不过我们可以指定slop参数，来控制输入的trems之间，通过最多几次转换能够搜索到
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match": {
            "query": "climb rock",
            "fields": [
                "about",
                "interests"
            ],
            "type": "phrase",
            "slop": 3
        }
    },
    "_source": [
        "title",
        "about",
        "interests"
    ]
}'
```

Match Phrase Prefix Query 匹配短语前缀查询
匹配短语前缀查询可以指定单词的一部分字符前缀即可查询到该单词，和match phrase query一样我们也可以指定slop参数；同时其还支持max_expansions参数限制被匹配到的terms数量来减少资源的使用
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "match_phrase_prefix": {
            "summary": {
                "query": "cli ro",
                "slop": 3,
                "max_expansions": 10
            }
        }
    },
    "_source": [
        "about",
        "interests",
        "first_name"
    ]
}'
```

Query String
query_string查询提供了一种手段可以使用一种简洁的方式运行multi_match queries, bool queries, boosting, fuzzy matching, wildcards, regexp以及range queries的组合查询。在下面的例子中，我们运行了一个模糊搜索(fuzzy search)，搜索关键字是search algorithm，并且作者包含grant ingersoll或者tom morton。并且搜索了所有的字段，其中summary字段的权重为2
``` bash
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "query_string" : {
            "query": "(saerch~1 algorithm~1) AND (grant ingersoll) OR (tom morton)",
            "fields": ["_all", "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}'
```

Simple Query String 简单查询字符串
simple_query_string是query_string的另一种版本，其更适合为用户提供一个搜索框中，因为其使用+/|/- 分别替换AND/OR/NOT，如果用输入了错误的查询，其直接忽略这种情况而不是抛出异常。使用如下：
``` bash
curl -POST 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "simple_query_string" : {
        "query": "(saerch~1 algorithm~1) + (grant ingersoll) | (tom morton)",
        "fields": ["_all", "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}'
```

Sorted 结果排序
对输出结果按照多层进行排序
``` bash
curl -XPOST 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "term" : {
            "interests": "music"
        }
    },
    "_source" : ["interests","first_name","about"],
    "sort": [
        { "publish_date": {"order":"desc"}},
        { "id": { "order": "desc" }}
    ]
}'

```

Range Query 范围查询
``` bash
curl -XPOST 'localhost:9200/person/worker/_search?pretty' -d '
{
    "query": {
        "range" : {
            "birthday": {
                "gte": "2017-02-01",
                "lte": "2017-05-01"
            }
        }
    },
    "_source" : ["first_name","last_name","birthday"]
}'
```

Filtered Query 过滤查询
``` bash
curl -XPOST :9200/megacorp/employee/_search?pretty' -d '
{
    "query": {
        "filtered": {
            "query" : {
                "multi_match": {
                    "query": "music",
                    "fields": ["about","interests"]
                }
            },
            "filter": {
                "range" : {
                    "birthday": {
                        "gte": 2017-02-01
                    }
                }
            }
        }
    },
    "_source" : ["first_name","last_name","about", "interests"]
}'
```
过滤查询(Filtered queries)并不强制过滤条件中指定查询,如果没有指定查询条件，则会运行match_all查询，其将会返回index中所有文档，然后对其进行过滤，在实际运用中，过滤器应该先被执行，这样可以减少需要查询的范围，而且，第一次使用fliter之后其将会被缓存，这样会对性能代理提升。Filtered queries在Elasticsearch 5.0中移除了，我们可以使用bool查询来替换他，下面是使用bool查询来实现上面一样的查询效果，返回结果一样：
``` bash
curl -XPOST 'localhost:9200/megacorp/employee/_search?pretty' -d '
{
    "query": {
        "bool": {
            "must" : {
                "multi_match": {
                    "query": "music",
                    "fields": ["about","interests"]
                }
            },
            "filter": {
                "range" : {
                    "birthday": {
                        "gte": 2017-02-01
                    }
                }
            }
        }
    },
    "_source" : ["first_name","last_name","about", "interests"]
}'
```
Multiple Filters 多过滤器查询
``` bash
curl -XPOST 'localhost:9200/iteblog_book_index/book/_search?pretty' -d '
{
    "query": {
        "filtered": {
            "query" : {
                "multi_match": {
                "query": "elasticsearch",
                "fields": ["title","summary"]
                }
            },
            "filter": {
                "bool": {
                    "must": {
                        "range" : { "num_reviews": { "gte": 20 } }
                    },
                    "must_not": {
                        "range" : { "publish_date": { "lte": "2014-12-31" } }
                    },
                    "should": {
                        "term": { "publisher": "oreilly" }
                    }
                }
            }
        }
    },
    "_source" : ["title","summary","publisher", "num_reviews", "publish_date"]
}'

```

Function Score: Field Value Factor 自定义得分计算
在某些场景下，你可能想对某个特定字段设置一个因子(factor)，并通过这个因子计算某个文档的相关度(relevance score)。这是典型地基于文档(document)的重要性来抬高其相关性的方式。在下面例子中，我们想找到更受欢迎的图书(是通过图书的评论实现的)，并将其权重抬高，这里可以通过使用field_value_factor来实现
``` bash
curl -XPOST 'localhost:9200/iteblog_book_index/book/_search?pretty' -d '
{
    "query": {
        "function_score": {
            "query": {
                "multi_match" : {
                    "query" : "search engine",
                    "fields": ["title", "summary"]
                }
            },
            "field_value_factor": {
                "field" : "num_reviews",
                "modifier": "log1p",
                "factor" : 2
            }
        }
    },
    "_source": ["title", "summary", "publish_date", "num_reviews"]
}'
```