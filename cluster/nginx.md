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

