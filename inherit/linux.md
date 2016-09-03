# Linux
[TOC]

## 零、 Linux Tools Quick Tutorial
http://linuxtools-rst.readthedocs.io/zh_CN/latest/base/index.html

## 一、Linux 常用软件安装

特殊软件会在相应的章节里详细说明，此处仅仅是一些简单到不能更简单的软件安装 .
**以下没有特殊说明，是指在centos6.5中安装**

### 1.1 配置静态IP（虚拟机中）
> vim /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0
HWADDR=00:0C:29:53:EB:65
TYPE=Ethernet
UUID=bd5805d5-cb36-4a9b-8b4e-603b5da58b14
ONBOOT=yes
BOOTROTO=static
IPADDR=192.168.0.201
NETMARK=255.255.255.0
GATEWAY=192.168.0.1
DNS1=192.168.1.1
USERCTL=no
```

重启网络服务：
> service network restart

### 1.2 Java

**1.首先如何卸载原本的`openjdk`**

```
[root@localhost ~]# rpm -qa|grep jdk
java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64

 [root@localhost ~]# rpm -qa|grep gcj
java-1.4.2-gcj-compat-1.4.2.0-40jpp.115
 libgcj-4.1.2-48.el5

[root@localhost ~]# yum -y remove java java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
[root@localhost ~]# yum -y remove java java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
[root@localhost ~]# yum -y remove java java-1.4.2-gcj-compat-1.4.2.0-40jpp.115
 [root@localhost ~]# yum -y remove libgcj-4.1.2-48.el5
```

**2.下载并解压jdk**
**3. 配置环境变量**

```
export JAVA_HOME=/usr/java/jdk1.6.0_22
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
```

### 1.3 rz sz

`yum -y install lrzsz `


### 1.4  如何复制虚拟机中的centos 6.5

主要是解决 网卡 eth0问题

修改`/etc/udev/rules.d/70-persistent-net.rules`
将eth0这行注释掉或者删除，这里记载的还是克隆系统时的MAC地址，但是新启动的系统MAC已经更改，
1 .将**NAME**="eth1" 改为 “eth0”，ATTR 标记的MAC地址，这个是虚拟机为这个虚拟网卡分配的MAC
2. **用上面的MAC**替换掉 /etc/sysconfig/network-scripts/ifcfg-eth0中的MAC
3. 然后重启即可

还有一个办法，不用eth0,直接用eth1等，把/etc/sysconfig/network-scripts/ifcfg-eth0复制成/etc/sysconfig/network-scripts/ifcfg-eth1


### 1.5 修改hostname

需要修改两处：
1. 一处是/etc/sysconfig/network，另一处是/etc/hosts，只修改任一处会导致系统启动异常。首先切换到root用户。

2. /etc/sysconfig/network
用任一款你喜爱的编辑器打开该文件，里面有一行 HOSTNAME=localhost.localdomain (如果是默认的话），修改 localhost.localdomain 为你的主机名。

3. /etc/hosts
打开该文件，会有一行 127.0.0.1 localhost.localdomain localhost 。其中 127.0.0.1 是本地环路地址， localhost.localdomain 是主机名(hostname)，也就是你待修改的。localhost 是主机名的别名（alias），它会出现在Konsole的提示符下。将第二项修改为你的主机名，第三项可选。

> 特别注意：请把/etc/hosts文件中的第一行localhost也修改掉 啧啧

> 同时配置多台虚拟机：[root@demo80 ~]# scp /etc/hosts/ 10.200.157.82:/etc/hosts

> 如果系统重装，会报错：remote host identification has changed

> 解决：
```
[root@demo80 ~]# ssh-keygen -R 10.200.157.82
```


### 1.6 install tomcat as a service

```
#!/bin/bash
# description: Tomcat Start Stop Restart
# processname: tomcat
# chkconfig: 234 20 80
JAVA_HOME=/usr/java/jdk1.7.0_60
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
CATALINA_HOME=/usr/share/apache-tomcat-8.0.8

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

### 1.7 Epoll
EPOLL事件有两种模型：
**Edge Triggered (ET)** 和
**Level Triggered (LT)**

二者的差异在于level-trigger模式下只要某个socket处于readable/writable状态，无论什么时候进行epoll_wait都会返回该socket；而edge-trigger模式下只有某个socket从unreadable变为readable或从unwritable变为writable时，epoll_wait才会返回该socket。
