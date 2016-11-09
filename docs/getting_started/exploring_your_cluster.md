# 集群探索

## 集群健康检查

通过curl调用HTTP/REST接口可以查看集群健康状况：

```bash
curl -XGET 'localhost:9200/_cat/health?v&pretty'
```

作为输出：

```bash
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```
集群状态有green、yellow、red三种:

- green：集群所有状态很好（集群所有功能正常）
- yellow：所有数据都可以获取，但是集群有部分副本没有建立（集群所有功能正常）
- red：有些数据不能获取（集群部分功能正常）

从上面到输出可以看到现在只有一个节点，没有分片因为还没有存数据。另外我们是用默认的集群名称，有可能其他几点也会加入。
可以通过下面来看集群中节点:

```bash
curl -XGET 'localhost:9200/_cat/nodes?v&pretty'
```

作为输出：

```bash
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

## 查看所有索引

```bash
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

作为输出:
```bash
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

## 建索引

建一个名为"customer"的索引并产看所有索引：

```bash
curl -XPUT 'localhost:9200/customer?pretty&pretty'
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

作为输出：
```bash
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```

输出告诉我们，有一个名为customer的索引，有5个主分片，1个副本，里面没有文档。
同时可以看到集群健康状态变成yellow，表示有副本没有建立，因为我们只起了一个节点，因此没法建立副本。

## 索引并查询一个文档

现在尝试在customer索引中存一些东西，之前提到，每个文档都必须定义一个类型，我们定义一个"external"类型，文档ID为1:

```bash
curl -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -d'
{
      "name": "John Doe"
}'
```

输出如下：

```bash
{
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
              "total" : 2,
              "successful" : 1,
              "failed" : 0
            },
      "created" : true
}
```

查看我们建立的索引：

```bash
curl -XGET 'localhost:9200/customer/external/1?pretty&pretty'
```

输出：

```bash
{
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "found" : true,
      "_source" : { "name": "John Doe" }
}
```

`_source`是我们建立索引的时候传的原始信息。

## 删除索引：

```bash
curl -XDELETE 'localhost:9200/customer?pretty'
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

输出:

```bash
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

总结：目前为止学习的命令有

```bash
curl -XPUT 'localhost:9200/customer?pretty'
curl -XPUT 'localhost:9200/customer/external/1?pretty' -d'
{
      "name": "John Doe"
}'
curl -XGET 'localhost:9200/customer/external/1?pretty'
curl -XDELETE 'localhost:9200/customer?pretty'
```

与elasticsearch交互命令格式可以总结如下：

```bash
<REST Verb> /<Index>/<Type>/<ID>
```

