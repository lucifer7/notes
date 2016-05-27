[TOC]

# 参考文档
[零拷贝技术](http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/)
[rocketmq特性](www.jianshu.com/p/453c6e7ff81c)
[rocketmq-demo代码-考拉哥](http://lifestack.cn/archives/tag/rocketmq)
[RocketMq 斩秋的专栏](http://blog.csdn.net/u010311445/article/category/2548637)

# 1 Rocket MQ 介绍

## 1.1 简介

RocketMQ 是一款分布式、队列模型的消息中间件，具有以下特点：
能够保证严格的消息顺序
提供丰富的消息拉取模式
高效的订阅者水平扩展能力
实时的消息订阅机制
亿级消息堆积能力
**选用理由**：
 强调集群无单点，可扩展，任意一点高可用，水平可扩展。
 海量消息堆积能力，消息堆积后，写入低延迟。
 支持上万个队列
 消息失败重试机制
 消息可查询
 开源社区活跃
 成熟度（经过双十一考验）


## 1.2 安装

**1. 配置hostname**
`vim /etc/hosts`
```
192.168.0.121 rocketmq-nameserver1
192.168.0.121 rocketmq-master1
192.168.0.122 rocketmq-nameserver2
192.168.0.122 rocketmq-master2
```

**2.解压文件**

```
# 上传alibaba-rocketmq-3.2.6.tar.gz文件至/usr/local
# tar -zxvf alibaba-rocketmq-3.2.6.tar.gz -C /usr/local
# mv alibaba-rocketmq alibaba-rocketmq-3.2.6
# ln -s alibaba-rocketmq-3.2.6 rocketmq
ll /usr/local
```


**3. 创建存储路径**

```
# mkdir /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store/commitlog
# mkdir /usr/local/rocketmq/store/consumequeue
# mkdir /usr/local/rocketmq/store/index
```

**4.修改配置文件**

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a|broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

**4.修改日志配置文件**

```
# mkdir -p /usr/local/rocketmq/logs
# cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' *.xml
```


**5.测试环境修改JVM参数**

`vim /usr/local/rocketmq/bin/runbroker.sh`

```
#============================================================
==================
# 开发环境JVM Configuration
#============================================================
==================
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -
XX:PermSize=128m -XX:MaxPermSize=320m"
```

`vim /usr/local/rocketmq/bin/runserver.sh`

```
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -
XX:PermSize=128m -XX:MaxPermSize=320m"
```


**6. 启动机器**
a. 启动2台broker
```
# cd /usr/local/rocketmq/bin
# nohup sh mqnamesrv &
```
b. 启动master1

```
# cd /usr/local/rocketmq/bin
# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 &
# netstat -ntlp
# jps
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log
```

c.启动master2

```
# cd /usr/local/rocketmq/bin
# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-b.properties >/dev/null 2>&1 &
# netstat -ntlp
# jps
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log
```


**7.数据清理：**

```
# cd /usr/local/rocketmq/bin
# sh mqshutdown broker
# sh mqshutdown namesrv
# --等待停止
# rm -rf /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store/commitlog
# mkdir /usr/local/rocketmq/store/consumequeue
# mkdir /usr/local/rocketmq/store/index
# --按照上面步骤重启NameServer与BrokerServer
```


## 1.3 消息中间件需要解决哪些问题

具体请参考官方使用手册



# 2.  API

## 2.1 HelloWorld


**生产者**

```java
  //创建一个容器（例如tomcat中唯一的producer，这个名字必须唯一）
        DefaultMQProducer producer = new DefaultMQProducer("quickstart_producer");
        //设定nameSrv
        producer.setNamesrvAddr("192.168.0.121:9876;192.168.0.122:9876");
        //producer
        producer.start();
        //发送100条消息
        for (int i = 0; i < 100; i++) {
            Message message =
                    new Message("TopicQuickStart", "TagA", ("Hello ," + i).getBytes());
            try {
                SendResult result = producer.send(message);
                System.out.println(result);
            } catch (MQClientException e) {
                e.printStackTrace();
            } catch (RemotingException e) {
                e.printStackTrace();
            } catch (MQBrokerException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        producer.shutdown();
```

**消费者**
```java
 //创建消费者Push对象
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("quickstart_consumer");
        //设置NameSvr
        consumer.setNamesrvAddr("192.168.0.121:9876;192.168.0.122:9876");
        //订阅主题，并指明tags
        consumer.subscribe("TopicQuickStart", "*");
        //设置监听器对象
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                try {
                    for (MessageExt msg : msgs) {
                        String topic = msg.getTopic();
                        String msgBody = new String(msg.getBody(), "utf-8");
                        String tags = msg.getTags();
                        System.out.printf("收到消息：topic:%s,tags:%s,msg:%s",topic,tags,msgBody);
                        System.out.println();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //开始
        consumer.start();
```

> 注意问题：
> 1. 消费端一般应该先启动
> 2. 服务端的new DefaultMQProducer("quickstart_producer")这个名字应该保证一个jvm中唯一？


## 2.2 批处理 batch

在push的方式下，如果消息堆积，可以设置
` consumer.setConsumeMessageBatchMaxSize(4);`来批处理

在pull的方式下，也可以通过``来设置批处理

## 2.3 重试策略

**三种情景：**

服务端-》broker-》consumer

1. 服务端发送broker失败，可以通过设置` producer.setRetryTimesWhenSendFailed(3);
        producer.setRetryAnotherBrokerWhenNotStoreOK(true|false);`来设置
 2. broker发送给consumer失败，服务端会不断的重试
 3. consumer消费失败，可以通过返回`ConsumeConcurrentlyStatus.RECONSUME_LATER;`来重试，此时，broker会在根据配置1s,5s,1min。。。依次时间之后进行重试。

> 可以在消费端故意抛出异常测试，直接扔出异常就可以


## 2.4 Customer的负载均衡

### 2.4.1 集群模式与广播模式

默认就是集群模式

```java
	//设置为广播模式的方法
	consumer.setMessageModel(MessageModel.BROADCASTING);
```

### 2.4.2 负载均衡情景可能会有重复消费问题

只要有网络传输，必然存在这种可能性，需要在客户端手动处理，例如去重表等等手段

### 2.4.3 消息回馈

在没有任何消息回馈的时候，可能会转发到其他的customer上，实验如下：

**1.先启动2个客户端 01,02，其中01客户端会暂停**

```
 System.out.printf("1111111收到消息：topic:%s,tags:%s,msg:%s\n\r",
                                msg.getTopic(),
                                msg.getTags(),
                                new String(msg.getBody(), "utf-8"));
                        TimeUnit.SECONDS.sleep(60); //休眠一分钟之中关闭连接
```

**2.生产者生成数据**
**3.手动关闭01**
**4.发现02有了数据**

>这说明broker在没有接收到01的回馈的时候将消息发送给了02



#3. 架构

### 3.1 双Master模式


双master模式下，如果其中一个broker，例如broker-a挂了，这时候已经发送到a的数据会无法接收到，但是**不会丢失**，只有a重新启动的时候在会被消费

一般可以手动监听哪个master挂了，然后立马通知运维去重新启动，一般生产环境中这种多master的性能是最好的，但是也存在丢失实时性的可能。


**测试如下：**

**1. 一个producer发送数据**

**2. 手动关闭broker-a**

**3.开启消费者**

**4.发现只消费了部分数据**

**5.手动启动broker-a**

**6.消费者此时消费了全部的数据**


### 3.2 多master多slave的方式


master - slave之间丢失了部分的性能，但是能满足更好的可用性，实时性，有2种方式，这2种方式都是针对于主从之间的数据一致性


**1.异步复制**

	一个数据写到master，异步的发送给slave，丢了部分的安全性，但是性能更好

**2.同步双写**

	一个数据写到master，同步的发送给slave，丢失了部分的性能，但是数据更加安全

**如何安装：**

	1. 相同的brokerName
	2. brokerId>0表示是从
	3. 还有个角色记得改为SLAVE

例如 2m-2s-async/broker-a-s.properties如下所示:

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876;rocketmq-nameserver3:9876;rocketmq-nameserver4:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```


以下是说明：
1. 当master由于异常而挂了的话，slave不会自动升级为master
2. 此时的slave只能接受读请求而不能接受写请求
3. 如果是阿里的闭源版本可能会自动将slave升级为master
4. rocketmq一个topic，默认有4个queue

### 3.3 rocketmq架构漫谈

redis这样的产品可以实现很多的功能，但主要是用来当作缓存，redis也可以当作消息分发，当作队列，当作session等等各种各样形形色色的功能，但主要还是用来当作缓存

rocketmq最大的优点是什么，个人感觉是消息的失败重试机制，这点rocketmq做的非常完美，消息中间件，当producer将消息发送给broker，就可以什么都不用管了。

rocketmq的底层源码也很值得学习，你可以通过源码学习

# 4. 特性


## 4.1 顺序消费

groupName非常重要，不管是customer端还是producer端

1个topic可以设置多个队列

顺序消费的原理，可以指定队列

```java
	//指定发送到这个消息队列中
 MessageQueueSelector selector = new MessageQueueSelector() {
            public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                Integer id = (Integer) o;
                return list.get(id);
            }
        };

        for (int j = 0; j < 5; j++) {
            for (int i = 0; i < 10; i++) {
                Message message =
                        new Message("TopicQuickStart", "TagA", ("Hello ," + j + "------------" + i).getBytes());
                SendResult result = producer.send(message, selector, j % 4);
//            SendResult result = producer.send(message);
                System.out.println(result);
            }
        }
```

> 1. rocketmq的逻辑结构是topic，但是一个topic对应多个queue
> 2. rocketmq理论上支持一个topis下有上万个queue
> 3. 通过测试发现，如果指定了queue，相同队列的数据会发送相同的customer中


## 4.2 如何去重复消费

	可以利用去重表，最好不要第三方的工具，例如redis等等，因为去重表主要是为了去重，有强一致性的要求，因此，应该使用rds来保证去重。



## 4.3 事务消费

具体可以参考上面的博客

如何在3.2.6 之后阉割版的事务消费功能
可以考虑和消费端进行交互解决，返回Unkonwn的情况

## 4.4 Pull消费

	pull的方式在生产环境中并不多见，当消息堆积到中间件的时候，pull的方式可以保证cunstomer慢慢处理，慢慢的处理这些消息。我主动来拿的话就比较好控制，比如以前项目碰见过第三方服务方老是抱怨我们的并发太强，他们承受不住，这种场景下可以考虑使用中间件。

提示关键字：
SecheduleService
5s自动和服务器同步
可以批处理
处理失败是不会重试的，需要自己处理
记录日志

```java
package com.carl.pull;

import com.alibaba.rocketmq.client.consumer.*;
import com.alibaba.rocketmq.common.message.MessageExt;
import com.alibaba.rocketmq.common.message.MessageQueue;
import com.alibaba.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

/**
 * Created by carl on 16-5-13.
 */
public class Consumer {

    public static void main(String[] args) throws Exception {

        final MQPullConsumerScheduleService scheduleService = new MQPullConsumerScheduleService("GroupName1");
        //
        scheduleService.getDefaultMQPullConsumer().setNamesrvAddr("127.0.0.1:9876");
        //设置拉取的方式也可以是集群方式，即是负载均衡
        scheduleService.setMessageModel(MessageModel.CLUSTERING);

        //注册监听器
        scheduleService.registerPullTaskCallback("TopicTest", new PullTaskCallback() {

            @Override
            public void doPullTask(MessageQueue mq, PullTaskContext context) {
                MQPullConsumer consumer = context.getPullConsumer();
                try {
                    // 这种方式服务器有机制会和客户端自动同步上次拉取的位置
                    long offset = consumer.fetchConsumeOffset(mq, false);
                    if (offset < 0)
                        offset = 0;

                    //pull(队列，过滤方式，从什么地方开始拉去，批处理数量)
                    PullResult pullResult = consumer.pull(mq, "*", offset, 32);
//                    System.out.println(offset + "\t" + mq + "\t" + pullResult);
                    switch (pullResult.getPullStatus()) {
                        case FOUND:
                            //如果拉取到了数据
                            List<MessageExt> messageExtList = pullResult.getMsgFoundList();
                            //消费端都需要手动去重
                            System.out.println("拉取到了数据---------------");
                            for (MessageExt messageExt : messageExtList) {
                                System.out.println(new String(messageExt.getBody(), "utf-8"));
                                //消息不会重发，需要主动做日志处理
                            }
                            System.out.println("-----------------");
                            break;
                        case NO_MATCHED_MSG:
                            break;
                        case NO_NEW_MSG:
                        case OFFSET_ILLEGAL:
                            break;
                        default:
                            break;
                    }

                    // 存储Offset，客户端每隔5s会定时刷新到Broker，很关键
                    consumer.updateConsumeOffset(mq, pullResult.getNextBeginOffset());

                    // 设置再过10s去拿数据，一般建议大于5s，因为每5s同步一次数据
                    context.setPullNextDelayTimeMillis(10 * 1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });

        scheduleService.start();
        System.out.println("customer start...");
    }

}

```


> 以上代码可以保证不管生产者的并发有多强，消息会堆积在分布式broker，利用rocketmq的强大堆积能力，可以保证稳定消费.

