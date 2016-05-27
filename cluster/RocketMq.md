1. get latest release
[release download page](https://github.com/alibaba/RocketMQ/releases)

一、安装rocketmq
<pre>
# tar zxf alibaba-rocketmq-3.2.4-beta1.tar.gz -C /usr/local/
# cd /usr/local/
# ln -s /usr/local/alibaba-rocketmq /usr/local/rocketmq
# cd rocketmq/
</pre>

4、启动RocketMQ

# cd ../bin/
# nohup sh mqnamesrv >/var/log/ns.log &
# nohup sh mqbroker -c ../conf/2m-noslave/broker-a.properties > /var/log/mq.log 2>&1 &
5、查看启动日志

# tail -f /var/log/ns.log 
# tail -f /var/log/mq.log
6、查看启动端口

复制代码
# netstat -tunpl
# jps
# kill -9 22596
# kill -9 22564
# kill -9 9967
# netstat -tunpl
# netstat -tunpl |grep java
复制代码
7、关闭RocketMQ

# sh mqshutdown
1 Useage: mqshutdown broker | namesrv
# sh mqshutdown broker
# sh mqshutdown namesrv
8、再次启动

# nohup sh mqnamesrv >/var/log/ns.log &
# nohup sh mqbroker -c ../conf/2m-noslave/broker-a.properties > /var/log/mq.log 2>&1 &
9、验证状态

# jps

[See Reference Page here](http://www.cnblogs.com/babywaa/p/4874595.html)
