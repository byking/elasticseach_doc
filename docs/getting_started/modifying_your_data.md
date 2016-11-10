# 数据修改

## 索引/替换文档

首先新建一个文档:

```bash
curl -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -d'
{
      "name": "John Doe"
}'
```

然后替换：

```bash
curl -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -d'
{
      "name": "Jane Doe"
}'
```

上面操作将ID为1的文档从"John Doe"替换为"Jane Doe"。下面用不同的编号，文档内容相同，
不会替换不同ID的文档。

```bash
curl -XPUT 'localhost:9200/customer/external/2?pretty&pretty' -d'
{
      "name": "Jane Doe"
}'
```

上面操作新建一个ID为2的文档。

建索引的时候ID：文档编号是可选的，如果没有指定，elasticsearch会随机生成ID用来建索引。

下面这个例子说明如何在不指定文档ID的时候建索引：

```bash
curl -XPOST 'localhost:9200/customer/external?pretty&pretty' -d'
{
      "name": "Jane Doe"
}'
```

## 更新文档

替换文档示例:

```bash
curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d'
{
      "doc": { "name": "Jane Doe" }
}'
```

也可以多加字段:

```bash
curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d'
{
      "doc": { "name": "Jane Doe", "age": 20 }
}'
```

更新age字段，给age加5：

```bash
curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d'
{
      "script" : "ctx._source.age += 5"
}'
```
`ctx._source` 指向当前文档的soruce。

这种更新操作每次只能更新一个文档，以后会考虑支持更新多个，就像SQL `UPDATE -WHARE`语句。

## 删除文档

删除文档操作很简单:

```bash
curl -XDELETE 'localhost:9200/customer/external/2?pretty'
```

如果要删除所有的文档，可以直接删除索引。

## 批量操作

上面看到可以对单个文档进行新建、更新、删除操作，elasticsearch还提供了`_bulk API`来支持批量操作。

批量建索引，建立两个文档ID 1 和ID 2:

```bash
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }'
```

批量操作，更新ID 1的文档，然后再删除ID 2的文档：

```bash
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}'
```

bulk操作顺序执行，有一个失败会继续下一个，最后的返回结果会记录每个操作的操作状态。
例如上面操作的返回：

```bash
{
      "took" : 11,
      "errors" : false,
      "items" : [
              {
                        "update" : {
                                    "_index" : "customer",
                                    "_type" : "external",
                                    "_id" : "1",
                                    "_version" : 7,
                                    "result" : "noop",
                                    "_shards" : {
                                                  "total" : 2,
                                                  "successful" : 1,
                                                  "failed" : 0
                                                },
                                    "status" : 200
                                  }
                      },
              {
                        "delete" : {
                                    "found" : true,
                                    "_index" : "customer",
                                    "_type" : "external",
                                    "_id" : "2",
                                    "_version" : 2,
                                    "result" : "deleted",
                                    "_shards" : {
                                                  "total" : 2,
                                                  "successful" : 1,
                                                  "failed" : 0
                                                },
                                    "status" : 200
                                  }
                      }
            ]
}
```

