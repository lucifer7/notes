# tomcat常见问题

## 无法shutdown

**一、问题现象**

Tomcat关联java进程仍然存活
这种问题往往是由于应用开发的问题导致无法shutdown

**二、方案**

**强行kill进程**

修改2个文件，一个shutdown.sh
一个catalina.sh

```
==============================bin/shutdown.sh   
exec "$PRGDIR"/"$EXECUTABLE" stop  -force "$@"  #加上 -force    
   
==============================bin/catalina.sh    
在PRGDIR=`dirname "$PRG"`后面加上  
if [ -z "$CATALINA_PID" ]; then  
      CATALINA_PID=$PRGDIR/CATALINA_PID  
      cat $CATALINA_PID  
fi  
```


**修改应用**

但是，现在很多web应用都是由spring来管理bean的，实际情况下我们可能需要注入很多第三方jar中的bean。比如在applicationContext.xml中配置如下bean：

<bean id ="dataSource" class ="org.apache.commons.dbcp.BasicDataSource"destroy-method="close">

则在org.apache.commons.dbcp.BasicDataSource需要做好销毁数据库线程池的逻辑。

当然我们也可以自己设置销毁逻辑。在web.xml文件中增加监听器。
```

<listener>  
    <listener-class>com.mysite.MySpecialListener</listener-class>  
</listener>  
```


```
public class MySpecialListener extends ApplicationContextListener {  
    @Override  
    public void contextInitialized(ServletContextEvent sce) {  
        // On Application Startup, please…  
        // Usually I'll make a singleton in here, set up my pool, etc.  
    }  
    @Override  
    public void contextDestroyed(ServletContextEvent sce) {  
        // On Application Shutdown, please…  
        // 1. Go fetch that DataSource  
        Context initContext = new InitialContext();  
        Context envContext  = (Context)initContext.lookup("java:/comp/env");  
        DataSource datasource = (DataSource)envContext.lookup("jdbc/database");  
        // 2. Call the close() method on it  
        datasource.close();  
        // 3. Deregister Driver  
        try {  
            java.sql.Driver mySqlDriver = DriverManager.getDriver("jdbc:mysql://localhost:3306/");  
            DriverManager.deregisterDriver(mySqlDriver);  
        } catch (SQLException ex) {  
            logger.info("Could not deregister driver:".concat(ex.getMessage()));  
        }   
        dataSource = null;  
    }  
} 
```




# tomcat 优化


[TOC]

## 一、 服务器的性能优化思路-TOMCAT

		影响服务器的性能是多方面的，包括软件架构，部署环境，服务器硬件配置，软件设计，开发实现等方面


服务器优化的方向：

1. 尽量缩短单个请求的处理时间
2. 尽可能多的并发处理请求，让服务器的CPU处理忙碌状态
3. 必须做到横向扩展

**关于tomcat**

	现在tomcat默认优化的已经不错，留给我们可以优化的空间很小，我们主要能够调整的主要是：跟具体的使用场景相关设置hi，大致有：


1.**合理分配Tomcat需要的内存，主要涉及到JVM的东西**

`catalina.sh` 中的JAVA_OPS

(1) -server: 启用jdk的Server版
(2) -Xms: 虚拟机初始化时的最小内存
(3) -Xmx: 虚拟机可以使用的最大内存（建议到物理内存的80%）
(4) -XX:PermSize: 持久代初始值
(5) -XX:MaxPermSize: 持久代最大内存（默认是32M）
(6) -XX:MaxNewSize: 新生代的最大内存（默认是16M）


**说明：**

- **一般-Xms、Xmx相等**，默认空余堆内存小于40%会增大到最大限制；空余堆内存大于70%时，JVM会减少堆知道-Xms的最小限制。

- 查看配置是否生效:`jmap -heap tomcat的进程号`


**Tomcat本身的配置优化：**(基于tomcat8)

（1） maxConnections: 最大连接数，对BIO模式，默认等于maxThreads；对NIO默认是10000；对APR/native默认是8192，最主要是根据系统cpu等等。

（2）maxThreads：最大的线程数，即同时处理的任务个数，默认是200，200-1000，机器不错的话，1200，1500都行，具体问题具体测试

（3） acceptCount：当处理任务的线程数达到最大的时候，接受排队的请求个数，默认是100，一般可以设置到maxThreads的70%左右。

（4）minSpareThreads：最小的空闲线程数，默认是10，怎么都要大一点50，100最哦有

（5）compression：设置是否开启GZip压缩，浏览器也会自动解压缩

（6）compressableMimeType：哪些类型需要压缩，默认是text/html,text/xml,text/plain，怎么都得把javascript和css加上

（7）compressionMinSize：启用压缩的输出内容大小，默认是2048字节，就是2k，如果小文件太多，可以适当扩大，因为压缩也是需要消耗时间的

（8）是否反查域名，为了提高处理，应设置为false
（9）connectionTimeout：网络连接超时，单位毫秒，设置为-1表示永不超时，通常可以设置为2000ms，看情况而定

tomcat可以配置的参数依旧很多，可以结合官方文档继续研究，上述知识一些常用


**说明：**

1. 如果要加大最大并发连接数，应同时加大maxThreads和acceptCount，可以设置2个一样，依旧需要具体情况具体分析

2. WebServer允许的最大连接数还受制于操作系统的内核参数设置`ulimit -a` 例如open files 1024是相当的小

3. 如果配置了<Executor>执行器，在<Connector>中通过executor属性指定，那么关于线程的以这里的配置为准


**BIO/NIO/APR**

（1） **BIO**是最稳定最老的一个连接器，是采用阻塞的方式，意味着每个连接线程都绑定到每个Http请求，直到获得http响应返回，如果Http客户端请求的是keep-Alive连接，那么这些连接也许一直保持着直至达到timeout时间，这期间不能用于其他请求。

（2）**NIO** 使用Java的异步IO技术，不做阻塞，要使用的话，直接修改server.xml 修改protocol="org.apache.coyote.http11.HttpNioProtocol"

(3) **APR**是使用原生的C语言编写的非阻塞I/0，但是需要安装apr和native，直接启动就支持apr，能大幅度提升性能，使用时指定protocol="org.apache.coyote.http11.Http11AprProtocol".

可以知道到apache官方去下载:

apr-1.5.0 apr-icon-1.2.1 apr-util.jar等等
















