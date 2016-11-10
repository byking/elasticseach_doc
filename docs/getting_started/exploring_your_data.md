# 数据探索

## 示例数据结构

下面是一个JSON的文档，存储银行用户账户信息，每个文档都有下面的结构：

```bash
{
        "account_number": 0,
        "balance": 16623,
        "firstname": "Bradshaw",
        "lastname": "Mckenzie",
        "age": 29,
        "gender": "F",
        "address": "244 Columbus Place",
        "employer": "Euron",
        "email": "bradshawmckenzie@euron.com",
        "city": "Hobucken",
        "state": "CO"
}
```
可以使用 [www.json-generator.com](http://www.json-generator.com/) 来生成随机数据

## 加载示例数据

可以通过 [这里](https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json#) 下载示例数据。解压后放在当前目录并加载：

```bash
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
```

输出如下：

```bash
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb
```

## 检索接口`_search`

检索有两种方式，通过REST request URI或者通过REST request body。通过body传参的方式使用json格式表达能力更强。

使用URI的方式：

```bash
curl -XGET 'localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty'
```

其中`q=*`是对bank索引中所有文档检索，条件是按照`account_number`升序，pretty按照json返回

结果中包括几个关键字段：

- took：elasticsearch执行的时间，单位毫秒
- `time_out`：是否超时
- `_shards`：操作了多少个分片，其中多少成功、多少失败
- hits：检索结果
- hits.total：满足条件的文档总数
- hits.hits：查询结果数组（默认为前10个文档）
- sort：返回结果中用来排序的字段的值（使用score排序时没有该字段）
- `_score` 和 `max_score`：打分

通过body执行上面的检索：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} },
      "sort": [
              { "account_number": "asc" }
            ]
}'
```

## 查询语言介绍

elasticsearch支持Query DSL能够很好的描述查询条件：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} }
}'
```
另外可以在传除了"query"之外的字段：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} },
      "size": 1
}'
```

`size`设置返回1条。

设置返回第11个到第20个文档，返回的文档从零开始计数：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} },
      "from": 10,
      "size": 10
}'
```

下面设置返回结果根据`balance`降序排序，返回前10个：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} },
      "sort": { "balance": { "order": "desc" } }
}'
```

## 更多检索示例Searchs

尝试Query DSL的更多用法。

默认情况下查询都会返回文档的所有内容，可以在查询的设置`_source`字段来获取指定的字段：


```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_all": {} },
      "_source": ["account_number", "balance"]
}'
```

返回中`_source`字段只会包括`account_number`和`balance`字段 

根据文档中字段检索：使用`match query`：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match": { "account_number": 20 } }
}'
```
返回`account_number=20`的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match": { "address": "mill" } }
}'
```
返回address为mail的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_phrase": { "address": "mill lane" } }
}'
```
返回address为mill或者lane的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": { "match_phrase": { "address": "mill lane" } }
}'
```
`match_phrase`返回address为`mill lane`的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": {
              "bool": {
                        "must": [
                                    { "match": { "address": "mill" } },
                                    { "match": { "address": "lane" } }
                                  ]
                      }
            }
}'
```
`boolean query`返回address中即包括mill又包括lane的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": {
              "bool": {
                        "should": [
                                    { "match": { "address": "mill" } },
                                    { "match": { "address": "lane" } }
                                  ]
                      }
            }
}'
```
相反返回address中保护mill或者lane的文档。

```bash 
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": {
              "bool": {
                        "must_not": [
                                    { "match": { "address": "mill" } },
                                    { "match": { "address": "lane" } }
                                  ]
                      }
            }
}'
```
返回address即不是mill又不是lane的文档。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": {
              "bool": {
                        "must": [
                                    { "match": { "age": "40" } }
                                  ],
                        "must_not": [
                                    { "match": { "state": "ID" } }
                                  ]
                      }
            }
}'
```
返回age 40并且state不是ID的文档。


## 过滤Filters

`_score` 是检索结果和文档相关性打分，越高越匹配。
使用场景中并不是所有的查询都通过`_score`来过滤，这时候我们可以通过`filter`指令来过滤，例如：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "query": {
              "bool": {
                        "must": { "match_all": {} },
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
}'
```
只获取balance在20000到30000的文档。

## 聚合Aggregations

可以通过聚合将数据分组，类似与SQL的Group BY。
例如对所有的账户通过state字段分组，默认返回前10个，默认根据count降序排序(所有返回默认都是这样)。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "size": 0,
      "aggs": {
              "group_by_state": {
                        "terms": {
                                    "field": "state.keyword"
                                  }
                      }
            }
}'
```
其语义对应：
```bash
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```

通过设置`size=0`来避免返回hits的中文档的内容，因为我们只关心聚合的结果。

在上个例子的基础上，还可以计算每个state平均的balance值：

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "size": 0,
      "aggs": {
              "group_by_state": {
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
                      }
            }
}'
```
上面我们在聚合`group_by_state`中又嵌套了聚合。

下面对平均balance降序排列:

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
{
      "size": 0,
      "aggs": {
              "group_by_state": {
                        "terms": {
                                    "field": "state.keyword",
                                    "order": {
                                                  "average_balance": "desc"
                                                }
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
}'
```

下面这个例子对年龄分桶，然后根据gender分组，并计算每个年龄桶中没种gender的平均年龄。

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -d'
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
}'
```


