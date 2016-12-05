### 1. 手动分区

```
/boot   for 200m ext4
swap    for 2G  swap
/home   for 2G  ext4
/       the rest
```

![image](http://note.youdao.com/yws/res/6333/WEBRESOURCEd58258c0dd3dd6a0359d32824e9c7ced)

### 2. IP addr

```
ifconfig not found
```


CentOS 7 minimal systems, use the commands “ip addr” and “ip link” to find the details of a network interface card. To know the statistics use “ip -s link”.

**Install ifconfig**

```
yum provides ifconfig
yum install net-tools
```

**Mind: BOTH _ifconfig_ and _netstat_ is obsolete, use _ip_ and _ss_ instead.**

### 3. 配置静态IP地址
1. 查看网络管理器服务的状态：

```
[root@localhost network-scripts]# systemctl status NetworkManager.service
```

2. 查看网络管理器管理的网络接口: (for eno167 is our goal)

```
[root@localhost network-scripts]# nmcli dev status
DEVICE       TYPE      STATE      CONNECTION
eno16777736  ethernet  connected  eno16777736
lo           loopback  unmanaged  --
```


3. 修改配置文件

```
vim /etc/sysconfig/network-scripts/eno16777736
```

```
*BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736"
UUID="4c9bca38-d727-4352-b5c9-a4be75a8ed62"
DEVICE="eno16777736"
ONBOOT="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_PRIVACY="no"
*NM_CONTROLLED=no       #不通过网络管理器管理
*IPADDR=192.168.1.15    #must
*DNS1={DNS}
```

4. 重启网络服务：

```
systemctl restart network.service
```

5. ping connect: Network is unreachable

```
ping -c 4 192.168.1.9
```

用指令dhclient來與DHCP連線:

```
$ dhclient
```

6. yum "Couldn't resolve host 'mirrorlist.centos.org'" for CentOS 6
```
# vim /etc/resolv.conf

And append:

 nameserver 8.8.8.8 nameserver 8.8.4.4 nameserver 127.0.0.1

```

7. Destination unreachable / No route to host / 断网等各种事件
因为VM ware Bridge 模式无法使用，在虚拟机网络编辑器里重置默认值，然后在win10宿主机中为VM net的属性中选上Bridge protocol.

究极大法：Shutdown all VMs, Reboot Host, Reset VMware adapter, Boot VMs, Shutdown firewall

### 4. Install lrzsz Error

```
Error downloading packages: lrzsz-0.12.20-36.el7.x86_64: [Errno 256] No more mirrors to try.
```

Try this:

```
$ yum clean all
$ yum makecache
```

and ***dhclient*** if necessary

### 5. 防火墙问题
Visit tomcaat at http://192.168.1.15:8080/ failed.
Caused by firewall, stop or disable it:

```
$ systemctl stop firewalld.service
```
OR
```
[root@localhost ~]# systemctl disable firewalld.service
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```
OR 允许8080端口流量通过

```
firewall-cmd --zone= --add-port=8080/tcp --permanent
```

参考：

```
firewall-cmd --state # 检查防火墙状态
firewall-cmd --add-service=http --permanent # 允许服务流量通过防火墙，后跟 --permanent 为永久修改，否则重载以后就不失效了
firewall-cmd --zone= --add-port=80/tcp --permanent # 允许端口流量通过防火墙
firewall-cmd --reload # 重新加载防火墙
```
> [Red Hat 防火墙使用](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)

### 6. Hostname 修改

```
[root@localhost ~]# hostnamectl set-hostname master15
```

查看结果(静态、瞬态和灵活主机名):

```
$ hostnamectl status
```

**注意**

对于静态/瞬态主机名，任何特殊字符或空格都会被移除，字母转小字。修改静态主机名，/etc/hostname 将被自动更新。但/etc/hosts需要手动更新。

### 7. User and SSH
Initial new server by new user and SSH:
[New Server Initial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)

### 8. Configuring Basic Firewall
Start up _firewalld_:
```
sudo systemctl start firewalld
```
Allow SSH with default port:
```
sudo firewall-cmd --permanent --add-service=ssh
```
Timezone config:
```
sudo timedatectl
sudo timedatectl set-timezone Asia/Shanghai

ntpdate time.windows.com 
hwclock -w 
```

