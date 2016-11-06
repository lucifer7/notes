
## Centos6.5
### Basic Install
#### 1. install tomcat, jdk, nginx

```
[root@localhost temp]# tar -zxvf apache-tomcat-7.0.68.tar.gz -C ../tomcat
[root@localhost temp]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.zip
[root@localhost pcre-8.37]# yum install gcc gcc-c++ autoconf make
[root@localhost pcre-8.37]# ./configure & make & make install
[root@localhost temp]# ./configure --prefix=/usr/local/nginx --conf-path=/usr/local/nginx/nginx.conf
```
- Simplier way to install nginx

First, install Extra Packages for Enterprise Linux
```
$ sudo yum install epel-release
```
Then, install nginx:
```
$ sudo yum install nginx
```
Then, start ngix:
```
$ sudo /etc/init.d/nginx start
```
Config here:
```
$ sudo vim /etc/nginx/nginx.conf
```
Shutdown:
```
$ nginx -s quit
```

**注意：**
普通用户无法启动 nginx, 其端口为80，1024以下的端口只有 root 能监听，
有些文件也无法访问

**解决：**
切换 root 用户, /usr/sbin：
```
# chown root nginx
# chmod u+s nginx
```
- nginx startup cmd

```
[root@localhost ~]# /usr/local/nginx/sbin/nginx
[root@localhost ~]# netstat -ano |grep 80
```

#### 2. Install tomcat as a service

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

```
[root@srv6 init.d]# chmod 755 tomcat
[root@srv6 init.d]# chkconfig --add tomcat
[root@srv6 init.d]# chkconfig --level 234 tomcat on
```

> [reference url](http://www.davidghedini.com/pg/entry/install_tomcat_7_on_centos)

#### 3. Install Redis

```
[root@nginx redis]# make && make install
[root@nginx redis]# redis-server /etc/redis/redis.conf
```

```
vi /etc/redis/redis.conf
```

> 仅修改： daemonize yes （no-->yes）

```
echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local
```

> [CentOS6 安装 Redis](https://segmentfault.com/a/1190000002685224)

#### 4. install mysql
查看 mysql：

```
yum list | grep mysql
```

安装：

```
yum install mysql-server mysql mysql-devel
```


查看安装结果：

```
[root@nginx ~]# rpm -qi mysql-server
```

首次启动：

```
[root@nginx ~]# service mysqld start
```

Set a password:

```
[root@nginx ~]# /usr/bin/mysqladmin -u root password 123456
```

设置开机自启动：

```
[root@nginx ~]# chkconfig mysqld on
```

#### 5. install varnish
使用epel源

```
[root@localhost bin]# rpm -ivh http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm
```

check varnish:

```
[root@localhost bin]# yum list|grep varnish
```

Startup:

```
[root@localhost bin]# service varnish start
```

Check:

```
[root@localhost bin]# varnishd -V
```

For more details:
> [Varnish official User Guide](https://varnish-cache.org/docs/4.0/index.html#)

#### 6. Install Maven
wget and tar and config the env.

For mvnw:

https://github.com/takari/maven-wrapper

#### 1. Install ZooKeeper
- 启动

```
zookeeper-3.4.6/bin/zkServer.sh start
```

- 输入jps命令查看进程

```
1573 QuorumPeerMain
1654 Jps
```

### Configuration
### 1. Set time zone

```
[root@demo81 Asia]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

[root@demo81 Asia]# vim /etc/sysconfig/clock
```

Add line:

```
UTC=false or no — The hardware clock is set to local time.
```

Reload:

```
[root@demo81 Asia]# /etc/init.d/ntpd restart
OR:
[root@dlp ~]# source /etc/sysconfig/clock
```

Check:

```
[root@demo81 Asia]# date
  2016年 05月 26日 星期四 09:36:10 CST
```

#### 2. 环境变量 $PATH

```
# echo 'pathmunge /usr/local/maven/bin' > /etc/profile.d/mvn.sh
# chmod +x /etc/profile.d/mvn.sh
# . /etc/profile
# echo $PATH
/usr/local/maven/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/jdk/bin:/usr/local/jdk/jre/bin:/root/bin:/usr/local/jdk/bin:/usr/local/jdk/jre/bin
```
pathmunge 是redhat版本中系统变量profile中的函数，判断当前系统是否有此命令，如果没有，放置在之前或之后 ？


#### 3. vim
1. Vim show line number 显示行号
```
Find vimrc file:
$ whereis vimrc
$ sudo vim /etc/vimrc
Add: set nu!
```

1. Vim 查找单词
配置：
> set insearch

使用 /word 查找单词

[vim 深造](http://coolshell.cn/articles/5426.html)
