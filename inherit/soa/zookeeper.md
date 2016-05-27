
# zookeeper
[TOC]

## 帮助文档
[跟着实例zookeeper](http://ifeve.com/zookeeper-path-cache/)

## 一、zookeeper 基础概念

### 1.1 什么是zookeeper
	zookeeper 是一个高效的分布式协调服务，它暴露了一些公用服务，比如命名/配置/同步控制/群组服务等。我们可以使用ZK来实现一些功能，例如：达成共识/集群管理/leader选举等。

	Zookeeper是一个高可用的分布式管理与协调框架，基于ZAB算法（原子消息广播协议）的实现。该框架能够很好的保证数据的一致性。

**特性：**

**顺序一致性：** 从一个客户端发起的事务请求，最终会严格的按照发起顺序被应用到zookeeper中去。一个写请求发送到节点，节点首先同步到其他的节点中，在这个过程中，其他的写请求会一致等待到该请求结束在执行。

**原子性：**所有事务的请求结果在集群中的应用情况是一致的，要不所有机器上都成功，要不所有的机器都失败

**单一视图：**连接到集群中任何一个机器都是一样的

**可靠性：**一旦服务器成功了应用了一个事务，并且完成对客户端的响应，那么该事务引起的服务端变化会被保存，哪怕是集群中网络中通信出现故障，也能保证可靠性

**实时性：**zookeeper仅仅能保证zai一段时间内，客户端能读取到最新的数据状态



### 1.2 zookeeper 的设计

从设计模式的角度上，是一个基于观察者模式的**分布式服务管理框架**，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，zookeeper就将负责同志已经在zookeeper注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式。

**目标1：** 简单的数据结构（树形结构仿照linux结构）
**目标2：**可以构建集群。3台以上的奇数，只要半数以上的机器能够正常工作，那么整个集群就能正常对外服务
**目标3：**顺序访问，对于来自每一个客户端的每一个请求，zookeeper都会分配一个全局的递增编号，这个编号反应了所有事务的**操作顺序**，应用程序可以使用这个特性来实现更高层次的同步
**目标4：**高性能。由于zookeeper将全局数据存储在内存中，并直接服务于所有的非事务请求，因此尤其在读操作为主的场景下性能尤其的突出。在JMeter的压力测试下（100%）读，其结果大概在12~13w的QPS


### 1.3 数据模型：

- 每个子目录如NameService都被称为znode，这个znode是被它所在的路径唯一标识，如Server1这个znode的标识为/NameService/Server1
- **znode可以有子节点目录**，并且每个znode可以存储数据，注意EPHEMERAL类型的目录节点中不能有子节点目录
- znode是有版本的，每个znode中存储的数据可以有**多个版本**，也就是一个访问路径可以存储多份数据
-  znode可以是临时节点，一旦创建这个znode的客户端与服务器失去联系，znode将自动删除，Zookeeper客户端和服务器通信采用**长连接**的方式，每个客户端与服务器通过心跳来保持连接，这个链接状态称为`session`，如果znode是临时节点，这个session失效，znode就删除了
-  znode的目录名可以自动编号，如果App1已经存在，自动命名为App2
-  znode可以被**Watch**，包括这个目录节点中存储数据的修改，子节点目录的变化等，一旦变化可以通知设置监控的客户端，这个是Zookeeper的**核心特性**



### 1.4 组成

`Leader:` 负责write请求

`Follower:`负责read请求，参与leader选举

`Observer:`特殊的Follower，可以接受客户端的read请求，但不参与选举，可以扩容系统的支撑能力，提高对读请求的处理。因为它不接受任何同步的写入请求，**只负责和leader同步数据**。


### 1.5 应用场景


**一、配置管理：** 配置的管理在分布式应用环境中很常见，比如机器的配置列表，运行时的开关配置，数据库配置信息等。这些全局信息通常具备以下3个特性：
**数据量比较小**
**数据内容在运行时动态变化**
**集群中各个集群共享信息，配置一致**



**二、集群管理**
Zookeeper不仅能帮你维护当前的机器中机器的服务状态，而且能帮你选出一个总管来管理集群，并且实现集群的容错功能
1. 希望知道当前集群中有多少机器正在工作
2. 对集群中每天集群的运行时状态进行数据采集
3. 对集群中每台集群进行上下线操作

**三、发布和订阅**

**四、数据库的自动切换**
	当我们初始化zookeeper的时候读取其节点上的数据库配置文件，当配置一旦发生变更时，zookeeper就能帮助我们把变更的通知发送到各个客户端。

**五、分布式日志收集**

**六、分布式场景的高可用**
	原声api实现分布式供能也非常困难，可以采用第三方的客户端的完美实现，比如Curator框架，Appache顶级项目


###1.6 PAXOS算法

暂时没有彻底想通，以下资料可能有用。

[维基百科-PAXOS算法](https://zh.wikipedia.org/zh-cn/Paxos%E7%AE%97%E6%B3%95)
[维基百科-拜占庭将军问题](https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98)
[csdn-zookeeper和paxos](http://blog.csdn.net/xhh198781/article/details/10949697)
[github-paxdemo-java-python版本](https://github.com/cocagne/paxos)
[source-libpaxos3](https://sourceforge.net/projects/libpaxos/files/LibPaxos3/)



### 1.7 ZAB协议

[文档](http://www.cnblogs.com/sunddenly/articles/4073157.html)

Zookeeper使用了一种称为Zab（Zookeeper Atomic Broadcast）的协议作为其一致性复制的核心，据其作者说这是一种新发算法，其特点是充分考虑了Yahoo的具体情况：**高吞吐量、低延迟、健壮、简单，但不过分要求其扩展性**。
　　（1）Zookeeper的实现**是由Client、Server构成**
　　　　① Server端提供了一个一致性复制、存储服务；
　　　　② Client端会提供一些具体的语义，比如分布式锁、选举算法、分布式互斥等；
　　　　
　　（2）从存储内容来说，Server端更多的是存储一些数据的状态，而非数据内容本身，因此Zookeeper可以作为一个小文件系统使用。数据状态的存储量相对不大，完全可以全部加载到内存中，从而极大地消除了通信延迟。
　　
　　（3）Server可以Crash后重启，考虑到容错性，Server必须“记住”之前的数据状态，因此数据需要持久化，但吞吐量很高时，磁盘的IO便成为系统瓶颈，其解决办法是使用缓存，把随机写变为连续写。☆☆☆
　　
　　（4）安全属性
　　考虑到Zookeeper主要操作数据的状态，为了保证状态的一致性，Zookeeper提出了两个安全属性（Safety Property）
　　　　① 全序（Total order）：**如果消息a在消息b之前发送，则所有Server应该看到相同的结果**
　　　　② 因果顺序（Causal order）：**如果消息a在消息b之前发生（a导致了b），并被一起发送，则a始终在b之前被执行。☆**☆
　　（5）安全保证
　　为了保证上述两个安全属性，Zookeeper使用了TCP协议和Leader。
　　　　① 通过使用TCP协议保证了消息的全序特性（先发先到）
　　　　② 通过Leader解决了因果顺序问题：先到Leader的先执行。
　　因为有了Leader，Zookeeper的架构就变为：Master-Slave模式，但在该模式中Master（Leader）会Crash，因此，Zookeeper引入了Leader选举算法，以保证系统的健壮性。
　　（6）Zookeeper整个工作分两个阶段：
　　　　① Atomic Broadcast
　　　　② Leader选举
　　（7）Zab特性

　　ZooKeeper中提交事务的协议并不是Paxos，而是由二阶段提交协议改编的ZAB协议。Zab可以满足以下特性：

　　　　①可靠提交 Reliable delivery：如果消息m被一个server递交了，那么m也将最终被所有server递交。
　　　　②全局有序 Total order：如果server在递交b之前递交了a，那么所有递交了a、b的server也会在递交b之前递交a。
　　　　③因果有序 Casual order：对于两个递交了的消息a、b，如果a因果关系优先于(causally precedes)b，那么a将在b之前递交。　

　　第三条的因果优先指的是同一个发送者发送的两个消息a先于b发送，或者上一个leader发送的消息a先于当前leader发送的消息。

**一、Atomic Broadcast**

同一时刻存在一个Leader节点，其他节点称为“Follower”:

	1.如果是写请求，如果客户端连接到Leader节点，则由Leader节点执行其请求；如果连接到Follower节点，则需转发请求到Leader节点执行。
	2.但对读请求，Client可以直接从Follower上读取数据，如果需要读到最新数据，则需要从Leader节点进行
	3.Zookeeper设计的读写比例是`2：1`。

Leader通过一个简化版的二段提交模式向其他Follower发送请求，但与二段提交有两个明显的不同之处：
	1. 因为只有一个Leader，Leader提交到Follower的请求一定会被接受（没有其他Leader干扰）
	2. 不需要所有的Follower都响应成功，只要一个多数派即可

>通俗地说，如果有2f+1个节点，允许f个节点失败。因为任何两个多数派必有一个交集，当Leader切换时，通过这些交集节点可以获得当前系统的最新状态。如果没有一个多数派存在（存活节点数小于f+1）则，算法过程结束。但有一个特例：
	如果有A、B、C三个节点，A是Leader，如果B Crash，则A、C能正常工作，因为A是Leader，A、C还构成多数派；如果A Crash则无法继续工作，因为Leader选举的多数派无法构成。


**二、Leader Election**

Leader选举主要是依赖**Paxos**算法，具体算法过程请参考其他博文，这里仅考虑Leader选举带来的一些问题。Leader选举遇到的最大问题是，”新老交互“的问题，新Leader是否要继续老Leader的状态。这里要按老Leader Crash的时机点分几种情况：

老Leader在COMMIT前Crash（已经提交到本地）
老Leader在COMMIT后Crash，但有部分Follower接收到了Commit请求

	1. 第一种情况，这些数据只有老Leader自己知道，当老Leader重启后，需要与新Leader同步并把这些数据从本地删除，以维持状态一致。
	
	2. 第二种情况，新Leader应该能通过一个多数派获得老Leader提交的最新数据
	老Leader重启后，可能还会认为自己是Leader，可能会继续发送未完成的请求，从而因为两个Leader同时存在导致算法过程失败，解决办法是把Leader信息加入每条消息的id中，Zookeeper中称为zxid，zxid为一64位数字，高32位为leader信息又称为epoch，每次leader转换时递增；低32位为消息编号，Leader转换时应该从0重新开始编号。通过zxid，Follower能很容易发现请求是否来自老Leader，从而拒绝老Leader的请求。

因为在老Leader中存在着数据删除（情况1），因此Zookeeper的数据存储要支持补偿操作，这也就需要像数据库一样记录log。


**三、 Zab与Paxos**

Zab的作者认为Zab与paxos并不相同，只所以没有采用Paxos是因为Paxos保证不了全序顺序：
Because multiple leaders can
propose a value for a given instance two problems arise.
First, proposals can conflict. Paxos uses ballots to detect and resolve conflicting proposals. 
Second, it is not enough to know that a given instance number has been committed, processes must also be able to figure out which value has been committed.

Paxos算法的确是不关系请求之间的逻辑顺序，而只考虑数据之间的全序，但很少有人直接使用paxos算法，都会经过一定的简化、优化。
一般Paxos都会有几种简化形式，其中之一便是，在存在Leader的情况下，可以简化为1个阶段（Phase2）。仅有一个阶段的场景需要有一个健壮的Leader，因此工作重点就变为Leader选举，再考虑到Learner的过程，还需要一个”学习“的阶段，通过这种方式，Paxos可简化为两个阶段：
之前的Phase2
Learn
如果再考虑多数派要Learn成功，这其实就是Zab协议。Paxos算法着重是强调了选举过程的控制，对决议学习考虑的不多，Zab恰好对此进行了补充。
之前有人说，所有分布式算法都是Paxos的简化形式，虽然很绝对，但对很多情况的确如此，但不知Zab的作者是否认同这种说法？




## 二、zookeeper的安装

**一、安装**

 1.  下载zookeeper tar 文件 并且解压到zookeeper文件夹
 2. 配置环境变量
 3. 修改配置文件
	 `cd zookeeper/conf/`
	 `mv zoo_sample.cfg zoo.cfg`
4.  修改conf：
	`vi zoo.cfg` 修改2处:
	`dataDir=/usr/local/software/zookeeper/data`
	`#文件最后添加`
	`
	server.0=192.168.0.201:2888:3888
	server.1=192.168.0.202:2888:3888
	server.2=192.168.0.203:2888:3888
	`
5. 到上述指定data路径下新建data文件夹和myid文件
	`vi myid` 写入内容为服务器标识，分别为0，1，2

6. 重新`source`环境变量


**关于环境变量：**
```
export JAVA_HOME=/usr/local/software/jdk
export ZOOKEEPER_HOME=/usr/local/software/zookeeper
# 用冒号作为连接符
export PATH=.:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$PATH

```
**二、启动**
```
zkServer.sh start
zkServer.sh status
```

 操作客户端:
```
# 进入客户端
zkCli.sh
# 1. 查找
ls /
ls /zookeeper
# 2. 创建并且赋值
create /carl hadoop
# 3. 获取
get /carl
# 4. 设值
set /carl /newhadoop
# 5. 递归删除节点
rmr /path
# 6. 删除某个指定节点
delete /paht/child

# 创建节点有2种类型:短暂（ephemeral）和持久化(persistent)
```


##  三、使用原声api操作zookeeper

### 3.1 基础操作

提供了2套创建节点的方式，同步和异步创建节点

**（1）同步方式:**
	**参数1**, 节点路径，/nodeName （不允许递归创建节点，也就是说父节点不存在的情况下，不允许创建子节点）
	**参数2**，节点内容，也就是字节数组（不支持序列化方式，如果需要实现序列化，可以使用java相关序列化框架，如Hessian，Kryo框架）
	**参数3**，节点权限：使用Ids.OPEN_ACL_UNSAFE开放权限即可
	**参数4**，节点类型：
		`PERSISTENT`：持久节点
		`PERSISTENT_SEQUENTIAL`：持久顺序节点
		`EPHEMERAL`：临时节点
		`EPHEMERAL_SEQUENTIAL`：临时顺序节点

**（2）异步方式**，注册一个异步回调函数，要实现AsynCallBack.StringCallBack接口，重写
`processResult(int rc, String path, Object ctx, String name)`
1. rc 服务端响应码，0表示成功，-4表示端口连接，-110表示指定节点存在，-112表示会话过期
2. path：接口调用时传入API的数据节点的路径参数
3. ctx：调用接口传入API的ctx的值
4. name：实际在服务端创建节点的名称

基础demo源码:
```java
package bjsxt.zookeeper.base;

import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.*;
import org.apache.zookeeper.Watcher.Event.EventType;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooDefs.Ids;

/**
 * Zookeeper base学习笔记
 *
 * @since 2015-6-13
 */
public class ZookeeperBase {

    /**
     * zookeeper地址
     */
    static final String CONNECT_ADDR = "192.168.0.201:2181,192.168.0.202:2181,192.168.0.203:2181";
    /**
     * session超时时间
     */
    static final int SESSION_OUTTIME = 2000;//ms
    /**
     * 信号量，阻塞程序执行，用于等待zookeeper连接成功，发送成功信号
     */
    static final CountDownLatch connectedSemaphore = new CountDownLatch(1);

    public static void main(String[] args) throws Exception {

        ZooKeeper zk = new ZooKeeper(CONNECT_ADDR, SESSION_OUTTIME, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                //获取事件的状态
                KeeperState keeperState = event.getState();
                EventType eventType = event.getType();
                //如果是建立连接
                if (KeeperState.SyncConnected == keeperState) {
                    if (EventType.None == eventType) {
                        //如果建立连接成功，则发送信号量，让后续阻塞程序向下执行
                        connectedSemaphore.countDown();
                        System.out.println("zk 建立连接");
                    }
                }
            }
        });

        //进行阻塞
        connectedSemaphore.await();

        System.out.println("..");
        //创建父节点
//		zk.create("/testRoot", "testRoot".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

        //创建子节点
//		zk.create("/testRoot/children", "children data".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
       /* zk.create("/testRoot/children", "children data".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        byte[] result = zk.getData("/testRoot/children", false, null);
        System.out.println(new String(result));
        Thread.sleep(10000);*/
        //获取节点洗信息
//		byte[] data = zk.getData("/testRoot", false, null);
//		System.out.println(new String(data));
//		System.out.println(zk.getChildren("/testRoot", false));

        //修改节点的值
//		zk.setData("/testRoot", "modify data root".getBytes(), -1);
//		byte[] data = zk.getData("/testRoot", false, null);
//		System.out.println(new String(data));

        //判断节点是否存在
//		System.out.println(zk.exists("/testRoot/children", false));
        //删除节点
//		zk.delete("/testRoot/children", -1);
//		System.out.println(zk.exists("/testRoot/children", false));

        //异步删除
        zk.delete("/testRoot", -1, new AsyncCallback.VoidCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx) {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(rc);
                System.out.println(path);
                System.out.println(ctx);
            }
        }, "a");

        zk.close();
        Thread.sleep(3000L);

    }

}

```


### 3.2 Watch 机制

**概念:**

	zookeeper有watch事件，是一次性触发的，当watch监视的数据发生变化时，会通知设置了该watch的client，即watchr。同样，其watcher是监听数据发生了某些变化，那就一定会有对应的事件类型和状态类型。

**事件类型：**

	是跟znode节点相关的：
	1. EventType.NodeCreated
	2. EventType.NodeDataChanged
	3. EventType.NodeChildrenChanged
	4. EventType.NodeDeleted

**状态类型：**

	是跟客户端实例相关的：
	1. KeeperState.Disconnected
	2. KeeperState.SyncConnected
	3. KeeperState.AuthFailed
	4. KeeperState.Expired

```java
package bjsxt.zookeeper.watcher;

import org.apache.zookeeper.*;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Created by carl on 2016/3/30.
 */
public class ZookeeperWatcherDemo {

    /**
     * zookeeper地址
     */
    static final String CONNECT_ADDR = "192.168.0.201:2181,192.168.0.202:2181,192.168.0.203:2181";
    /**
     * session超时时间
     */
    static final int SESSION_OUTTIME = 2000;//ms
    /**
     * 信号量，阻塞程序执行，用于等待zookeeper连接成功，发送成功信号
     */
    static final CountDownLatch connectedSemaphore = new CountDownLatch(1);

    private static final String PARENT_PATH = "/p";

    private static final String CHILDREN_PATH = "/p/c";

    private static AtomicInteger seq = new AtomicInteger();

    public static void main(String[] args) throws Exception {
        //在创建的时候添加watcher
        //1 .监控到创建连接事件
        ZooKeeper zk = new ZooKeeper(CONNECT_ADDR, SESSION_OUTTIME, new MyWatcher());
        connectedSemaphore.await();
        _cleanAllPath(zk);
        _createPathWithOutWatcher(zk);
        System.out.println("get data:" + new String(zk.getData(PARENT_PATH, false, null)));
        _cleanAllPath(zk);
        //2. 监控到创建路径事件
        _createPathWithWatcher(zk);
        //3. 监控删除节点事件
        _deletePathWithWatcher(zk);
        //4. 监控增加子节点事件
        _createPathWithOutWatcher(zk);
        _createChildrenWithWatcher(zk);
        //5. 监控修改子节点数据事件，貌似监控不到
        _updateChildDataWithWatcher(zk);
        //6. 监控自身数据的修改
        _updateWithWatcher(zk);
        Thread.sleep(2000L);
        _close(zk);
    }

    private static boolean _updateWithWatcher(ZooKeeper zk) {
        try {
            zk.exists(PARENT_PATH, true);
            zk.setData(PARENT_PATH, (System.currentTimeMillis() + "").getBytes(), -1);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }

    private static boolean _updateChildDataWithWatcher(ZooKeeper zk) {
        try {
            zk.getChildren(PARENT_PATH, true);
            zk.setData(CHILDREN_PATH, (System.currentTimeMillis() + "").getBytes(), -1);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }

    private static boolean _createChildrenWithWatcher(ZooKeeper zk) {
        try {
            zk.getChildren(PARENT_PATH, true);
            zk.create(CHILDREN_PATH, "init child data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }

    private static boolean _createPathWithOutWatcher(ZooKeeper zk) {
        //1. 添加watch事件
        try {
            zk.exists(PARENT_PATH, false);
            zk.create(PARENT_PATH, "init data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }

    private static boolean _deletePathWithWatcher(ZooKeeper zk) {
        try {
            zk.exists(PARENT_PATH, true);
            zk.delete(PARENT_PATH, -1);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }

    private static boolean _createPathWithWatcher(ZooKeeper zk) {
        //1. 添加watch事件
        try {
            zk.exists(PARENT_PATH, true);
            zk.create(PARENT_PATH, "init data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
            return false;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return false;
        }
    }


    static class MyWatcher implements Watcher {
        @Override
        public void process(WatchedEvent event) {
            System.out.println("start process");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if (event == null) {
                return;
            }
            Event.KeeperState keeperState = event.getState();
            Event.EventType eventType = event.getType();
            String logPrefix = "【Watcher-" + seq.incrementAndGet() + "】";
            System.out.println(logPrefix + "收到Watcher通知");
            System.out.println(logPrefix + "连接状态:\t" + keeperState.toString());
            System.out.println(logPrefix + "事件类型:\t" + eventType.toString());
            //1. 如果是连接状态
            if (Event.KeeperState.SyncConnected == keeperState) {
                if (Event.EventType.None == eventType) {
                    //1.1 第一次连接zookeeper
                    System.out.println(logPrefix + "第一次连接上服务器");
                    System.out.println("-----------------------------------");
                    connectedSemaphore.countDown();
                } else if (Event.EventType.NodeCreated == eventType) {
                    //1.2 如果监听到创建节点信息
                    String path = event.getPath();
                    System.out.println(logPrefix + "创建了节点:" + path);
                    System.out.println("-----------------------------------");
                } else if (Event.EventType.NodeDeleted == eventType) {
                    //1.3 如果监听到删除节点信息
                    String path = event.getPath();
                    System.out.println(logPrefix + "删除了节点:" + path);
                    System.out.println("-----------------------------------");
                } else if (Event.EventType.NodeChildrenChanged == eventType) {
                    //1.4 如果监听到修改子节点信息
                    String path = event.getPath();
                    System.out.println(logPrefix + "增加了子节点:" + path);
                    System.out.println("-----------------------------------");
                } else if(Event.EventType.NodeDataChanged == eventType){
                    //1.5 监控自身的数据修改
                    String path = event.getPath();
                    System.out.println(logPrefix + "自身的数据修改:" + path);
                    System.out.println("-----------------------------------");
                }
            }

        }
    }


    private static void _close(ZooKeeper zk) {
        if (zk != null) {
            try {
                zk.close();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }


    private static void _cleanAllPath(ZooKeeper zk) {
        try {
            try {
                zk.delete(CHILDREN_PATH, -1);
            } catch (Exception e) {

            } finally {
                zk.delete(PARENT_PATH, -1);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
}

```



### 3.3 安全机制
### 3.4 zkClient


## 四、Curator

### 4.1 基础API-增删改查和watcher

**简介：**

	为了更好的实现java操作zookeeper服务器，出现Curator框架里面提供了zookeeper的丰富操作，例如
	session超时重连，主从选举，分布式计数器，分布式锁等等适用于各种分布式场景的api操作。

```
 <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.10.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-client</artifactId>
            <version>2.10.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.10.0</version>
        </dependency>
```

**java 代码如下：**


```java
package com.carl.demo.zookeeper.base;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.framework.recipes.cache.*;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.utils.CloseableUtils;
import org.apache.zookeeper.CreateMode;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 2016/3/31.
 */
public class BaseDemo {
    static final String CONNECT_ADDR = "192.168.0.201:2181,192.168.0.202:2181,192.168.0.203:2181";

    /**
     * session超时时间:指的是连接之后如果5000s不使用session，自动断开并且关闭连接
     */
    static final int SESSION_OUTTIME = 5000;//ms

    public static void main(String[] args) throws Exception {
        //1 重试策略：初试时间为1s 重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 2);
        //2 通过工厂创建连接
        CuratorFramework cf = CuratorFrameworkFactory.builder()
                .connectString(CONNECT_ADDR)
                .sessionTimeoutMs(SESSION_OUTTIME)
                .retryPolicy(retryPolicy)
//					.namespace("super")
                .build();
        cf.start();
        _delete(cf);
//        _create(cf, CHILD);
//        _createWithCallBack(cf);
//        _testWatch(cf);
        _testWatch2(cf);
        CloseableUtils.closeQuietly(cf);
    }

    private static final String SUPER = "/super";
    private static final String CHILD = "/super/c1";

    public static void _createWithCallBack(CuratorFramework cf) throws Exception {
        ExecutorService pool = Executors.newCachedThreadPool();
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                .inBackground(new BackgroundCallback() {
                    public void processResult(CuratorFramework cf, CuratorEvent ce) throws Exception {
                        System.out.println("code:" + ce.getResultCode());
                        System.out.println("type:" + ce.getType());
                        System.out.println("线程为:" + Thread.currentThread().getName());
                    }
                }, pool)
                .forPath("/super/c3", "c3内容".getBytes());
        Thread.sleep(2000L);
        pool.shutdown();
    }

    public static void _create(CuratorFramework cf, String path) throws Exception {
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath(path, "init data".getBytes());
    }

    public static void _delete(CuratorFramework cf) {
        try {
            cf.delete().forPath(SUPER);
        } catch (Exception e) {
        }
    }

    /**
     * <pre>
     *     监听节点的增加和修改
     * </pre>
     * @param cf
     */
    private static void _testWatch(CuratorFramework cf) throws Exception {
        final NodeCache cache = new NodeCache(cf, "/super", false);
        cache.start(true);
        cache.getListenable().addListener(new NodeCacheListener() {
            /**
             * <B>方法名称：</B>nodeChanged<BR>
             * <B>概要说明：</B>触发事件为创建节点和更新节点，在删除节点的时候并不触发此操作。<BR>
             * @see org.apache.curator.framework.recipes.cache.NodeCacheListener#nodeChanged()
             */
            public void nodeChanged() throws Exception {
                System.out.println("路径为：" + cache.getCurrentData().getPath());
                System.out.println("数据为：" + new String(cache.getCurrentData().getData()));
                System.out.println("状态为：" + cache.getCurrentData().getStat());
                System.out.println("---------------------------------------");
            }
        });

        Thread.sleep(1000);
        cf.create().forPath("/super", "123".getBytes());

        Thread.sleep(1000);
        cf.setData().forPath("/super", "456".getBytes());
        //无法监控delete事件
        Thread.sleep(1000);
        cf.delete().forPath("/super");
    }

    public static void _testWatch2(CuratorFramework cf) throws Exception {
        PathChildrenCache cache = new PathChildrenCache(cf, "/super", true);
        //5 在初始化的时候就进行缓存监听
        cache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
        cache.getListenable().addListener(new PathChildrenCacheListener() {
            /**
             * <B>方法名称：</B>监听子节点变更<BR>
             * <B>概要说明：</B>新建、修改、删除<BR>
             * @see org.apache.curator.framework.recipes.cache.PathChildrenCacheListener#childEvent(org.apache.curator.framework.CuratorFramework, org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent)
             */
            public void childEvent(CuratorFramework cf, PathChildrenCacheEvent event) throws Exception {
                switch (event.getType()) {
                    case CHILD_ADDED:
                        System.out.println("CHILD_ADDED :" + event.getData().getPath());
                        break;
                    case CHILD_UPDATED:
                        System.out.println("CHILD_UPDATED :" + event.getData().getPath());
                        break;
                    case CHILD_REMOVED:
                        System.out.println("CHILD_REMOVED :" + event.getData().getPath());
                        break;
                    default:
                        break;
                }
            }
        });

        //创建本身节点不发生变化
        cf.create().forPath("/super", "init".getBytes());

        //添加子节点
        Thread.sleep(1000);
        cf.create().forPath("/super/c1", "c1内容".getBytes());
        Thread.sleep(1000);
        cf.create().forPath("/super/c2", "c2内容".getBytes());

        //修改子节点
        Thread.sleep(1000);
        cf.setData().forPath("/super/c1", "c1更新内容".getBytes());

        //删除子节点
        Thread.sleep(1000);
        cf.delete().forPath("/super/c2");

        //删除本身节点
        Thread.sleep(1000);
        cf.delete().deletingChildrenIfNeeded().forPath("/super");

    }
}

```

**（1）链式风格**
```java
 RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 2);
  CuratorFramework cf = CuratorFrameworkFactory.builder()
                .connectString(CONNECT_ADDR)
                .sessionTimeoutMs(SESSION_OUTTIME)
                .retryPolicy(retryPolicy)
//					.namespace("super")
                .build();
```

	链式风格代码，注意`sessionTimeoutMs` 是指session超时时间而不是连接超时时间
	重试策略有很多种,上述只是初始时间为1s 重试10次



**（2）支持回调函数**

```java
public static void _createWithCallBack(CuratorFramework cf) throws Exception {
        ExecutorService pool = Executors.newCachedThreadPool();
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                .inBackground(new BackgroundCallback() {
                    public void processResult(CuratorFramework cf, CuratorEvent ce) throws Exception {
                        System.out.println("code:" + ce.getResultCode());
                        System.out.println("type:" + ce.getType());
                        System.out.println("线程为:" + Thread.currentThread().getName());
                    }
                }, pool)
                .forPath("/super/c3", "c3内容".getBytes());
        Thread.sleep(2000L);
        pool.shutdown();
    }

```

**（3）支持watcher**

	第一种watcher支持监听当前节点的增加，和修改，注意，不能监听删除事件
```java
 /**
     * <pre>
     *     监听节点的增加和修改
     * </pre>
     * @param cf
     */
    private static void _testWatch(CuratorFramework cf) throws Exception {
        final NodeCache cache = new NodeCache(cf, "/super", false);
        cache.start(true);
        cache.getListenable().addListener(new NodeCacheListener() {
            /**
             * <B>方法名称：</B>nodeChanged<BR>
             * <B>概要说明：</B>触发事件为创建节点和更新节点，在删除节点的时候并不触发此操作。<BR>
             * @see org.apache.curator.framework.recipes.cache.NodeCacheListener#nodeChanged()
             */
            public void nodeChanged() throws Exception {
                System.out.println("路径为：" + cache.getCurrentData().getPath());
                System.out.println("数据为：" + new String(cache.getCurrentData().getData()));
                System.out.println("状态为：" + cache.getCurrentData().getStat());
                System.out.println("---------------------------------------");
            }
        });

        Thread.sleep(1000);
        cf.create().forPath("/super", "123".getBytes());

        Thread.sleep(1000);
        cf.setData().forPath("/super", "456".getBytes());
        //无法监控delete事件
        Thread.sleep(1000);
        cf.delete().forPath("/super");
    }
```


	第二种也是最常用的监听器，支持监听子节点的增加，删除，修改事件

```java
 public static void _testWatch2(CuratorFramework cf) throws Exception {
        PathChildrenCache cache = new PathChildrenCache(cf, "/super", true);
        //5 在初始化的时候就进行缓存监听
        cache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
        cache.getListenable().addListener(new PathChildrenCacheListener() {
            /**
             * <B>方法名称：</B>监听子节点变更<BR>
             * <B>概要说明：</B>新建、修改、删除<BR>
             * @see org.apache.curator.framework.recipes.cache.PathChildrenCacheListener#childEvent(org.apache.curator.framework.CuratorFramework, org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent)
             */
            public void childEvent(CuratorFramework cf, PathChildrenCacheEvent event) throws Exception {
                switch (event.getType()) {
                    case CHILD_ADDED:
                        System.out.println("CHILD_ADDED :" + event.getData().getPath());
                        break;
                    case CHILD_UPDATED:
                        System.out.println("CHILD_UPDATED :" + event.getData().getPath());
                        break;
                    case CHILD_REMOVED:
                        System.out.println("CHILD_REMOVED :" + event.getData().getPath());
                        break;
                    default:
                        break;
                }
            }
        });

        //创建本身节点不发生变化
        cf.create().forPath("/super", "init".getBytes());

        //添加子节点
        Thread.sleep(1000);
        cf.create().forPath("/super/c1", "c1内容".getBytes());
        Thread.sleep(1000);
        cf.create().forPath("/super/c2", "c2内容".getBytes());

        //修改子节点
        Thread.sleep(1000);
        cf.setData().forPath("/super/c1", "c1更新内容".getBytes());

        //删除子节点
        Thread.sleep(1000);
        cf.delete().forPath("/super/c2");

        //删除本身节点
        Thread.sleep(1000);
        cf.delete().deletingChildrenIfNeeded().forPath("/super");

    }
```


### 4.2 分布式Barrier

	这种barrier没有数量限制，直接堵塞，必须由其他线程来唤醒。

**Distributed Barrier**

```java
package com.carl.demo.zookeeper;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.barriers.DistributedBarrier;

/**
 * Created by carl on 16-4-1.
 */
public class DistributeBarrierExample {

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            //开启5个线程等待barrier
            new Thread() {
                @Override
                public void run() {
                    CuratorFramework cf = ZkUtils.getConn();
                    cf.start();
                    DistributedBarrier barrier = new DistributedBarrier(cf, "/super");
                    try {
                        System.out.println(Thread.currentThread().getName() + " waiting for barrier");
                        barrier.setBarrier();
                        barrier.waitOnBarrier();
                        System.out.println(Thread.currentThread().getName() + " finish");
                        cf.close();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }.start();
        }


        new Thread() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                new Thread() {
                    @Override
                    public void run() {
                        CuratorFramework cf = ZkUtils.getConn();
                        cf.start();
                        DistributedBarrier barrier = new DistributedBarrier(cf, "/super");
                        try {
                            System.out.println("go to stop barrier");
                            barrier.setBarrier();
                            barrier.removeBarrier();
                            cf.close();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }.start();
            }
        }.start();

    }
}

```


**Distributed double barrier**

	一同开始一同结束

```
package com.carl.demo.zookeeper;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.barriers.DistributedDoubleBarrier;
import org.apache.zookeeper.ZKUtil;

/**
 * Created by carl on 16-4-1.
 */
public class DoubleBarrierExample {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread() {
                @Override
                public void run() {
                    String name = Thread.currentThread().getName();
                    CuratorFramework cf = ZkUtils.getConn();
                    cf.start();
                    DistributedDoubleBarrier barrier = new DistributedDoubleBarrier(cf, "/super", 5);
                    System.out.println(name + " is start..");
                    try {
                        barrier.enter();
                        System.out.println(name + " is doing..");
                        barrier.leave();
                        System.out.println(name + " finish..");
                    } catch (Exception e) {
                        e.printStackTrace();
                    }finally {
                        cf.close();
                    }

                }
            }.start();
        }
    }
}

```


### 4.3 扩展功能：namespace,stateListener,transaction

**（1）Listener:**

	后台操作的通知和监控可以通过ClientListener接口发布. 你可以在CuratorFramework实例上通过addListener()注册listener, Listener实现了下面的方法:
	eventReceived() 一个后台操作完成或者一个监控被触发
	
	
```
package com.carl.demo.zookeeper.basic;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.state.ConnectionState;
import org.apache.curator.framework.state.ConnectionStateListener;
import org.apache.zookeeper.ZKUtil;

/**
 * Created by carl on 16-4-1.
 */
public class ListenerDemo {
    public static void main(String[] args) throws Exception {
        CuratorFramework cf = ZkUtils.getConn();
        cf.getConnectionStateListenable().addListener(new ConnectionStateListener() {
            @Override
            public void stateChanged(CuratorFramework curatorFramework, ConnectionState connectionState) {
                System.out.println("state:" + connectionState.name());
            }
        });

        cf.start();
        ZkUtils.sleep(1000*60);
        cf.close();
    }
}

```

**（2）namespace**


你可以使用命名空间Namespace避免多个应用的节点的名称冲突。 CuratorFramework提供了命名空间的概念，这样CuratorFramework会为它的API调用的path加上命名空间：

```
CuratorFramework    client = CuratorFrameworkFactory.builder().namespace("MyApp") ... build();
 ...
client.create().forPath("/test", data);
// node was actually written to: "/MyApp/test"
```


**(3) 事务**

	 CuratorFramework提供了事务的概念，可以将一组操作放在一个原子事务中。 什么叫事务？ 事务是原子的， 一组操作要么都成功，要么都失败。

```java
package com.carl.demo.zookeeper.basic;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.api.transaction.CuratorTransaction;
import org.apache.curator.framework.api.transaction.CuratorTransactionFinal;
import org.apache.curator.framework.api.transaction.CuratorTransactionResult;

import java.util.Collection;

/**
 * Created by carl on 16-4-1.
 */
public class TransactionExample {
    public static void main(String[] args) throws Exception {
        CuratorFramework cf = ZkUtils.getConn();
        cf.start();
        //一组原子性操作
        cf.inTransaction().create().forPath("/s/c1").and().create().forPath("/s/c2").and().commit();
        cf.close();
    }

    public static Collection<CuratorTransactionResult> transaction(CuratorFramework client) throws Exception {
        // this example shows how to use ZooKeeper's new transactions
        Collection<CuratorTransactionResult> results = client.inTransaction().create().forPath("/a/path", "some data".getBytes())
                .and().setData().forPath("/another/path", "other data".getBytes())
                .and().delete().forPath("/yet/another/path")
                .and().commit(); // IMPORTANT!
        // called
        for (CuratorTransactionResult result : results) {
            System.out.println(result.getForPath() + " - " + result.getType());
        }
        return results;
    }

    /*
     * These next four methods show how to use Curator's transaction APIs in a
     * more traditional - one-at-a-time - manner
     */
    public static CuratorTransaction startTransaction(CuratorFramework client) {
        // start the transaction builder
        return client.inTransaction();
    }

    public static CuratorTransactionFinal addCreateToTransaction(CuratorTransaction transaction) throws Exception {
        // add a create operation
        return transaction.create().forPath("/a/path", "some data".getBytes()).and();
    }

    public static CuratorTransactionFinal addDeleteToTransaction(CuratorTransaction transaction) throws Exception {
        // add a delete operation
        return transaction.delete().forPath("/another/path").and();
    }

    public static void commitTransaction(CuratorTransactionFinal transaction) throws Exception {
        // commit the transaction
        transaction.commit();
    }
}

```


### 4.4 缓存

	可以利用ZooKeeper在集群的各个节点之间缓存数据。 每个节点都可以得到最新的缓存的数据。 Curator提供了三种类型的缓存方式：Path Cache,Node Cache 和Tree Cache。


**Path Cache:**

	Path Cache用来监控一个ZNode的子节点. 当一个子节点增加， 更新，删除时， Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态。 这也正如它的名字表示的那样， 那监控path。

实际使用时会涉及到**四个类**：

`PathChildrenCache`
`PathChildrenCacheEvent`
`PathChildrenCacheListener`
`ChildData`

（1）通过下面的构造函数创建Path Cache:

`public PathChildrenCache(CuratorFramework client, String path, boolean cacheData)`
想使用cache，必须调用它的start方法，不用之后调用close方法。 start有两个， 其中一个可以传入StartMode，用来为初始的cache设置暖场方式(warm)：

`NORMAL`: 初始时为空。
`BUILD_INITIAL_CACHE`: 在这个方法返回之前调用rebuild()。
`POST_INITIALIZED_EVENT`: 当Cache初始化数据后发送一个`PathChildrenCacheEvent.Type#INITIALIZED`事件

（2）public void addListener(PathChildrenCacheListener listener)可以增加listener监听缓存的改变。

（3）getCurrentData()方法返回一个List<ChildData>对象，可以遍历所有的子节点。

这个例子摘自官方的例子， 实现了一个控制台的方式操作缓存。 它提供了三个命令， 你可以在控制台中输入。

`set ` 用来新增或者更新一个子节点的值， 也就是更新一个缓存对象
`remove`  是删除一个缓存对象
`list ` 列出所有的缓存对象
另外还提供了一个help命令提供帮助。

设置/更新、移除其实是使用client (CuratorFramework)来操作, 不通过PathChildrenCache操作：

`client.setData().forPath(path, bytes);
client.create().creatingParentsIfNeeded().forPath(path, bytes);
client.delete().forPath(path);`
而查询缓存使用下面的方法：
```java
for (ChildData data : cache.getCurrentData()) {
    System.out.println(data.getPath() + " = " + new String(data.getData()));
}
```

**Node Cache:**

	类似，当前节点的删除和新增

**Tree Node Cache:**
	
	类似，以上2种的结合体


### 4.5 分布式锁

分布式的锁全局同步， 这意味着任何一个时间点不会有两个客户端都拥有相同的锁。

**一、可重入锁Shared Reentrant Lock**

	首先我们先看一个全局可重入的锁。 Shared意味着锁是全局可见的， 客户端都可以请求锁。 Reentrant和JDK的ReentrantLock类似， 意味着同一个客户端在拥有锁的同时，可以多次获取，不会被阻塞。 它是由类InterProcessMutex来实现。 它的构造函数为：

`public InterProcessMutex(CuratorFramework client, String path)`


1. 通过`acquire`**获得锁**，并提供超时机制：
	```java
	public void acquire()
	Acquire the mutex - blocking until it's available. Note: the same thread can call acquire
	re-entrantly. Each call to acquire must be balanced by a call to release()
	
	public boolean acquire(long time,
	                       TimeUnit unit)
	Acquire the mutex - blocks until it's available or the given time expires. Note: the same thread can
	call acquire re-entrantly. Each call to acquire that returns true must be balanced by a call to release()
	
	Parameters:
	time - time to wait
	unit - time unit
	Returns:
	true if the mutex was acquired, false if not
	```
2. 通过`release()`方法**释放锁**。 InterProcessMutex 实例可以重用。

3. Revoking ZooKeeper recipes wiki定义了可协商的撤销机制。 为了撤销mutex, 调用下面的方法：
	```
	public void makeRevocable(RevocationListener<T> listener)
	将锁设为可撤销的. 当别的进程或线程想让你释放锁是Listener会被调用。
	Parameters:
	listener - the listener
	```
4. 如果你请求撤销当前的锁， 调用`Revoker`方法。
	```
	public static void attemptRevoke(CuratorFramework client,
	                                 String path)
	                         throws Exception
	Utility to mark a lock for revocation. Assuming that the lock has been registered
	with a RevocationListener, it will get called and the lock should be released. Note,
	however, that revocation is cooperative.
	Parameters:
	client - the client
	path - the path of the lock - usually from something like InterProcessMutex.getParticipantNodes()
	```
	
5. **错误处理** 还是强烈推荐你使用ConnectionStateListener处理连接状态的改变。 当连接LOST时你不再拥有锁。


创建共享资源


```java
package com.colobu.zkrecipe.lock;

import java.util.concurrent.atomic.AtomicBoolean;

public class FakeLimitedResource {
    private final AtomicBoolean inUse = new AtomicBoolean(false);

    public void use() throws InterruptedException {
        // 真实环境中我们会在这里访问/维护一个共享的资源
        //这个例子在使用锁的情况下不会非法并发异常IllegalStateException
        //但是在无锁的情况由于sleep了一段时间，很容易抛出异常
        if (!inUse.compareAndSet(false, true)) { 
            throw new IllegalStateException("Needs to be used by one client at a time");
        }
        try {
            Thread.sleep((long) (3 * Math.random()));
        } finally {
        //如果此时finally没有执行会抛出异常
            inUse.set(false);
        }
    }
}
```


```
package com.carl.demo.zookeeper.tool;

import com.carl.demo.zookeeper.basic.ZkUtils;
import com.google.common.base.Throwables;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 16-4-1.
 */
public class ReentrantLockDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(5);
        final CountDownLatch latch = new CountDownLatch(5);
        //1. 创建共享资源
        final FakeLimitedResource resource = new FakeLimitedResource();

        //2. 创建线程访问共享资源
        for (int i = 0; i < 5; i++) {
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    //2.1 获取连接
                    CuratorFramework cf = ZkUtils.getConn();
                    //2.2 创建分布式锁
                    InterProcessMutex lock = new InterProcessMutex(cf, "/super");
                    cf.start();
                    try {
                        //2.3 重入锁两次
                        lock.acquire();
                        lock.acquire();
                        System.out.println(Thread.currentThread().getName());
                        resource.use();
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        try {
                            lock.release();
                            lock.release();
                        } catch (Exception e) {
                            Throwables.propagate(e);
                        } finally {
                            cf.close();
                            latch.countDown();
                        }
                    }
                }
            });
        }
        latch.await();
        System.out.println("every thing is finished");
        pool.shutdown();
    }
}

```


**二、不可重入锁: InterProcessSemaphoreMutex**
	
	代码类似重入锁，2.3 注释部分改为只入一次，没有重入，否则重入死锁


**三、可重入读写锁：InterProcessReadWriteLock**

```java
public InterProcessLock readLock()
public InterProcessLock writeLock()
```


**四、 不可重入读写锁：？**


### 4.6 分布式锁原理探究



### 4.7 分布式信号量

