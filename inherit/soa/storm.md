# Storm-基础篇
[TOC]
## 第一部分 storm简介


### 1.1 什么是Storm
	Storm是Twitter开源的一个分布式的实时计算系统，用于数据的实时分析，持续计算，分布式RPC等等。

一、实时计算典型用来解决的问题：

- **推荐系统**，例如淘宝等电商的推荐商品，hadoop适合做离线的数据分析
- **车流量的实时计算**，例如可以实时计算北京市每一个路段的拥挤度等相关路况信息
- **股票系统**，实时计算机制



二、功能：
**1.低延迟**

		实时系统的延迟肯定要低啊

**2.高性能：**

		几台普通的服务器就能实现相当可观的性能。

**3.支持容错**

		分布式的通用问题，节点挂了不影响对外的服务，Storm可以轻松做到在节点挂了的时候实现任务转移，并且在节点重启的时候（重新投入生产环境的时候）自动平衡任务。

**4.可扩展**

		分布式的通用问题

**5.可靠性**

		9之前使用Zeromq做异步队列，9之后使用Netty。Strorm保证每个消息至少能得到至少一次的完整处理。任务失败的时候，它会负责从消息源中重试消息。

**6.支持本地模式**


### 1.2 体系结构

	开源的分布式系统，可以简单可靠的处理大量的数据流。Storm拥有很多使用场景：例如实时分析，在线机器学习，持续计算，分布式RPC，ETL等等。Storm支持水平扩展，具有高容错性，保证每个消息都会得到处理，而且处理速度很快（在一个小集群中，每个节点每秒可以处理数以百万的消息）。Storm的部署和运维都很便捷，而且支持多语言。

	Nimbus：主节点通常运行一个后台程序-Nimbus，用于相应分布在集群中的节点，分配任务和检测故障。这个很类似于Hadoop中的Job Tracker

	Zookeeper：协调公有数据的存放（如心跳信息，集群状态，配置信息等等），Nimbus将分配给Supervisitor的任务下载zookeeper中。应用程序实现时的逻辑会被封装到Strom中的“topology”。topology是一组由Spouts(数据源)和Bolts(数据操作)通过Stream Groupings进行连接的图。

	Supervisor：工作节点同样会运行一个后台程序-Supervisor，用于收听工作指派基于要求运行工作进程。每个工作节点都是topology中一个子集的实现。而Nimbus和Supervisor之间的协调则通过Zookeeper系统集群。

	worker：负责具体逻辑的进程。

	Topology： Storm中运行的一个实时的应用程序，因为各个组件间的消息流动形成逻辑上的一个拓扑结构。一个topology是由spouts（水流的源头）和bolts（闪电）组成的图，通过stream groupings将图中的spouts和bolts连接起来。类似于工作流？




### 1.3 搭建storm环境

- 关闭防火墙，修改/etc/hosts配置（3台机器的IP可以相互通信）
- jdk7（至少1.6以上版本)
- Zk
- python （2.6.6版本以上）
- 下载Storm，解压，配置环境变量STORM_HOME
- 修改strom.yaml 配置文件，创建data文件夹
```
 storm.zookeeper.servers:
	- "192.168.0.115"
	- "192.168.0.117"
	- "192.168.0.118"
 nimbus.host: "192.168.0.115"
 storm.local.dir: "usr/local/storm/data"
 ui.port: 18080
 supervisor.slots.ports:
	 - 6700
	 - 6701
	 - 6702
	 - 6703
```


- 启动Storm各个后台进程

主机器：storm nimbus &
从机器：storm supervisor &
主机器（ui运行）：storm ui &
主机器 （logviewer运行）：storm logviewer & (查看工作日志)
然后再浏览器中输入主机器的ip：可以打开ui


## 第二部分 Storm 基础概念

### 2.1 The Storm data model

Storm 应用中运行的基本数据单元被称为**tuple**, Each tuple consists of a predefined（预先定义的） list of fields.
The value of each field can be a byte,char, long ,float,double , Boolean , or byte array. Strom also provides an API to define your own data types, which can be serialized as fields in a  tuple.
总结就是任意可以被序列化的对象都可以看作tuple中的fields，而tuple就是一个fields 的list。

A tuple is dynamically typed ,that is , you just need to define the names of the fields in a tuple and not their data Type. The choice of dynamic typing helps to simplify he API and make it easy to use. Also, since a  processing unit in Storm can process multiple types of tuples, it's not practical to declare field types. 就是说tuple不用预先指定类型，storm支持动态的类型机制，这样是为了使API变得更加容易使用。

可以使用`getValueByField(String)`来通过名字获取或在list中的下标(`index`)方法`getValue(int)`来获取。
tuple也提供了`getIntegerByField(String)`来自动帮你`cast`。
详细的Tuple javadoc在 [Tuple](https://storm.incubator.apache.org/apidocs/backtype/storm/tuple/Tuple.html)

#### 2.1.1 Definition of a Storm topology
In Storm terminology, a topology is an abstraction that defines the graph of the computation.
You create a Storm topology and deploy it on a Storm cluster to process the data. A topology can be represented by a direct acyclic graph, where each node does some kind of processing and forwards it to the next node(s) in the flow.  一个Storm的拓扑就是一个图计算的抽象，你创建一个Storm拓扑，然后把它部署到节点上去运行。 一个可以用一个有向无环图来表现。

The following are the components of a Storm topology.

**Stream:** A Stream is an unbounded sequence of tuples that can be processed in parallel by Storm. Each stream can be processed by a single or multiple types of bolts. Thus, Storm can also be viewed as a platform to transform streams. In the preceding diagram, streams are represented by arrows.

Each stream in a Storm application is given an ID and the bolts can produce 






## 第三部分 API

首先来看一个大概：

- **Topology** 拓扑,有向图
- **Stream grouping** 流分组，数据的分发方式
- **Spout** 喷口，消息源
- **Bolt** 螺栓、处理器
- **Worker** 工作进程
- **Executor** 执行器，Task的线程
- **Task** 具体的执行任务
- **Configuration** 配置

注意理解 Worker Executor Task 

	一个woker可以理解为一个JVM，假如你有3台机器，就可以搞成3个worker
	
	一个worker中可以设置更细粒度的并行度，加入你有2个worker，如果executor设置为2的话，那么一个worker上应该是一个executor thread
	
	一个executor上可以设置更细粒度的线程task，继续上一个例子，有2个worker，2个executor，此时task numbers 设置为4，那么一个worker（jvm）中有1个executor（Thread），每个thread pool上有2个task，每个task应该有一个线程去执行。

	一般而言，tasks一般都不配置。默认情况下, 一个执行器执行一个任务，但是如果指定了任务的数目，任务会平均分配到执行器中。



