### Ref Doc Url
1.  [图说设计模式](http://design-patterns.readthedocs.io/zh_CN/latest/index.html)

### Centos
1. install tomcat, jdk, nginx

```
[root@localhost temp]# tar -zxvf apache-tomcat-7.0.68.tar.gz -C ../tomcat
[root@localhost temp]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.zip
[root@localhost pcre-8.37]# yum install gcc gcc-c++ autoconf make
[root@localhost pcre-8.37]# ./configure & make & make install
[root@localhost temp]# ./configure --prefix=/usr/local/nginx --conf-path=/usr/local/nginx/nginx.conf
```


1. nginx startup cmd

```
[root@localhost ~]# /usr/local/nginx/sbin/nginx
[root@localhost ~]# netstat -ano |grep 80
```


1. Install tomcat as a service

Change to the /etc/init.d directory and create a script called 'tomcat' as shown below.
```
[root@srv6 share]# cd /etc/init.d
[root@srv6 init.d]# vi tomcat
```

And here is the script we will use.


```
#!/bin/bash
# description: Tomcat Start Stop Restart
# processname: tomcat
# chkconfig: 234 20 80
JAVA_HOME=/usr/java/jdk1.7.0_05
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
CATALINA_HOME=/usr/share/apache-tomcat-7.0.29

case $1 in
start)
sh $CATALINA_HOME/bin/startup.sh
;;
stop)
sh $CATALINA_HOME/bin/shutdown.sh
;;
restart)
sh $CATALINA_HOME/bin/shutdown.sh
sh $CATALINA_HOME/bin/startup.sh
;;
esac
exit 0
```


> [Tomcat install reference url](http://www.davidghedini.com/pg/entry/install_tomcat_7_on_centos)

1. Install Redis

```
vi /etc/redis/redis.conf
```

> 仅修改： daemonize yes （no-->yes）

```
echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local
```

> [CentOS6 安装 Redis](https://segmentfault.com/a/1190000002685224)

```
[root@srv6 init.d]# chmod 755 tomcat
[root@srv6 init.d]# chkconfig --add tomcat
[root@srv6 init.d]# chkconfig --level 234 tomcat on
```

> [reference url](http://www.davidghedini.com/pg/entry/install_tomcat_7_on_centos)

1. load balacing

```
http {
    upstream app {
        server localhost:8080;
    }
}
server {
    local / {
        proxy_pass http://app;
    }
}
```


1. nginx iptables [remove  ]
```
[root@nginx nginx]# iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
[root@nginx nginx]# /etc/init.d/iptables status
表格：filter
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:8080
2    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:80
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
4    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
5    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
6    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22
7    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination
1    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination

```
1. redis install

```
[root@nginx redis]# make && make install
[root@nginx redis]# redis-server /etc/redis/redis.conf
```

1. install mysql
查看 mysql：

```
yum list | grep mysql
```

安装：
> yum install mysql-server mysql mysql-devel

查看安装结果：
> [root@nginx ~]# rpm -qi mysql-server

首次启动：
> [root@nginx ~]# service mysqld start

Set a password:
> [root@nginx ~]# /usr/bin/mysqladmin -u root password 123456

设置开机自启动：
> [root@nginx ~]# chkconfig mysqld on

1. install varnish
使用epel源
> [root@localhost bin]# rpm -ivh http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm

check varnish:
> [root@localhost bin]# yum list|grep varnish

Startup:
> [root@localhost bin]# service varnish start

Check:
> [root@localhost bin]# varnishd -V

For more details:
[Varnish official User Guide](https://varnish-cache.org/docs/4.0/index.html#)

1. Install ZooKeeper
- 启动
> zookeeper-3.4.6/bin/zkServer.sh start

- 输入jps命令查看进程
1573 QuorumPeerMain
1654 Jps


1. Set time zone
> [root@demo81 Asia]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

> [root@demo81 Asia]# vim /etc/sysconfig/clock

Add line:
> UTC=false or no — The hardware clock is set to local time.

Reload:
> [root@demo81 Asia]# /etc/init.d/ntpd restart

OR:
> [root@dlp ~]# source /etc/sysconfig/clock

Check:
> [root@demo81 Asia]# date
  2016年 05月 26日 星期四 09:36:10 CST
