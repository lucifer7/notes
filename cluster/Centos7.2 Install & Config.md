### 1. 手动分区
/boot   for 200m ext4
swap    for 2G  swap
/home   for 2G  ext4
/       the rest
![image](http://note.youdao.com/yws/res/6333/WEBRESOURCEd58258c0dd3dd6a0359d32824e9c7ced)

### 2. IP addr
> ifconfig not found

CentOS 7 minimal systems, use the commands “ip addr” and “ip link” to find the details of a network interface card. To know the statistics use “ip -s link”.

**Install ifconfig**
> yum provides ifconfig

> yum install net-tools

### 配置静态IP地址
1. 查看网络管理器服务的状态：
> [root@localhost network-scripts]# systemctl status NetworkManager.service

2. 查看网络管理器管理的网络接口: (for eno167 is our goal)
> [root@localhost network-scripts]# nmcli dev status
DEVICE       TYPE      STATE      CONNECTION
eno16777736  ethernet  connected  eno16777736
lo           loopback  unmanaged  --

3. 修改配置文件
> vim /etc/sysconfig/network-scripts/eno16777736

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
```

4. 重启网络服务：
> # systemctl restart network.service

5. ping connect: Network is unreachable

> ping -c 4 192.168.1.9

用指令dhclient來與DHCP連線:
> dhclient


