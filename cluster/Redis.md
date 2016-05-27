[TOC]

### Intro
Redis
- 运行在内存中
- 数据结构服务器（data structures server）
- 单进程（除持久化时） -> 一个进程，一个CPU

Ref:
算法时间复杂度收敛

![图片](https://raw.githubusercontent.com/gnuhpc/All-About-Redis/master/DataStructure/timecomplex.gif)

Redis 核心对象
![redis objects](https://pic2.zhimg.com/8aded9b83a20d9d1c26ec1f5afbd78c5_b.png)

### Jedis 连接失败问题
1. java.net.ConnectException: Connection refused: connect
> bind的问题. 把Redis的配置文件redis.conf里
> \#bind localhost  注释掉
2. ERROR: DENIED Redis is running in protected mode
> [root@nginx ~]# redis-cli
  127.0.0.1:6379> CONFIG SET protected-mode no
  OK
3. bind ip [ip...]
network interface?
访问的IP

### 命令行
1. use **scan** instead of **keys**, which is safer

> 127.0.0.1:6379> scan 0

> (scan cursor [Match patter][Count number])

2. **SETEX key seconds value** equals **SET key value** plus **EXPIRE key seconds**

3. **CONFIG GET** 获取所有配置信息, use **CONFIG REWRITE** to save configurations modified by **CONFIG SET**

4. 迁移 数据命令
> MIGRATE host port key destination-db timeout [COPY] [REPLACE]

Start another instance at port 6380:
> $ ./redis-server --port 6380 &

And use redis-cli to check and connect:
>$ ./redis-cli -p 6380

Then do migrate:
>MIGRATE 192.168.1.16 6380 greeting 0 1000 copy

5. 修改密码与 AUTH 认证命令
> 127.0.0.1:6379> config get requirepass

> 127.0.0.1:6379> config set requirepass 123456

Checking and auth:
> 127.0.0.1:6379> config get requirepass

> (error) NOAUTH Authentication required.

> 127.0.0.1:6379> auth 123456

6. redis-cli 连接
> redis-cli -h 127.0.0.1 -p 6379

For help:
> redis-cli --help

6. Redis benchmark
> [root@localhost ~]# redis-benchmark -h 10.200.157.50 -p 6379 

### 订阅/发布机制
- Redis 通过 PUBLISH 、 SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能
- 这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等
- 为了解耦发布者(publisher)和订阅者(subscriber)之间的关系，Redis 使用了 channel (频道)作为两者的中介
![channel for subscribe](http://img.techtarget.com.cn/database/article/2012/2012-08-10-10-08-12.jpg)

### Redis Schema
Redis没有严格意义上的表名和字段名，以　Key-Value　键值对的方式存储，因此一般采用　schema:key　形式做为键值，其中

schema: 可理解为传统数据库中的表名
key: 　　 可理解为表中的主键

因此使用redis存放你的session时，需要一个schema前辍，　比如这个key：　sessionid:i4w3axuzyj4nwwg75y6k5us2

Redis 也仅能对Key进行检索， 尚不支持对Key所存放的Hash Key的检索。

### Redis 集群方案 (三种)
#### 1. Official, Redis Cluster
- 服务器 Sharding
- 使用 slot (槽)，共13284个，一槽一node节点
- 对key 进行散列，分配到某个槽
- defect: 某个 node 故障，集群死球
- 解决： node 配置成主从结构，主节点失效，slave中的一个升为 master（选举算法）
- For client： 整个 cluster 视为整体
- Since 3.0 version

#### 2. Redis Sharding 集群
- 客户端 Shading
- sharding 处理在客户端，服务器端Redis 实例相互独立

#### 3. 代理中间件
- 综合上两者优点
- twemproxy: proxy and key sharding

### 应用
Example：
1. 消息队列（通知类、延迟更新类）
2. 热点数据的实时缓存（比如feed，数据库、缓存同时写）
3. 热点列表数据缓存（首页、热门话题等）
4. counter（计数器，大多是用缓存实现的）
5. 记日志最好不要用redis，用mongodb比较适合。

### TODO LIST
1. 集合内联(sinter,zinterstore)和连接，多表查询 

2. 计数器、bitmaps以及地理位置索引