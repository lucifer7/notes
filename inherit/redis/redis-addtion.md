# redis基础

## 文档
[jedis-tests](https://github.com/xetorthio/jedis/tree/master/src/test/java/redis/clients/jedis/tests)
[thought-jedis-pool](https://github.com/xetorthio/jedis/issues/738)

[TOC]

## 一、redis install

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

## 二、redis 基础

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

*   多个单词之间以`.`或者`:`来分隔

*   Key的命名，应该在可读的情况下，尽量简短

        ><span class="hljs-keyword">set</span> user:<span class="hljs-number">001</span>:<span class="hljs-property">name</span> <span class="hljs-string">"carl"</span>
    `</pre>

    **Redis的Value支持五种类型**

1.  **String**：字符串，可以存储String、Integer、Float型的数据，甚至是二进制数据，一个字符串最大容量是512M
2.  **List**：字符串List，底层实现上不是数组，而是链表，也就是说在头部和尾部插入一个新元素，其时间复杂度是常数级别的；其弊端是：元素定位比数组慢
3.  **Set**：字符串Set，无序不可重复，是通过HashTable实现的
4.  **Hash**：按Hash方式来存放字符串
5.  **ZSet**：字符串Set，有序且不可重复，根据Score来排序。底层使用散列表和跳跃表来实现，所以读取中间部分数据也很快

    ## 三、redis 基础命令

    [redis中文网](http://www.redis.cn/commands.html)

    ### 关于redis的超时命令

1.  **EXPIRE** key seconds:

        *   if the key is already out of date, remove automatically
    *   set/update the expire time of the key
2.  **PTTL** key : return ttl as ms

        *   if key not exist return -2
    *   if key have no expire time, return -1
    *   return the expire time (ms)
3.  **PEXPIREAT** key milliseconds-timestamp:

        *   if operation is successful, return 1
    *   else key not exist or can't do it ,return 0
4.  **PERSIST** key

        *   if key not exist or key have no expire time, return 0
    *   return 1 if success

    ## 四、redis 集群

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

   
## 五、实战

### 5.1 redis缓存数据行

**思路：**
1. redis实战一书中提供的思路是利用后台进程去同步，具体参考 
2. 自定义方式01，使用一个hash去保存一行对象，同时使用一个list或者set去保存对应的id
3. 自定义方式02，考虑使用hash结构，每个key是id，value是序列化后的对象比如json
关键代码如下:
```java
package com.carl.demo.redis.service;

import com.carl.demo.redis.dao.GoodsDao;
import com.carl.demo.redis.vo.Goods;
import com.carl.demo.util.DateUtils;
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Transaction;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by carl on 2016/3/9.
 */
@Service
@Transactional
public class GoodsRedisServiceImpl {
    @Autowired
    private GoodsDao goodsDao;

    private static final Log LOG = LogFactory.getLog(GoodsRedisServiceImpl.class);

    /**
     * <pre>
     *     1.先保存goods到db
     *     2.如果保存成功，将goods存入hash，并将id保存到list中
     * </pre>
     *
     * @param m
     */
    public void save(Goods m) {
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = jp.getResource();
        Transaction tran = jr.multi();
        Map<String, String> map = new HashMap<String, String>();
        goodsDao.save(m);
        map.put("id", "" + m.getId());
        map.put("name", m.getName());
        map.put("imgPath", m.getImgPath());
        map.put("createTime", DateUtils.formatDate(m.getCreateTime(), "yyyy-MM-dd hh:mm:ss"));
        //如果数据库发生异常，直接抛出运行期错误，程序终止，不会插入redis
        try {
            tran.hmset("GoodsH:" + m.getId(), map);
            tran.lpush("GoodsUuidsL", "" + m.getId());
            tran.exec();
        } catch (Exception e) {
            tran.discard();
        } finally {
            jp.returnResourceObject(jr);
        }
    }

    public void update(Goods m) {
        if (m.getId() == null) {
            return;
        }
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = jp.getResource();
        String key = "GoodsH:" + m.getId();
        if (!jr.exists(key)) {
            return;
        }
        goodsDao.save(m);
        Transaction tran = jr.multi();
        try {
            Map<String, String> map = new HashMap<String, String>();
            map.put("id", "" + m.getId());
            map.put("name", m.getName());
            map.put("imgPath", m.getImgPath());
            map.put("createTime", DateUtils.formatDate(m.getCreateTime(), "yyyy-MM-dd hh:mm:ss"));
            tran.hmset(key, map);
            tran.exec();
        } catch (Exception e) {
            tran.discard();
        } finally {
            jp.returnResourceObject(jr);
        }
    }

    public Goods findById(int id) {
        JedisPool jp = RedisHelper.getPool();
        String key = "GoodsH:" + id;
        Jedis jr = null;
        Goods goods = null;
        try {
            jr = jp.getResource();
            goods = _findById(jr, key);
        } catch (Exception e) {
            LOG.warn(e);
        } finally {
            jp.returnResourceObject(jr);
        }
        return goods;
    }


    private Goods _findById(Jedis jr, String key) {
        Goods ret = new Goods();
        Map<String, String> map = jr.hgetAll(key);
        if (map != null && map.size() > 0) {
            ret = new Goods();
            ret.setId(Integer.valueOf(map.get("id")));
            ret.setCreateTime(DateUtils.parseDate(map.get("createTime")));
            ret.setImgPath(map.get("imgPath"));
            ret.setName(map.get("name"));
        }
        return ret;
    }

    /**
     * <pre>
     *     这样实现太蛋疼，不过体验了个list,考虑使用第二种思路更简单
     * </pre>
     * @return
     */
    public List<Goods> findAll() {
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = null;
        String key = "GoodsUuidsL";
        List<Goods> ret = null;
        try {
            jr = jp.getResource();
            List<String> ids = null;
            if (jr.exists(key))
                ids = jr.lrange(key, 0, -1);
            if (ids != null && ids.size() > 0) {
                //非常麻烦，每个都要拿出来
                ret = new ArrayList<>(ids.size());
                for (int i = 0; i < ids.size(); i++) {
                    key = "GoodsH:" + ids.get(i);
                    Goods g = _findById(jr, key);
                    if (g != null)
                        ret.add(g);
                }
            }
        } catch (Exception e) {
            LOG.warn(e);
        } finally {
            jp.returnResourceObject(jr);
        }
        System.out.println(ret);
        return ret;
    }

}

```
> 关于redis的事务原理：redis是一个单进程，基于包装epoll()函数的内存数据库，将事务中的命令放到一个队列里，单进程保证队列中的命令可以顺序执行，并且提供了Watch方法，基于以下，可以发现redis的事务保证了以下：
> **顺序执行**
> **原子性**
> **不满足利用get参与判断的并发范式**
> **CAS**：Watch key 乐观锁的思路



### 5.2 使用redis记录最新访问的商品

```java
package com.carl.demo.redis.service;

import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Transaction;

/**
 * Created by carl on 16-3-10.
 */
@Service
public class LatestService {

    /**
     * @param goodUuid 被浏览商品
     * @param userId   浏览商品的用户
     */
    public void viewGood(String goodUuid, String userId) {
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = null;
        Transaction tran = null;
        try {
            jr = jp.getResource();
            tran = jr.multi();
            //记录商品被浏览了多少次
            tran.zincrby("viewed:", 1, userId);
            //记录该用户浏览了多少商品
            long now = System.currentTimeMillis();
            tran.zadd("viewed:" + userId, now, goodUuid);
            //只保留该用户浏览的前50个商品

            tran.zremrangeByRank("viewed:" + userId, 0, -10);
            tran.exec();
        } catch (Exception e) {
            tran.discard();
        } finally {
            jp.returnResourceObject(jr);
        }
    }

}

```


###  5.3 使用redis当作计数器：

**思路：**使用`hash`去保存每个时间维度的数据，用`set`去保存所有的记录

```java
package com.carl.demo.redis.service;

import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Transaction;

import java.util.Comparator;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;

/**
 * Created by carl on 2016/3/10.
 */
@Service
public class CounterService {

    private static final int[] PRECISION = new int[]{1, 5, 60, 300, 3600, 18000, 86400};
	
	/**
	* 当有url访问的时候更新计数器
	**/
    public void updateCounter(String url) {
        int now = (int) (System.currentTimeMillis() / 1000L);
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = null;
        Pipeline pp = null;
        try {
            jr = jp.getResource();
            pp = jr.pipelined();
            for (int i = 0; i < PRECISION.length; i++) {
                int prec = PRECISION[i];
                int pnow = (now / prec) * prec;
                String hash = prec + ":" + url;
                pp.zadd("known:", 0, hash);
                pp.hincrBy("count:" + hash, pnow + "", 1);
            }
            pp.sync();
        } catch (Exception e) {
        } finally {
            jp.returnResourceObject(jr);
        }
    }

    /**
     * 通过prec和url查找对应的记录
     * @param prec
     * @param url
     */
    public void getCounter(int prec, String url) {
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = jp.getResource();
        String hash = "count:" + prec + ":" + url;
        Map<String, String> map = jr.hgetAll(hash);
        TreeMap<String, String> ret = new TreeMap<String, String>(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                int i1 = 0;
                int i2 = 0;
                try {
                    i1 = Integer.valueOf(o1);
                    i2 = Integer.valueOf(o2);
                } finally {
                    return i1 - i2;
                }
            }
        });
        ret.putAll(map);
        System.out.println(ret);
        jp.returnResourceObject(jr);
    }

    /**
     * 精灵线程，即守护线程，去检测是否数据量超标
     */
    public void cleanCounters() {
        JedisPool jp = RedisHelper.getPool();
        Jedis jr = null;
        int index = 0;
        while (true) {
            int start = (int) (System.currentTimeMillis() / 1000L);
            Set<String> hash = jr.zrange("known:", index, index);
            index++;
            //如果konwn：中有内容
            if (hash != null && hash.size() > 0) {

            }
        }
    }
}

```


### 5.4 使用Redis实现分布式锁

