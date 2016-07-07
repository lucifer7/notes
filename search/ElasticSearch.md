### ElasticSearch Install
Download: [elasticsearch-2.3.3.tar.gz](https://www.elastic.co/thank-you?url=https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.3/elasticsearch-2.3.3.tar.gz)

> tar -zxvf elasticsearch-2.1.0.tar.gz

安装Head插件（Optional）：
> ./bin/plugin install mobz/elasticsearch-head

然后编辑ES的配置文件：
> vi config/elasticsearch.yml

修改以下配置项：

> cluster.name=es_cluster
node.name=node0
path.data=/tmp/elasticsearch/data
path.logs=/tmp/elasticsearch/logs
\#当前hostname或IP，我这里是centos2
network.host=centos2
network.port=9200

其他的选项保持默认，然后启动ES：