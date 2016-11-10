# 配置

很多配置都默认配置好了，而且可以通过`Cluster Update Settings`接口在elasticsearch启动后配置。

配置文件需要有节点的配置（例如node.name和paths），另外还要有集群配置，用来指定节点加入哪个集群。例如：
cluster.name和network.host。

## 配置文件路径
elasticsearch有两个配置文件：

- elasticsearch.yml：配置elasticsearch
- log4j2.properties：配置elasticsearch日志。

这两个文件在配置目录中，配置目录默认是`$ES_HOME/config/`.Debian和RPM配置在在/etc/elasticsearch/中。

配置文件路径可以通过下面修改：

```bash
./bin/elasticsearch -Epath.conf=/path/to/my/config/
```

## 配置文件格式

配置文件使用 [YAML](http://www.yaml.org/) 格式，下面举例修改数据或者日志路径:

```bash
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

可以这样配置：

```bash
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

## 使用环境变量替换

配置文件中使用的`${...}`标记会被替换成当前环境变量：

```bash
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

## 启动时配置

配置不想记录在配置文件的时候可以在启动的时候指定配置的参数，例如：

```bash
node:
      name: ${prompt.text}
```

启动的时候设置格式为：

```bash
`Enter` value of [node.name]：
```
`注: `启动时配置的elasticsearch的配置如果和已经启动的elasticsearch配置相同，这时候elasticsearch不会启动。

##设置默认配置：

默认配置都有`default.`前缀。可以通过下面的方式修改：

```bash
./bin/elasticsearch -Edefault.node.name=My_Node
```
这个节点名字默认变成了`My_Node`，除非通过命令行重写`es.node.name`或者配置文件中修改`node.name`

## 日志配置

日志配置在log4j2.properties文件中， [log4j2](http://logging.apache.org/log4j/2.x/)
elasticsearch通过`${sys:es.logs}`来指向日志文件，这个值会在elasticsearch运行的时候设置,更加运行目录＋集群名称来确定。
例如：如果日志目录为`/var/log/elasticsearch`，集群名称为`production`，那么这个变量值为`/var/log/elasticsearch/production`。

```bash
appender.rolling.type = RollingFile 
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs}.log 
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %.10000m%n
appender.rolling.filePattern = ${sys:es.logs}-%d{yyyy-MM-dd}.log 
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy 
appender.rolling.policies.time.interval = 1 
appender.rolling.policies.time.modulate = true 
```
- 添加日志切分的配置
- 日志filename为：/var/log/elasticsearch/production.log
- 切分日志格式为：/var/log/elasticsearch/production-yyyy-MM-dd.log
- policies.time.type：使用基于时间的策略
- policies.time.interval：天级别切分
- policies.time.modulate：按照天的时间（而不是按照启动时间）


