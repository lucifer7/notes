### 1. Install
see linux.md

### 2. Configuration
#### 2.1 四种配置：
- main 全局配置
- server 主机配置（虚拟主机域名、端口）
- upstrean 上游服务器（反向代理，负载均衡）
- location 地址匹配

server extends main,
location extends server,
upstream is final

#### 2.1 虚拟目录配置
> /etc/nginx/conf.d/*.conf

可在此目录下，一个文件配置一个站点，没有此目录可以新建一个，并在 nginx.conf 中引入。

配置：
- 在 nginx.conf 中配置 upstream
- 在 default.conf 中配置 location

### 3. Sample config
#### 3.1 load balancing

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


#### 3.2 nginx iptables [remove  ]
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
