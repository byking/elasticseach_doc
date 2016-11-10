# 安装

elasticsearch的java版本至少为java8
下面两种方式中选一个保证: 

```bash
java -version
echo $JAVA_HOME
```

下载elasticsearch安装包`www.elastic.co/downloads`
示例：下载elasticsearch 5.0.0 

```bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.0.tar.gz
tar -xzvf elasticsearch-5.0.0.tar.gz
cd elasticsearch-5.0.0/bin
./elasticsearch
```
