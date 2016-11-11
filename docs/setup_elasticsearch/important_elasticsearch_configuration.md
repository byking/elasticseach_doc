# elasticsearch的关键配置

尽管elasticsearch只需要很少的配置，但是在生产环境使用还需要修改下面几个重要的配置。

- path.data 和 path.log
- cluster.name
- node.name
- `bootstrap.memory_lock`
- network.host
- discovery.zen.ping.unicast.hosts
- `discovery.zen.minimum_master_nodes`

## path.data 和 path.log

如果通过.zip或者.tar.gz解压，data和log目录就在解压后的子目录下面，但是生产环境中还建议替换，防止
被删除或者替换。

```bash
path:
    logs: /var/log/elasticsearch
    data: /var/data/elasticsearch
```
 
path.data可以配置多个路径，但是一个分片的数据都在同一个目录下。

```bash
path:
    data:
      - /mnt/elasticsearch_1
      - /mnt/elasticsearch_2
      - /mnt/elasticsearch_3
```

## cluster.name

节点根据配置的cluseter.name来加入相同的集群。默认是elasticsearch，可以修改配置中的：

```bash
cluseter.name: logging-prod
```
## node.name

默认情况下node id是随机生成的，之后node就不会再变了。
node.name最好起一个有意义的名字：

```bash
node.name: prod-data-2
```

也可以直接用服务器名称来命名：

```bash
node.name: ${HOSTNAME}
```

## bootstrap.memory_lock

`bootstrap.memory_lock`设置为true可以防止内存被交换到磁盘。

## network.host

默认network.host值为127.0.0.1和`[::1]`，事实上，再同一个机器的路径上可以起多个节点，这样方便测试其集群功能。但是生产环境中不建议这样。
为了和能和其他server通信来组建一个集群，节点需要配置为例如下面这样的地址:

```bash
network.host: 192.168.1.10
```

## discovery.zen.ping.unicast.hosts

默认情况下，elasticsearch会通过绑定的lookback地址描9300和9305端口，来发现其他节点来通讯。通过这种方式可以自己组建集群。

当要和其他server组成集群的时候，需要提供一份在集群中的节点列表，如下：

```bash
discovery.zen.ping.unicast.hosts:
    - 192.168.1.10:9300
    - 192.168.1.11  
    - seeds.mydomain.com 
```
其中没有写端口的默认是9300端口，也可以通过域名配置（会尝试域名下面所有的机器）

## discovery.zen.minimum_master_nodes

为了防止数据丢失，`discovery.zen.minimum_master_nodes`配置很关键，设置最少有多少机器才能进行选主操作。
为了防止脑裂，这个值需要配置如下：

```bash
(master_eligible_nodes / 2) + 1
```
该值默认是1。

