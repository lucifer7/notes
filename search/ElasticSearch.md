### ElasticSearch Install
1. [Guidance Book](http://es.xiaoleilu.com/010_Intro/00_README.html)
Download: [elasticsearch-2.3.3.tar.gz](https://www.elastic.co/thank-you?url=https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.3/elasticsearch-2.3.3.tar.gz)

> tar -zxvf elasticsearch-2.1.0.tar.gz

1. 安装Head插件（Optional）：
> ./bin/plugin install mobz/elasticsearch-head

1. 然后编辑ES的配置文件：
> vi config/elasticsearch.yml

1. 修改以下配置项：
> cluster.name=es_cluster
node.name=node0
path.data=/tmp/elasticsearch/data
path.logs=/tmp/elasticsearch/logs
\#当前hostname或IP，我这里是centos2
network.host=centos2
network.port=9200

1. 其他的选项保持默认，然后启动ES：
> ./bin/elasticsearch

#### Error Handle
1. Mind!! If run with root, error occurs:
> [root@localhost bin]# ./elasticsearch
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:93)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:144)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:270)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)

Caused by:
> 这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考虑， 
  建议创建一个单独的用户用来运行ElasticSearch

1. Toggle comment on path.data and path.logs
1. Create new usergroup and user and assign
> groupadd elsearch
  useradd elsearch -g elsearch -p 123456
  cd /usr/local
  chown -R elsearch:elsearch  elasticsearch/  
  
1. Switch to elsearch user
> su elsearch
./elstaticsearch

后台启动运行：
> [elsearch@localhost bin]$ ./elasticsearch -d
[elsearch@localhost bin]$ jps
36554 Jps
36540 Elasticsearch

