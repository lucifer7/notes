# redis基础

[TOC]

## redis install

*   去官网下载最新的版本：[http://redis.io/download](http://redis.io/download) ，这里用的是3.0.2

*   解压后，进入解压好的文件夹

*   redis的安装非常简单，因为已经有现成的Makefile文件，所以直接先make，然后make install就

可以了

*   安装的位置在/usr/local/bin ，有：

1.  redis-benchmark：性能测试工具，测试Redis在你的系统及配置下的读写性能

2.  redis-check-aof：用于修复出问题的AOF文件

3.  redis-check-dump：用于修复出问题的dump.rdb文件

4.  redis-cli：Redis命令行操作工具

5.  redis-sentinel：Redis集群的管理工具

6.  redis-server：Redis服务器启动程序

*   启动Redis的时候，只有一个参数，就是指定配置文件redis.conf的路径。redis.conf在解压的文

件夹里面有，复制一个出来，按需修改即可，也可--port来指定端口

*   连接Redis并操作，使用redis-cli，如果有多个实例，可以redis-cli -h 服务器ip -p 端口

*   关闭Redis，redis-cli shutdown，如果有多个实例，可以指定端口来关闭：redis-cli -p 6379

shutdown

## redis 基础

### 单进程

1.  Redis的服务器程序采用的是_单进程模型_来处理客户端的请求。对读写等事件的响应

     是通过对epoll函数的包装来做到的。

2.  Redis的实际处理速度完全依靠主进程的执行效率，假如同时有多个客户端并发访问

     服务器，则服务器处理能力在一定情况下将会下降。假如你要提升服务器的并发能力，那

     么可以采用在单台机器部署多个redis进程的方式。
> 采用多个端口依旧会造成cpu的竞争状况

### 多数据库

1.  Redis每个数据库对外都是以从0开始递增的数字来命名，默认16个数据库，默认使用0号数

据库，可以使用Select 数字来选择要使用的数据库

1.  使用Dbsize可以查看当前数据库的key的数量

2.  可以在多个数据库间移动数据，使用move key 目的数据库编号就可以了

3.  使用flushdb可以清除某个数据库的数据

4.  Redis不支持自定义数据库名字

5.  Redis不支持为每个数据库设置不同的访问密码

6.  多个数据库之间并不是完全独立的，FlushAll可以清空全部的数据

7.  Redis的数据库更像是一个命名空间

### key - &gt; value

**Redis的key是字符串类型，如果中间有空格或者转义字符等，要用“”**

*   命名建议：对象类型:对象ID:对象属性

*   多个单词之间以“.”来分隔

*   Key的命名，应该在可读的情况下，尽量简短

        ><span class="hljs-keyword">set</span> user:<span class="hljs-number">001</span>:<span class="hljs-property">name</span> <span class="hljs-string">"carl"</span>
    `</pre>

    **Redis的Value支持五种类型**

1.  String：字符串，可以存储String、Integer、Float型的数据，甚至是二进制数

    据，一个字符串最大容量是512M

1.  List：字符串List，底层实现上不是数组，而是链表，也就是说在头部和尾部插

    入一个新元素，其时间复杂度是常数级别的；其弊端是：元素定位比数组慢

1.  Set：字符串Set，无序不可重复，是通过HashTable实现的
2.  Hash：按Hash方式来存放字符串
3.  ZSet：字符串Set，有序且不可重复，根据Score来排序。底层使用散列表和跳跃

    表来实现，所以读取中间部分数据也很快

    ## redis 基础命令

    [redis中文网](http://www.redis.cn/commands.html)

    ### 关于redis的超时命令

1.  EXPIRE key seconds:

        *   if the key is already out of date, remove automatically
    *   set/update the expire time of the key
2.  PTTL key : return ttl as ms

        *   if key not exist return -2
    *   if key have no expire time, return -1
    *   return the expire time (ms)
3.  PEXPIREAT key milliseconds-timestamp:

        *   if operation is successful, return 1
    *   else key not exist or can't do it ,return 0
4.  PERSIST key

        *   if key not exist or key have no expire time, return 0
    *   return 1 if success

    ## redis 集群

    ### 手工创建一个集群

1.  首先进行集群配置

    只需要将每个数据库的cluster-enabled配置选项打开，然后再修改如下内容：pidfile、port、logfile、dbfilename、cluster-config-file

1.  分别启动这些Redis数据库，可以用info cluster查看信息
2.  连接节点，使用cluster meet，把所有的数据库都放到一个集群中来
     ```
     cluster info
     cluster meet 127.0.0.1 6381
     cluster meet 127.0.0.1 6382
     cluster meet 127.0.0.1 6383
     cluster meet 127.0.0.1 6384
     cluster meet 127.0.0.1 6385
     cluster meet 127.0.0.1 6386
     cluster nodes:
     eed06b97dc01eab6366948bade2ba95ec60db83e 127.0.0.1:6383 master - 0 1450055027319 0 connected

    d38d21f515afc3cf0bf2b45a747a78b69a0db61a 127.0.0.1:6386 slave - 0 1450055022271 3 connected

    b2bf82f55e64ecebada2204e7f1561a43c681390 127.0.0.1:6382 master - 0 1450055028328 2 connected

    e6dc9830efe57045450b799674af6b2cadf35eaa 127.0.0.1:6384 master - 0 1450055025803 3 connected

    3c848f10569d9ec22943f297ca69fabcfb884b00 127.0.0.1:6381 myself,master - 0 0 1 connected

    a9537157a65a179edea04937937b1b8e79be2889 127.0.0.1:6385 slave - 0 1450055026308 1 connected

    <pre>`<span class="hljs-escape">``</span>`
    `</pre>

1.  可以通过cluster info ，或者cluster nodes 查看信息
2.  设置部分数据库为slave，使用cluster replicate，例如在6384上设置为6381的复制集
     ```
     cluster nodes
     cluster replicate 3c848f10569d9ec22943f297ca69fabcfb884b00
     cluster nodes:
     69539d9808ce356b8c967a448d6199dd4846ecb2 127.0.0.1:6384 myself,slave 81e23d1c2af1dd3c4d066fb31f065ed54de3da2c 0 0 3 connected
    <pre>`<span class="hljs-escape">``</span>`
    `</pre>

1.  然后就该来分配插槽了，使用cluster addSlots，这个命令目前只能一个一个加，如果要加区间的话，就得客户端编写代码循环添加。有个实用的技巧：把所有的Redis停下来，然后直接修改node的配置文件，只需要配置master的数据库就可以，然后再重启数据库。分配完插槽，可以使用cluster slots查看信息。
2.  通过cluster info查看集群信息，如果显示ok，那就可以使用了
    > 修改配置文件nodes.conf,手动分配0-16383 这16384个节点

    下面我们可以讨论这种集群的优劣

    ### redis服务器的集群

    **常见的分片实现：**

1.  在客户端进行分片
2.  通过代理来进行分片，比如：Twemproxy
3.  查询路由：就是发送查询到一个随机实例，这个实例会保证转发你的查询到正确的节点，

    Redis集群在客户端的帮助下，实现了查询路由的一种混合形式，请求不是直接从Redis实

    例转发到另一个，而是客户端收到重定向到正确的节点

1.  在服务器端进行分片， Redis采用哈希槽（hash slot）的方式在服务器端进行分片：

    Redis集群有16384个哈希槽，使用键的CRC16编码对16384取模来计算一个键所属的哈希槽

    **redis集群的缺点**

    > 所谓缺点，就是要想办法绕过啊
> 
> 1.  不支持涉及多键的操作，如mget，如果所操作的键都在同一个节点，就正常执行，否则会提示错误
> 2.  分片的粒度是键，因此每个键对应的值不要太大
> 3.  数据备份会比较麻烦，备份数据时你需要聚合多个实例和主机的持久化文件
> 4.  容的处理比较麻烦
> 5.  故障恢复的处理会比较麻烦，可能需要重新梳理Master和Slave的关系，并调整每个复制集里面的数据

    **redis3.0集群**

1.  在以前版本中，Redis的集群是依靠客户端分片来完成，但是这会有很多缺点，比如维护成本高，需要客

    户端编码解决；增加、移出节点都比较繁琐等

1.  Redis3.0新增的一大特性就是支持集群，在不降低性能的情况下，还提供了网络分区后的可访问性和支

    持对主数据库故障的恢复。

1.  使用集群后，都只能使用默认的0号数据库
2.  每个Redis集群节点需要两个TCP连接打开，正常的TCP端口用来服务客户端，例如6379，加10000的端口

    用作数据端口，必须保证防火墙打开这两个端口

1.  Redis集群不保证强一致性，这意味着在特定的条件下，Redis集群可能会丢掉一些被系统收到的写入请

    求命令。

    **redis集群架构**

*   所有的Redis节点彼此互联，内部使用二进制协议优化传输速度和带宽
*   节点的fail是通过集群中超过半数的节点检测失效时才生效

*   客户端与Redis节点直连，不需要中间proxy层。客户端不需要连接集群所有节点，连接集

    群中任何一个可用节点即可

*   集群把所有的物理节点映射到[0-16383]插槽上，集群负责维护：节点-插槽-值的关系

    ### redis集群的插槽

    **键名的有效部分**

*   如果键名包含{}，那么有效部分就是{}中的值
*   否则就是取整个键名

    **移动已经分配的插槽**

    这个稍微麻烦点，尤其是有了数据过后，假设要迁移123号插槽从A到B，大致步骤如下：

1.  在B上执行cluster setslot 123 importing A
2.  在A上执行cluster setslot 123 migrating B
3.  在A上执行cluster getkeysinslot 123 要返回的数量
4.  对上一步获取的每个键执行migrate命令，将其从A迁移到B
5.  在集群中每个服务器上执行cluster setslot 123 node B

    **故障处理**

*   集群中每个节点都会定期向其他节点发出ping命令，如果没有收到回复，就认为

    该节点为疑似下线，然后在集群中传播该信息

*   当集群中的某个节点，收到半数以上认为某节点已下线的信息，就会真的标记该

    节点为已下线，并在集群中传播该信息

*   如果已下线的节点是master节点，那就意味着一部分插槽无法写入了
*   如果集群任意master挂掉，且当前master没有slave，集群进入fail状态
*   如果集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态
*   当集群不可用时，所有对集群的操作做都不可用，收到CLUSTERDOWN The

    cluster is down错误信息

    **自动故障恢复**

    > 发现某个master下线后，集群会进行故障恢复操作，来将一个slave变成master，基
> 
>     于Raft算法，大致步骤如下

1.  某个slave向集群中每个节点发送请求，要求选举自己为master
2.  如果收到请求的节点没有选举过其他slave，会同意
3.  当集群中有超过节点数一半的节点同意该slave的请求，则该Slave选举成功
4.  如果有多个slave同时参选，可能会出现没有任何slave当选的情况，将会等待一个随机时

    间，再次发出选举请求

1.  选举成功后，slave会通过slaveof no one命令把自己变成master如果故障后还想集群继续工作，可设置cluster-require-full-coverage为no，默认yes
    > **故障恢复说明**
> 
> *   master挂掉了，重启还可以加入集群，但挂掉的slave重启，如果对应的master变化了，是
> 
>     不能加入集群的，除非修改它们的配置文件，将其master指向新master
> 
> *   只要主从关系建立，就会触发主和该从采用save方式持久化数据，不论你是否禁止save
> *   在集群中，如果默认主从关系的主挂了并立即重启，如果主没有做持久化，数据会完全丢
> 
>     失，从而从的数据也被清空

    ### 使用redis-trib.rb 来操作集群

    **使用脚本来创建集群**

1.  安装Ruby

        *   下载ruby安装包，地址[https://www.ruby-lang.org/en/downloads/](https://www.ruby-lang.org/en/downloads/)
    *   然后分别configure、make、make install
    *   安装后通过ruby -v 查看一下版本，看是否正常
2.  还需要安装rubygems

        *   下载包，地址[https://rubygems.org/pages/download](https://rubygems.org/pages/download)

*   解压后进入解压文件夹，运行ruby setup.rb
*   安装后通过gem –v查看一下版本，看是否正常

1.  还需要安装redis的ruby library

*   由于连接国外源不太稳定，请先删除，如gem sources --remove

    [https://rubygems.org/](https://rubygems.org/) ，然后添加gem sources -a [https://ruby.taobao.org/](https://ruby.taobao.org/)

*   可以通过gem sources -l 查看源
*   运行gem install redis

1.  使用redis-trib.rb来初始化集群，形如：

    `ruby redis-trib.rb create --replicas 1 127.0.0.1:6381

    127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385

    127.0.0.1:6386`

    create表示要初始化集群，--replicas 1表示每个驻数据库拥有的从数据

    库为1个

1.  使用redis-trib.rb来迁移插槽，如下：

*   执行ruby redis-trib.rb reshard ip:port ，这就告诉Redis要重新分片，

    ip:port可以是集群中任何一个节点

*   然后按照提示去做就可以了
*   这种方式不能指定要迁移的插槽号

    ## java client

    ### use pipeline or transaction

    <pre>`

    <span class="cs">
    <span class="hljs-keyword">protected</span> Jedis jedis;

    @<span class="hljs-function">Before

    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{

        jedis = <span class="hljs-keyword">new</span> Jedis(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">6379</span>);

        beforeAllMethod();

    }

    @<span class="hljs-function">Test

        <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testPipeLine</span><span class="hljs-params">()</span> </span>{

            Pipeline pp = jedis.pipelined();

            pp.<span class="hljs-keyword">set</span>(<span class="hljs-string">"k1"</span>, <span class="hljs-string">"1111"</span>);

            Response<String> r1 = pp.<span class="hljs-keyword">get</span>(<span class="hljs-string">"k1"</span>);

            pp.<span class="hljs-keyword">set</span>(<span class="hljs-string">"k1"</span>, <span class="hljs-string">"22222"</span>);

            Response<String> r2 = pp.<span class="hljs-keyword">get</span>(<span class="hljs-string">"k1"</span>);

            pp.sync();

            System.<span class="hljs-keyword">out</span>.println(<span class="hljs-string">"k1-1"</span> + r1.<span class="hljs-keyword">get</span>());

            System.<span class="hljs-keyword">out</span>.println(<span class="hljs-string">"k1-2"</span> + r2.<span class="hljs-keyword">get</span>());

        }

        @<span class="hljs-function">Test

        <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testTran</span><span class="hljs-params">()</span> </span>{

            jedis.watch(<span class="hljs-string">"k1"</span>);

            jedis.<span class="hljs-keyword">set</span>(<span class="hljs-string">"k1"</span>, <span class="hljs-string">"error"</span>);

            Transaction tran = jedis.multi();

            Response<String> r1 = tran.<span class="hljs-keyword">get</span>(<span class="hljs-string">"k1"</span>);

            tran.<span class="hljs-keyword">set</span>(<span class="hljs-string">"k1"</span>, <span class="hljs-string">"2222"</span>);

            Response<String> r2 = tran.<span class="hljs-keyword">get</span>(<span class="hljs-string">"k1"</span>);

            tran.exec();

            System.<span class="hljs-keyword">out</span>.println(r1.<span class="hljs-keyword">get</span>());

            System.<span class="hljs-keyword">out</span>.println(r2.<span class="hljs-keyword">get</span>());

            jedis.disconnect();

        }</span>
    `</pre>

    ### redis with common pool

*   maxTotal,maxIdle,minIdle
*   whenExhaustedAction:
     1) FAIL: throw NoSuchElementException
     2) BLOCK
     3) GROW: maxTotal useless

*   maxWaitMillis
*   testOnBrrow,testOnReturn,testWhileIdle
*   timeBetweenEvictRunsMillis: time between testWhileIdle
*   numTestPerEvictionRun: numbers per test while idle
*   lifo: true
*   minEvictableIdle
    <pre>````

     @<span class="hljs-function">Test

    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">test</span><span class="hljs-params">()</span> </span>{

        jcfg = <span class="hljs-keyword">new</span> JedisPoolConfig();

        jcfg.setMaxTotal(<span class="hljs-number">10</span>);

        jcfg.setMinIdle(<span class="hljs-number">3</span>);

        JedisPool pool = <span class="hljs-keyword">new</span> JedisPool(jcfg, <span class="hljs-string">"localhost"</span>, <span class="hljs-number">6379</span>);

        Jedis jedis = pool.getResource();

        <span class="hljs-keyword">try</span> {

            System.<span class="hljs-keyword">out</span>.println(jedis.<span class="hljs-keyword">get</span>(<span class="hljs-string">"k2"</span>));

        } <span class="hljs-keyword">catch</span> (Exception e) {

        } <span class="hljs-keyword">finally</span> {

            pool.returnResourceObject(jedis);

        }

        pool.close();

    }

    ```
    `</pre>

    ### cluster

    maybe there still exist some exceptions when use jedis.

    ## 数据安全与性能保障

    ### RDB

1.  **以下操作会引起RDB**
    <pre>`-<span class="ruby"> <span class="hljs-constant">BGSAVE</span> command
    </span>
    -<span class="ruby"> <span class="hljs-constant">SAVE</span> command
    </span>
    -<span class="ruby"> save configuration： 比如save <span class="hljs-number">60</span> <span class="hljs-number">10000</span> <span class="hljs-string">"在60s内有10000次写入"</span>时触发<span class="hljs-constant">BGSAVE</span>命令
    </span>
    -<span class="ruby"> redis通过shutdown命令收到关闭服务端请求时，会执行一个<span class="hljs-constant">SAVE</span>命令
    </span>
    -<span class="ruby"> redis服务器连接另一个服务器，<span class="hljs-constant">SYNC</span>发送的时候，如果主服务器没有执行<span class="hljs-constant">BGSAVE</span>操作，或者主服务器并非刚刚执行完<span class="hljs-constant">BGSAVE</span>操作，那么主服务器会执行<span class="hljs-constant">BGSAVE</span>命令</span>
    `</pre>

1.  大数据

        *   数据量比较大，占用内存比较多，如果**空闲内存不多**,BGSAVE会引起系统长时间的停顿，也可能引发系统长时间的停顿，甚至无法使用
    *   执行BGSAVE的停顿时间在于系统，一般1g内存额外增加10-20ms
    *   save命令不需要额外的进程，但是会堵塞主要进程
    *   rdb造成的数据丢失
    > tips:
> 
> 1.  save 全部注释掉可以关闭rdb的BGSAVE功能
> 2.  一般而言都是使用bgsave
> 3.  如果不怎么需要备份：可以写一个脚本，比如凌晨3点启动一次SAVE命令，停止所有客户端的访问请求

1.  关于rdb的配置
    <pre>`
    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># Save the DB on disk:</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   save <seconds> <changes></span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   Will save the DB if both the given number of seconds and the given</span>

    <span class="hljs-preprocessor">#   number of write operations against the DB occurred.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   In the example below the behaviour will be to save:</span>

    <span class="hljs-preprocessor">#   after 900 sec (15 min) if at least 1 key changed</span>

    <span class="hljs-preprocessor">#   after 300 sec (5 min) if at least 10 keys changed</span>

    <span class="hljs-preprocessor">#   after 60 sec if at least 10000 keys changed</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   Note: you can disable saving completely by commenting out all "save" lines.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   It is also possible to remove all the previously configured save</span>

    <span class="hljs-preprocessor">#   points by adding a save directive with a single empty string argument</span>

    <span class="hljs-preprocessor">#   like in the following example:</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor">#   save ""</span>

    <span class="hljs-preprocessor">#save 900 1</span>

    <span class="hljs-preprocessor">#save 300 10</span>

    <span class="hljs-preprocessor">#save 60 10000</span>

    <span class="hljs-preprocessor"># By default Redis will stop accepting writes if RDB snapshots are enabled</span>

    <span class="hljs-preprocessor"># (at least one save point) and the latest background save failed.</span>

    <span class="hljs-preprocessor"># This will make the user aware (in a hard way) that data is not persisting</span>

    <span class="hljs-preprocessor"># on disk properly, otherwise chances are that no one will notice and some</span>

    <span class="hljs-preprocessor"># disaster will happen.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># If the background saving process will start working again Redis will</span>

    <span class="hljs-preprocessor"># automatically allow writes again.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># However if you have setup your proper monitoring of the Redis server</span>

    <span class="hljs-preprocessor"># and persistence, you may want to disable this feature so that Redis will</span>

    <span class="hljs-preprocessor"># continue to work as usual even if there are problems with disk,</span>

    <span class="hljs-preprocessor"># permissions, and so forth.</span>

    stop-writes-on-bgsave-error yes

    <span class="hljs-preprocessor"># Compress string objects using LZF when dump .rdb databases?</span>

    <span class="hljs-preprocessor"># For default that's set to 'yes' as it's almost always a win.</span>

    <span class="hljs-preprocessor"># If you want to save some CPU in the saving child set it to 'no' but</span>

    <span class="hljs-preprocessor"># the dataset will likely be bigger if you have compressible values or keys.</span>

    rdbcompression yes

    <span class="hljs-preprocessor"># Since version 5 of RDB a CRC64 checksum is placed at the end of the file.</span>

    <span class="hljs-preprocessor"># This makes the format more resistant to corruption but there is a performance</span>

    <span class="hljs-preprocessor"># hit to pay (around 10%) when saving and loading RDB files, so you can disable it</span>

    <span class="hljs-preprocessor"># for maximum performances.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># RDB files created with checksum disabled have a checksum of zero that will</span>

    <span class="hljs-preprocessor"># tell the loading code to skip the check.</span>

    rdbchecksum yes

    <span class="hljs-preprocessor"># The filename where to dump the DB</span>

    dbfilename dump.rdb

    <span class="hljs-preprocessor"># The working directory.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># The DB will be written inside this directory, with the filename specified</span>

    <span class="hljs-preprocessor"># above using the 'dbfilename' configuration directive.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># The Append Only File will also be created inside this directory.</span>

    <span class="hljs-preprocessor">#</span>

    <span class="hljs-preprocessor"># Note that you must specify a directory here, not a file name.</span>

    dir ./
    `</pre>

    ### AOF

    `appendonly yes` 开启aof

    `appendfsync`

*   always
*   everysec
*   no
    > 使用固态银盘要慎用always选项，可能将几年寿命的固态硬盘缩短为几个月
    <pre>`
    <span class="hljs-comment"># By default Redis asynchronously dumps the dataset on disk. This mode is</span>

    <span class="hljs-comment"># good enough in many applications, but an issue with the Redis process or</span>

    <span class="hljs-comment"># a power outage may result into a few minutes of writes lost (depending on</span>

    <span class="hljs-comment"># the configured save points).</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># The Append Only File is an alternative persistence mode that provides</span>

    <span class="hljs-comment"># much better durability. For instance using the default data fsync policy</span>

    <span class="hljs-comment"># (see later in the config file) Redis can lose just one second of writes in a</span>

    <span class="hljs-comment"># dramatic event like a server power outage, or a single write if something</span>

    <span class="hljs-comment"># wrong with the Redis process itself happens, but the operating system is</span>

    <span class="hljs-comment"># still running correctly.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># AOF and RDB persistence can be enabled at the same time without problems.</span>

    <span class="hljs-comment"># If the AOF is enabled on startup Redis will load the AOF, that is the file</span>

    <span class="hljs-comment"># with the better durability guarantees.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># Please check http://redis.io/topics/persistence for more information.</span>

    <span class="hljs-title">appendonly</span> <span class="hljs-built_in">no</span>

    <span class="hljs-comment"># The name of the append only file (default: "appendonly.aof")</span>

    appendfilename <span class="hljs-string">"appendonly.aof"</span>

    <span class="hljs-comment"># The fsync() call tells the Operating System to actually write data on disk</span>

    <span class="hljs-comment"># instead of waiting for more data in the output buffer. Some OS will really flush</span>

    <span class="hljs-comment"># data on disk, some other OS will just try to do it ASAP.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># Redis supports three different modes:</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># no: don't fsync, just let the OS flush the data when it wants. Faster.</span>

    <span class="hljs-comment"># always: fsync after every write to the append only log. Slow, Safest.</span>

    <span class="hljs-comment"># everysec: fsync only one time every second. Compromise.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># The default is "everysec", as that's usually the right compromise between</span>

    <span class="hljs-comment"># speed and data safety. It's up to you to understand if you can relax this to</span>

    <span class="hljs-comment"># "no" that will let the operating system flush the output buffer when</span>

    <span class="hljs-comment"># it wants, for better performances (but if you can live with the idea of</span>

    <span class="hljs-comment"># some data loss consider the default persistence mode that's snapshotting),</span>

    <span class="hljs-comment"># or on the contrary, use "always" that's very slow but a bit safer than</span>

    <span class="hljs-comment"># everysec.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># More details please check the following article:</span>

    <span class="hljs-comment"># http://antirez.com/post/redis-persistence-demystified.html</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># If unsure, use "everysec".</span>

    <span class="hljs-comment"># appendfsync always</span>

    appendfsync everysec

    <span class="hljs-comment"># appendfsync no</span>

    <span class="hljs-comment"># When the AOF fsync policy is set to always or everysec, and a background</span>

    <span class="hljs-comment"># saving process (a background save or AOF log background rewriting) is</span>

    <span class="hljs-comment"># performing a lot of I/O against the disk, in some Linux configurations</span>

    <span class="hljs-comment"># Redis may block too long on the fsync() call. Note that there is no fix for</span>

    <span class="hljs-comment"># this currently, as even performing fsync in a different thread will block</span>

    <span class="hljs-comment"># our synchronous write(2) call.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># In order to mitigate this problem it's possible to use the following option</span>

    <span class="hljs-comment"># that will prevent fsync() from being called in the main process while a</span>

    <span class="hljs-comment"># BGSAVE or BGREWRITEAOF is in progress.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># This means that while another child is saving, the durability of Redis is</span>

    <span class="hljs-comment"># the same as "appendfsync none". In practical terms, this means that it is</span>

    <span class="hljs-comment"># possible to lose up to 30 seconds of log in the worst scenario (with the</span>

    <span class="hljs-comment"># default Linux settings).</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># If you have latency problems turn this to "yes". Otherwise leave it as</span>

    <span class="hljs-comment"># "no" that is the safest pick from the point of view of durability.</span>

    <span class="hljs-built_in">no</span>-appendfsync-<span class="hljs-built_in">on</span>-rewrite <span class="hljs-built_in">no</span>

    <span class="hljs-comment"># Automatic rewrite of the append only file.</span>

    <span class="hljs-comment"># Redis is able to automatically rewrite the log file implicitly calling</span>

    <span class="hljs-comment"># BGREWRITEAOF when the AOF log size grows by the specified percentage.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># This is how it works: Redis remembers the size of the AOF file after the</span>

    <span class="hljs-comment"># latest rewrite (if no rewrite has happened since the restart, the size of</span>

    <span class="hljs-comment"># the AOF at startup is used).</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># This base size is compared to the current size. If the current size is</span>

    <span class="hljs-comment"># bigger than the specified percentage, the rewrite is triggered. Also</span>

    <span class="hljs-comment"># you need to specify a minimal size for the AOF file to be rewritten, this</span>

    <span class="hljs-comment"># is useful to avoid rewriting the AOF file even if the percentage increase</span>

    <span class="hljs-comment"># is reached but it is still pretty small.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># Specify a percentage of zero in order to disable the automatic AOF</span>

    <span class="hljs-comment"># rewrite feature.</span>

    auto-aof-rewrite-percentage <span class="hljs-number">100</span>

    auto-aof-rewrite-min-size 64mb

    <span class="hljs-comment"># An AOF file may be found to be truncated at the end during the Redis</span>

    <span class="hljs-comment"># startup process, when the AOF data gets loaded back into memory.</span>

    <span class="hljs-comment"># This may happen when the system where Redis is running</span>

    <span class="hljs-comment"># crashes, especially when an ext4 filesystem is mounted without the</span>

    <span class="hljs-comment"># data=ordered option (however this can't happen when Redis itself</span>

    <span class="hljs-comment"># crashes or aborts but the operating system still works correctly).</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># Redis can either exit with an error when this happens, or load as much</span>

    <span class="hljs-comment"># data as possible (the default now) and start if the AOF file is found</span>

    <span class="hljs-comment"># to be truncated at the end. The following option controls this behavior.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># If aof-load-truncated is set to yes, a truncated AOF file is loaded and</span>

    <span class="hljs-comment"># the Redis server starts emitting a log to inform the user of the event.</span>

    <span class="hljs-comment"># Otherwise if the option is set to no, the server aborts with an error</span>

    <span class="hljs-comment"># and refuses to start. When the option is set to no, the user requires</span>

    <span class="hljs-comment"># to fix the AOF file using the "redis-check-aof" utility before to restart</span>

    <span class="hljs-comment"># the server.</span>

    <span class="hljs-comment">#</span>

    <span class="hljs-comment"># Note that if the AOF file will be found to be corrupted in the middle</span>

    <span class="hljs-comment"># the server will still exit with an error. This option only applies when</span>

    <span class="hljs-comment"># Redis will try to read more data from the AOF file but not enough bytes</span>

    <span class="hljs-comment"># will be found.</span>

    aof-load-truncated <span class="hljs-built_in">yes</span>
    