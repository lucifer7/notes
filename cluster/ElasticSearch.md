### ElasticSearch Install
1. [Guidance Book](http://es.xiaoleilu.com/010_Intro/00_README.html)

Download: [elasticsearch-2.3.3.tar.gz](https://www.elastic.co/thank-you?url=https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.3/elasticsearch-2.3.3.tar.gz)

> tar -zxvf elasticsearch-2.1.0.tar.gz

1. 安装Head插件（Optional）：
> ./bin/plugin install mobz/elasticsearch-head

1. 然后编辑ES的配置文件：
> vi config/elasticsearch.yml

1. 修改以下配置项：
```
cluster.name=es_cluster
node.name=node0
path.data=/tmp/elasticsearch/data
path.logs=/tmp/elasticsearch/logs
\#当前hostname或IP，我这里是centos2
network.host=centos2
network.port=9200
```


1. 其他的选项保持默认，然后启动ES：
> ./bin/elasticsearch

#### Error Handle
1. Mind!! If run with root, error occurs:
> [root@localhost bin]# ./elasticsearch

```
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:93)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:144)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:270)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
```


Caused by:
> 这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考虑， 
  建议创建一个单独的用户用来运行ElasticSearch

1. Toggle comment on path.data and path.logs
1. Create new usergroup and user and assign
```
  groupadd elsearch
  useradd elsearch -g elsearch -p 123456
  cd /usr/local
  chown -R elsearch:elsearch  elasticsearch/
```

  
1. Switch to elsearch user
> su elsearch
> ./elstaticsearch

后台启动运行：
```
[elsearch@localhost bin]$ ./elasticsearch -d
[elsearch@localhost bin]$ jps
36554 Jps
36540 Elasticsearch
```


1. Vistit site to check 
> http://10.200.157.50:9200/

and get a json type string

1. 查看集群状态
Console：
> http://10.200.157.50:9200/_plugin/head/
Kibana Sense:
> yum OR tar
[root@localhost bin]# vim /opt/kibana/config/kibana.yml 
[root@localhost bin]# service kibana restart
visit: http://10.200.157.50:5601/app/kibana

#### Application
ELK, Elasticsearch + Logstash + Kibana
搭建实时日志分析平台
[参考地址](http://www.importnew.com/20464.html)

### NOTES
一个实时分布式搜索和分析引擎

- 全文搜索
- 结构化搜索
- 分析
- Document oriented, (save document)

节点(node)是一个运行着的Elasticsearch实例。集群(cluster)是一组具有相同cluster.name的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。

The name is important because a node can only be part of a cluster if the node is set up to join the cluster by its name. 

Document is a type, which belongs to Index.
Compare with RDB:
```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```

- Inverted index 倒排索引

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
VERB HTTP方法：GET, POST, PUT, HEAD, DELETE
PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
PORT Elasticsearch HTTP服务所在的端口，默认为9200
PATH API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据
BODY 一个JSON格式的请求主体（如果请求需要的话）
```
