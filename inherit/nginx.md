[TOC]

# 帮助文档

[nginx-demo](http://www.360doc.com/content/16/0503/15/32978314_555913915.shtml)
[nginx手册](http://shouce.jb51.net/nginx/index.html)
# 一、Nginx基础

## 1.1 nginx简介

	Nginx是一款轻量级的Web服务器，也是一款轻量级的**反向代理服务器**。

**Nginx常常功能丰富：**

		1：直接支持Rails和PHP的程序
		2：作为HTTP反向代理服务器
		3：作为负载均衡服务器
		4：作为邮件代理服务器
		5：帮助实现前端动静分离


**Nginx的优点：**

		高稳定、高性能、资源占用少、功能丰富、模块化结构、支持热部署


## 1.2 nginx安装

**源码安装**
演示环境：CentOS6.5

		安装make：
		yum -y install gcc automake autoconf libtool make
		安装g++:
		yum install gcc gcc-c++
		下面正式开始
		---------------------------------------------------------------------------
		一般我们都需要先装pcre, zlib，前者为了重写rewrite，后者为了gzip压缩。

		1.选定源码目录
		可以是任何目录，本文选定的是/usr/local/src
		cd /usr/local/src

		2.安装PCRE库
		ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/ 下载最新的 PCRE 源码包，使用下面命令下载编译和安装 PCRE 包：
		cd /usr/local/src
		wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.34.tar.gz
		tar -zxvf pcre-8.34.tar.gz
		cd pcre-8.34
		./configure
		make
		make install
		如果支持yum可以直接yum install pcre*

		3.安装zlib库
		http://zlib.net/zlib-1.2.8.tar.gz 下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：
		cd /usr/local/src

		wget http://zlib.net/zlib-1.2.8.tar.gz
		tar -zxvf zlib-1.2.8.tar.gz
		cd zlib-1.2.8
		./configure
		make
		make install
		如果支持yum，可以直接yum install zlib zlib-devel

		4.安装ssl（某些vps默认没装ssl)
		cd /usr/local/src
		wget http://www.openssl.org/source/openssl-1.0.1c.tar.gz
		tar -zxvf openssl-1.0.1c.tar.gz
		如果支持yum，可以直接yum install openssl openssl-devel

		5.安装nginx
		Nginx 一般有两个版本，分别是稳定版和开发版，您可以根据您的目的来选择这两个版本的其中一个，下面是把 Nginx 安装到 /usr/local/nginx 目录下的详细步骤：
		./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

		6.启动
		确保系统的 80 端口没被其他程序占用，运行/usr/local/nginx/nginx 命令来启动 Nginx，
		netstat -ano|grep 80
		如果查不到结果后执行，有结果则忽略此步骤（ubuntu下必须用sudo启动，不然只能在前台运行）
		sudo /usr/local/nginx/nginx
		打开浏览器访问此机器的 IP，如果浏览器出现 Welcome to nginx! 则表示 Nginx 已经安装并运行成功。


**启动nginx:**

- 测试配置文件：
安装路径下的/nginx/sbin/nginx -t

- 启动：
安装路径下的/nginx/sbin/nginx

- 停止
安装路径下的/nginx/sbin/nginx -s stop
或者是： nginx -s quit

- 重启
安装路径下的/nginx/sbin/nginx -s reload



## 1.3 Nginx 配置文件

nginx.conf
```

#user  nobody;

#开启进程数 <=CPU数
worker_processes  1;

#错误日志保存位置
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程号保存文件
#pid        logs/nginx.pid;

#每个进程最大连接数（最大连接=连接数x进程数）每个worker允许同时产生多少个链接，默认1024
events {
    worker_connections  1024;
}


http {
	#文件扩展名与文件类型映射表
    include       mime.types;
	#默认文件类型
    default_type  application/octet-stream;

	#日志文件输出格式 这个位置相于全局设置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

	#请求日志保存位置
    #access_log  logs/access.log  main;

	#打开发送文件
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
	#连接超时时间
    keepalive_timeout  65;

	#打开gzip压缩
    #gzip  on;

	#设定请求缓冲
	#client_header_buffer_size 1k;
	#large_client_header_buffers 4 4k;

	#设定负载均衡的服务器列表
	#upstream myproject {
		#weigth参数表示权值，权值越高被分配到的几率越大
		#max_fails 当有#max_fails个请求失败，就表示后端的服务器不可用，默认为1，将其设置为0可以关闭检查
		#fail_timeout 在以后的#fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
	#}

    #webapp
    #upstream myapp {
  	# server 192.168.1.171:8080 weight=1 max_fails=2 fail_timeout=30s;
	# server 192.168.1.172:8080 weight=1 max_fails=2 fail_timeout=30s;
    #}

	#配置虚拟主机，基于域名、ip和端口
    server {
		#监听端口
        listen       80;
		#监听域名
        server_name  localhost;

        #charset koi8-r;

		#nginx访问日志放在logs/host.access.log下，并且使用main格式（还可以自定义格式）
        #access_log  logs/host.access.log  main;

		#返回的相应文件地址
        location / {
            #设置客户端真实ip地址
            #proxy_set_header X-real-ip $remote_addr;
			#负载均衡反向代理
			#proxy_pass http://myapp;

			#返回根路径地址（相对路径:相对于/usr/local/nginx/）
            root   html;
			#默认访问文件
            index  index.html index.htm;
        }

		#配置反向代理tomcat服务器：拦截.jsp结尾的请求转向到tomcat
        #location ~ \.jsp$ {
        #    proxy_pass http://192.168.1.171:8080;
        #}

        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #

		#错误页面及其返回地址
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

	#虚拟主机配置：
	server {
		listen 1234;
		server_name bhz.com;
		location / {
		#正则表达式匹配uri方式：在/usr/local/nginx/bhz.com下 建立一个test123.html 然后使用正则匹配
		#location ~ test {
			## 重写语法：if return （条件 = ~ ~*）
			#if ($remote_addr = 192.168.1.200) {
			#       return 401;
			#}

			#if ($http_user_agent ~* firefox) {
			#	   rewrite ^.*$ /firefox.html;
			#	   break;
			#}

			root bhz.com;
			index index.html;
		}

		#location /goods {
		#		rewrite "goods-(\d{1,5})\.html" /goods-ctrl.html;
		#		root bhz.com;
		#		index index.html;
		#}

		#配置访问日志
		access_log logs/bhz.com.access.log main;
	}



    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```




# 二、Nginx功能模块


## 2.1 反向代理

**(1).正向代理的概念**

       正向代理，也就是传说中的代理,他的工作原理就像一个跳板，简单的说，我是一个用户，我访问不了某网站，但是我能访问一个代理服务器，这个代理服务器呢，他能访问那个我不能访问的网站，于是我先连上代理服务器，告诉他我需要那个无法访问网站的内容，代理服务器去取回来，然后返回给我。从网站的角度，只在代理服务器来取内容的时候有一次记录，有时候并不知道是用户的请求，也隐藏了用户的资料，这取决于代理告不告诉网站。

       结论就是，正向代理 是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

**(2).反向代理的概念**

继续举例:
       例用户访问 http://www.test.com/readme，但www.test.com上并不存在readme页面，他是偷偷从另外一台服务器上取回来，然后作为自己的内容返回用户，但用户并不知情。这里所提到的 www.test.com 这个域名对应的服务器就设置了反向代理功能。

       结论就是，反向代理正好相反，对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容原本就是它自己的一样。

**(3).两者区别**

从用途上来讲：

       正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率。反向代理的典型用途是将防火墙后面的服务器提供给Internet用户访问。反向代理还可以为后端的多台服务器提供负载平衡，或为后端较慢的服务器提供缓冲服务。另外，反向代理还可以启用高级URL策略和管理技术，从而使处于不同web服务器系统的web页面同时存在于同一个URL空间下。

从安全性来讲：

       正向代理允许客户端通过它访问任意网站并且隐藏客户端自身，因此你必须采取安全措施以确保仅为经过授权的客户端提供服务。反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。

```
location / {
  proxy_pass        http://localhost:8000;
  proxy_set_header  X-Real-IP  $remote_addr;
}
```


	指令说明：proxy_set_header

	语法：proxy_set_header header value
	默认值： Host and Connection
	使用字段：http, server, location
	这个指令允许将发送到被代理服务器的请求头重新定义或者增加一些字段。这个值可以是一个文本，变量或者它们的组合。proxy_set_header在指定的字段中没有定义时会从它的上级字段继承。


## 2.2 负载均衡

**1.demo**

```
upstream test.net{
ip_hash;
server 192.168.10.13:80;
server 192.168.10.14:80  down;
server 192.168.10.15:8009  max_fails=3  fail_timeout=20s;
server 192.168.10.16:8080;
}
server {
  location / {
    proxy_pass  http://test.net;
  }
}
```

`upstream`是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。在上面的设定中，通过upstream指令指定了一个负载均衡器的名称test.net。这个名称可以任意指定，在后面需要用到的地方直接调用即可。

**2.nginx的负载均衡策略：**

Nginx的负载均衡模块目前支持4种调度算法，下面进行分别介绍，其中后两项属于第三方调度算法。

- 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。Weight 指定轮询权值，Weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。

- ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。

- fair。这是比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。

- url_hash。此方法按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。

**3.upstream 支持的状态参数**

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

down，表示当前的server暂时不参与负载均衡。

backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。

max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

注，当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。


## 2.3 实现简单缓存

一般不会考虑使用nginx作为高速缓存，一般使用varnish来作为告诉缓存

## 2.4 动静分离

## 2.5 读写分离

## 2.6 rewrite


# 三、Nginx模块

## 3.1 nginx 示例配置

```
user root;
worker_processes  1;

# 生产环境级别可以调高一点
error_log  logs/error.log crit;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        logs/nginx.pid;


events {
    use epoll;
    worker_connections  1024;
}


http {
    include       mime.types;
    include	  ccproxy.conf;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    server_names_hash_bucket_size 256;

	# 设置client参数
    client_body_buffer_size 128k;
    client_header_buffer_size 8k;
    client_max_body_size 50m;
    client_body_timeout 1m;
    client_header_timeout 1m;
    large_client_header_buffers 4 8k;

    send_timeout	3m;


    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  120;
    tcp_nodelay on;
    tcp_nopush on;


	# 设置gzip功能
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

	# 设置负载均衡
    #load balance
    upstream cctest1.com{
	server 127.0.0.1:9080 weight=10;
	server 127.0.0.1:8080 weight=5;
	#server 127.0.0.1:8080 weight=5;
    }

    upstream cctest2.com{
		server 127.0.0.1:1111;
	}
    server {
        listen       80;
        server_name ccserver1;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

	index index.html index.htm index.jsp;

	root /usr/local/common/tomcat/webapps/ROOT/;

	# 正则表达式没有大小写区分，图片结尾，实际开发中应该像这么明确，不同的location最好不要冲突
	location ~* .*\.(jpg|jpeg|gif|png|swf|ico)$ {
		if (-f $request_filename){
			expires 15d;
			break;
		}
	}

	 location ~* .*\.(html|htm|js|css)$ {
		expires	1d;
	 }

	# 做应用的垂直分布
	location /goods {
		proxy_pass	http://goods.com;
	}

		# 优先级别最低，反向代理
        location / {
			proxy_pass	http://cctest1.com;
			# 设置头部参数，为客户端真实的IP
			proxy_set_header X-Real-IP $remote_addr;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
	    internal;
            root   errors/html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

- ccproxy.conf文件如下：

```
#!nginx (-)
# proxy.conf
proxy_redirect		off;
proxy_set_header	Host $host;
# 设置为真实IP
proxy_set_header	X-Real-IP $remote_addr;
proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
proxy_connect_timeout	90;
proxy_send_timeout	90;
proxy_read_timeout	90;
proxy_buffer_size	4k;
proxy_buffers		4 64k;
proxy_busy_buffers_size	128k;
proxy_temp_file_write_size 64k;

```


具体需要研究核心模块和代理模块

## 3.2 Location

Location区段,通过指定模式来与客户端请求的URI相匹配,基本语法如下:
`location [=|~|~*|^~|@] pattern{......}`

- 没有修饰符 表示:必须以指定模式开始,如:
```
	server {
		server_name sishuok.com;
		location /abc {
			......
		}
	}
```
那么,如下是对的:
http://sishuok.com/abc
http://sishuok.com/abc?p1=11&p2=22
http://sishuok.com/abc/
http://sishuok.com/abcde


- = 表示:必须与指定的模式精确匹配,如:
```
	server {
		server_name sishuok.com;
		location = /abc {
		......
		}
	}
```
那么,如下是对的:
http://sishuok.com/abc
http://sishuok.com/abc?p1=11&p2=22
如下是错的:
http://sishuok.com/abc/
http://sishuok.com/abcde



- ~ 表示:指定的正则表达式要区分大小写,如:
```
	server {
		server_name sishuok.com;
		location ~ ^/abc$ {
		......
		}
	}
```
那么,如下是对的:
http://sishuok.com/abc
http://sishuok.com/abc?p1=11&p2=22
如下是错的:
http://sishuok.com/ABC
http://sishuok.com/abc/
http://sishuok.com/abcde



- ~* 表示:指定的正则表达式不区分大小写,如:
```
server {
	server_name sishuok.com;
	location ~* ^/abc$ {
	......
	}
}
```
那么,如下是对的:
http://sishuok.com/abc
http://sishuok.com/ABC
http://sishuok.com/abc?p1=11&p2=22
如下是错的:
http://sishuok.com/abc/
http://sishuok.com/abcde


- ^~ 类似于无修饰符的行为,也是以指定模式开始,不同的是,如果模式匹配,
那么就停止搜索其他模式了。


- @ :定义命名location区段,这些区段客户段不能访问,只可以由内部产生的请
求来访问,如try_files或error_page等



**查找顺序和优先级:**

1:带有“=“的精确匹配优先
2:没有修饰符的精确匹配
3:正则表达式按照他们在配置文件中定义的顺序
4:带有“^~”修饰符的,开头匹配
5:带有“~” 或“~*” 修饰符的,如果正则表达式与URI匹配
6:没有修饰符的,如果指定字符串与URI开头匹配

```
	location = / {
	# 只匹配 / 的查询.
	[ configuration A ]
	}
	location / {
	# 匹配任何以 / 开始的查询,但是正则表达式与一些较长的字符串将被首先匹配。
	[ configuration B ]
	}
	location ^~ /images/ {
	# 匹配任何以 /images/ 开始的查询并且停止搜索,不检查正则表达式。
	[ configuration C ]
	}
	location ~* \.(gif|jpg|jpeg)$ {
	# 匹配任何以gif, jpg, or jpeg结尾的文件,但是所有 /images/ 目录的请求将在Configuration C中处
	理。
	[ configuration D ]
	}
```

各请求的处理如下例:
/ → configuration A
/documents/document.html → configuration B
/images/1.gif → configuration C
/documents/1.jpg → configuration D

## 3.3 ReWrite 模块

Rewrite模块:用来执行URL重定向。这个机制有利于去掉恶意访问的url,也有
利于搜索引擎优化(SEO)。

Nginx使用的语法源于Perl兼容正则表达式(PCRE)库,基本语法如下:

		^ :必须以^后的实体开头
		$ :必须以$前的实体结尾
		. :匹配任意字符
		[ ] :匹配指定字符集内的任意字符
		[^ ] :匹配任何不包括在指定字符集内的任意字符串
		| :匹配 | 之前或之后的实体
		() :分组,组成一组用于匹配的实体,通常会有|来协助
		n 捕获子表达式,可以捕获放在()之间的任何文本,比如:
		^(.*)(hello|sir)$
		字符串为“hi sir” 捕获的结果: 
		$1=hi
		$2=sir
		这些被捕获的数据,在后面就可以当变量一样使用了

内部请求
外部请求是客户端的url,内部请求是Nginx通过特殊的指令触发。
比如:`error_page、index、rewrite、try_files、include`等等

**内部请求分成两种类型:**
1:内部重定向:URI被改变,可能会匹配到其他的Location
2:子请求:比如使用Addition模块,指令add_after_body允许你在原始的URI之后指定一个URI,会把该URI被处理后的结果,插入到原始的URI的body中。
内部重定向示例:
```
server {
	server_name sishuok.com;
	location /abc/ {
		rewrite ^/abc/(.*)$ /bcd/$1
	}
	location /bcd/{
		internal;
		root pages;
	}
}
```


条件结构的基本语法:

		1:没有操作符:指定的字符串或者变量不为空,也不为0开始的字符串,取true
		2:= , != ,例:if($request_method = POST){...}
		3:~,~*,!~,!~* ,例:if($uri ~* “\.jsp$”){...}
		4:-f,!-f :用来测试指定文件是否存在,例:if(-f $request_filename){...}
		5:-d,!-d :用来测试指定目录是否存在
		6:-e,!-e:用来测试指定文件、目录或者符号链接是否存在
		7:-x,!-x:用来测试指定文件是否存在和是否可以执行
		8:break:跳出if块
		9:return:终止处理,并返回一个指定的http状态码
		10:set:初始化或者重定义一个变量