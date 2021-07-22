---
title: Elasticsearch-AdvanceSearch
tags:
  - Elasticsearch
toc: true
date: 2021-07-21 16:09:32
categories:
---

## 批量导入数据
将 account.json 的数据加载到 elastic 中
数据类似如下(2条)
```json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
```
```zsh
## cd account.json 文件所在的目录
curl -XPOST "localhost:9200/bank/_bulk?pretty&refresh" -H "Content-Type: application/json" --data-binary "@accounts.json"

curl -XGET "localhost:9200/_cat/indices?v"
## output 可以看到 1000 个 doc 已经存储到 bank 索引中了
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank     qerxmeePRsmIeXoZ58eNsQ   1   1       1000            0    396.8kb        396.8kb
yellow open   customer LbQLl0UQTSOPnn54wQwA2g   1   1          1            1      4.3kb          4.3kb
```

## 搜索 _search
注意，一旦获得了搜索结果，Elasticsearch就会结束这次搜索，不会再维护任何服务端资源，也没有结果游标，这与其他很多平台，如SQL，不一样。
### 通过 uri 传递参数
```zsh
GET /bank/_search?q=*&sort=account_number:asc&pretty
curl -XGET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
## 参数说明
# q=* 搜索索引中的所有文档
# sort=account_number:asc 搜索结果以字段account_number升序排列
# pretty 返回结果以漂亮的JSON格式打印
```
### 通过 request body 传递参数
```zsh
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}

curl -XGET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {"match_all": {}},
  "sort": [
    {"account_number": "asc"}
  ]
}
'
```

### 响应数据
```json
{
  "took" : 22,                      // 搜索时间(毫秒)
  "timed_out" : false,              // 搜索是否超时
  "_shards" : {                     // 搜索了多少分片，搜索分片的成功/失败计数
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {                        // 搜索结果
    "total" : {                     // 搜索命中总数信息
      "value" : 1000,               // 命中总数
      "relation" : "eq"             // 取值eq(等于)/gte(大于等于)，表示hits.total.value与实际的搜索命中数量的关系
                                    // 如果搜索结果很多，超过一定数量后，通常就不再统计，只是笼统地表示为：搜索结果超过XXXX个。hits.total的准确性由请求参数track_total_hits控制，当track_total_hits为true时，搜索时将精确地跟踪总命中数(“relationship”:“eq”)。track_total_hits默认值为10,000，意味着总命中数可以精确地跟踪到10000个文档，如果超过10000，会表示为超过10000个结果
    },
    "max_score" : null,             // hits._score 与 max_score – 分数是衡量文档与搜索条件匹配程度的一个指标。分数越高，文档越相关，分数越低，文档越不相关。并不总是需要生成分数，需不需要Elasticsearch会自动判断，以避免计算无用的分数。
    "hits" : [                      // 实际的搜索结果数组(默认为前10个文档)
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [                // 结果排序键(如果按分数排序，则忽略)
          0
        ]
      },
      ...
    ]
  }
}
```

## Query DSL(Query domain-specific language)
Elasticsearch提供了一种json风格的查询语言
### sort, from, size 排序,分页

```zsh
curl -XGET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {"match_all": {}},
  "sort": [
    {"account_number": "asc"}
  ],
  "from":5,
  "size":5
}
'
```

query 表示这次查询的定义
match_all 表示查询类型-匹配所有文档
sort 指定排序
from 参数(基于0)指定从哪个文档序号开始(默认0)，size参数指定返回多少个文档(默认10),这两个参数对于搜索结果分页非常有用

### _source 指定返回的字段
默认情况下， 搜索结果中包含了完整的json文档( _source 字段),如果不希望返回源文档全部内,可以设置要返回的字段
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query":{"match_all": {}},
  "_source":["account_number", "balance"]
}
'
```

### match_all 匹配所有文档
```zsh
curl -XGET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {"match_all": {}}
}
'
```
### match 匹配查询
#### 返回 account_number 为 20 的所有账户
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {"account_number": 20}
  }
}
'
```
#### 返回 address 包含 mill 的所有账户
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {"address": "mill"}
  }
}
'
```
#### 返回 address 包含 mill 或 lane 的所有账户
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {"address": "mill lane"}
  }
}
'
```
### match_phrase 匹配整个短语
#### 返回 address 包含 "mill lane" 的所有账户
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase": {"address": "mill lane"}
  }
}
'
```
### bool 布尔查询
布尔查询使用布尔逻辑，将小查询组合成大查询
#### bool must 包含两个 match,逻辑与
返回地址中包含 mill 且也包含 lane 的账户
must 下所有匹配条件为真，文档才视为匹配
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {"match": {"address": "mill"}},
        {"match": {"address": "lane"}}
      ]
    }
  }
}
'
```
#### bool should 包含两个 match,逻辑或
返回地址中包含 mill 或也包含 lane 的账户
must 下任意一个匹配条件为真，文档就视为匹配
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        {"match": {"address": "mill"}},
        {"match": {"address": "lane"}}
      ]
    }
  }
}
'
```
#### bool must_not包含两个 match,逻辑与非
返回地址中既不包含“mill”也不包含“lane”的帐户
bool must_not子句包含的匹配条件全部为假，文档将被视为匹配
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        {"match": {"address": "mill"}},
        {"match": {"address": "lane"}}
      ]
    }
  }
}
'
```
#### bool 查询中同时组合 must、should和must_not (解决should 不生效问题)
当使用should查询时，如果包含了must或者filter查询，那么should的查询语句就不是或者的意思了,而是有或者没有都行的含义。
```zsh
## 解决方法 1: 使用 minimum_should_match
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ],
      "should": [
        {"match": {"address": "Jackson"}},
        {"match": {"address": "Suydam"}}
      ],
      "minimum_should_match": 1
    }
  }
}
'
## 解决方法 2: 应先满足must，然后在must的基础上进行should查询,即转换成复杂的组合查询，就是满足一定的条件下，再满足条件1或者满足条件2或者满足条件3
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {  
          // 先满足前置条件
          "bool": {
            "must": [
              { "match": { "age": "40" } }
            ],
            "must_not": [
              { "match": { "state": "ID" } }
            ]
          }
        }, 
        {
          // 再满足后置条件
          "bool": {
            "should": [
              {"match": {"address": "Jackson"}},
              {"match": {"address": "Suydam"}}
            ]
          }
        }
      ]
    }
  }
}
'
```
## filter 过滤
_score(分数)字段是衡量文档与搜索条件匹配程度的一个指标。分数越高，文档越相关，分数越低，文档越不相关。并不总是需要生成分数，需不需要Elasticsearch会自动判断，以避免计算无用的分数。
布尔查询还支持filter子句，用于设置过滤条件。过滤条件不影响文档的相关性分数。

### 返回 balance 在 20000-30000 之间的账户
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {}},
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```
## aggs 聚合
聚合提供了对数据进行分组、统计的能力，类似于SQL中GROUP by和SQL聚合函数。在Elasticsearch中，可以同时返回搜索结果及其聚合计算结果，这是非常强大和高效的。

### terms 账户按所在州分组，统计每组账户数量，然后返回前10条目
类似sql中的 `SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;`

注: size=0 表示不显示搜索结果，我们只想看到聚合结果
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {       // group_by_state 不是关键字，只是结果展示的名称
      "terms": {              // 这个才是关键字
        "field": "state.keyword"
      }
    }
  }
}
'
```
### terms, avg 所有账户按州计算平均账户余额,返回前10条目,按账户数量降序排列
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {                   // 先按照州聚合
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {               // 再计算每个州的平均账户余额
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```
### range, term, avg 先按照年龄段分组，然后按性别分组，统计每个年龄等级，每种性别的平均账户余额
```zsh
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
'
```


## 参考资料
[Elasticsearch 搜索数据](https://www.qikegu.com/docs/3077)
[ES多条件查询must和should不能同时生效问题](https://blog.csdn.net/weixin_44993313/article/details/105905735)
[es bool多条件查询should和must同时使用注意](https://blog.csdn.net/JineD/article/details/111997205)