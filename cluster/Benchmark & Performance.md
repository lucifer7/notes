[TOC]
Start date: 2016-05-18

# Benchmark | Performance
## 一、背景与工具
### 1. Background
- Performance: the response time for an individual request
- Scalability: the ability to deliver with optimal response time to a larger number of concurrent requests
High-performance is the ability to deliver a page in under a second; 
scalability is the ability to deliver that page in under a second for many requests. 
trade-offs between performance and scalability.

### 2. Ab test
#### 2.1 Install Apache Benchmark
> [root@localhost ~]# yum provides /usr/bin/ab

> [root@localhost ~]# yum install httpd-tools

Simplest Example:
> [root@localhost ~]# ab http://10.200.157.82:80/

-n number
-c concurrency
-k keep-alive (HTTP session)
-C cache attribute(NO_CACHE=1)

#### 2.2 Analyze result
**requests per second**: the most critical element in regards to the scalability of a site. 
Mind **90/95% response time**: this gives an idea of how the site is actually performing. 
Check the number of failed requests, if not 0, investigation is necessary.

#### 2.3 Note
> Testing with a session cookie to emulate the experience of a logged-in user is extremely important, as the contrast between an anonymous user and a logged-in user may be drastically different.

#### 2.4 Other Testing Tools
J-Meter
The Grinder
Blitz.io

### 3. HttpErf Web压力测试
#### 3.1 安装
可按指定规律进行压力测试，模拟真实环境

Download:
[Visit official website]()http://www.labs.hpe.com/research/linux/httperf/)

Install:
tar, ./configure, make, install

Simplest Example:
> [root@localhost ~]# httperf --hog --server=10.200.157.82 --port=80

Adding more args:
> [root@localhost ~]# httperf --hog --server=10.200.157.82 --port=80 --num-conns 1000 --timeout 7 --wsess 20,15,0.1

#### 3.2 FD_SIZE 配置
Error shown:
> httperf: warning: open file limit > FD_SETSIZE; limiting max. # of open files to FD_SETSIZE

Search config file:
>[root@localhost ~]# grep -E "#define\W+__FD_SETSIZE" /usr/include/*.h /usr/include/*/*.h
/usr/include/bits/typesizes.h:#define	__FD_SETSIZE		1024
/usr/include/linux/posix_types.h:#define __FD_SETSIZE	1024
[root@localhost ~]# vi /usr/include/bits/typesizes.h
[root@localhost ~]# vi /usr/include/linux/posix_types.h

Modify 1024 to 65535

See Reference:
[Increasing the number of open file descriptors](https://cs.uwaterloo.ca/~brecht/servers/openfiles.html)

### 4. AutoBench
Download:
http://www.xenoclast.org/autobench/

Install:
tar, make, make install

### 5. Helpful tools..
#### 5.1 gnuplot 图表
portable command-line driven graphing utility

Download:
http://www.gnuplot.info/download.html

Install：
tar, ./configure, make, make install

Setting:
> [root@localhost bin]# echo set data style linespoints >> gnuplot.cmd

NOTES: unsuccessful, paused

### 6. JMeter

### 7. HttpRequester  ?

## 二、Varnish 与 Nginx 性能测试
### 1. Varnish: Accelerator
#### 1.1 Intro (from Wiki)
> Varnish is an HTTP accelerator designed for content-heavy dynamic web sites. In contrast to other HTTP accelerators, such as Squid, which began life as a client-side cache, or Apache, which is primarily an origin server, Varnish was designed from the ground up as an HTTP accelerator.

> Varnish is heavily threaded, with each client connection being handled by a separate worker thread.

#### 1.2 Without keep-alive:
> [root@localhost bin]# ab -n 10000 -c 500 http://10.200.157.50/

Performance:
<pre>
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   12   2.9     11      26
Processing:    10   25   6.8     27      40
Waiting:        0   18   5.9     21      28
Total:         28   37   5.1     38      49
</pre>

#### 1.3 With keep-alive(HTTP for Ab)
> [root@localhost bin]# ab -n 10000 -c 500 -k http://10.200.157.50/

Performance:
<pre>
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.7      0      17
Processing:     1    1   0.3      1      26
Waiting:        0    0   0.2      0       5
Total:          1    1   0.8      1      26
</pre>

They differ dramatically.

#### 1.4 Load and Performance Practice
- verify varnish cache
> [root@localhost ~]# curl -I http://10.200.157.50:80/
<pre>
HTTP/1.1 200 OK
Server: nginx/1.6.2
Content-Type: text/html;charset=ISO-8859-1
Vary: Accept-Encoding
Content-Length: 11197
Date: Wed, 18 May 2016 04:50:29 GMT
X-Varnish: 552596393
Age: 0
Via: 1.1 varnish
Connection: keep-alive
</pre>
See if "Age" is bigger than 0

- Timing when no cache
> [root@localhost ~]# time curl -I -H "Cookie: NO_CACHE=1;" http://10.200.157.50:80/
<pre>
HTTP/1.1 200 OK
Server: nginx/1.6.2
Content-Type: text/html;charset=ISO-8859-1
Vary: Accept-Encoding
Date: Wed, 18 May 2016 04:56:40 GMT
X-Varnish: 552596394
Age: 0
Via: 1.1 varnish
Connection: keep-alive
real	0m0.230s
user	0m0.006s
sys	0m0.020s
</pre>

- Testing scale and Throughout (with Ab)
> [root@localhost ~]# ab -n 10000 -c 1000 -C NO_CACHE=1 http://10.200.157.82/

slightly lower than cache enabled
> [root@localhost ~]# ab -n 10000 -c 1000 -C NO_CACHE=0 http://10.200.157.82/


Reference URL:
[Load and Performance Testing](https://pantheon.io/docs/load-and-performance-testing/)

Try refs:

[Nginx+Varnish compared to Nginx](ttps://www.garron.me/en/go2linux/nginx-varnish-vs-nginx-alone-compared.html)
Conclusion: no need to use varnish on nginx

[TCP keep-alive means...](https://blogs.technet.microsoft.com/nettracer/2010/06/03/things-that-you-may-want-to-know-about-tcp-keepalives/)

### 2. keep-alive mechanism
#### 2.1 keep-alive for Nginx
HTTP uses a mechanism called keepalive connections to hold open the TCP connection between the client and the server after an HTTP transaction has completed. 
If the client needs to conduct another HTTP transaction, it can use the idle keepalive connection rather than creating a new TCP connection.
![saving hand-shakes](https://assets.wp.nginx.com/wp-content/uploads/2014/03/tcpka.png)

But too many clients with keep-alive connections may cause severe performance problem.
Tragedy of commons 公地悲剧
some with idle keep-alive, while some cannot connect to server

In practice,
one client, 8 TCP connections, one TCP connections consumes one slot on server

Nginx has a mechanism to prevent this dilemma

#### 2.2 How Nginx improve performance
asynchronous  event-driven model
concurrent queries from client
proxy them to upstream server
with a local pool of keep-alive connections
![concurrency to upstream server](https://assets.wp.nginx.com/wp-content/uploads/2014/03/concurrency.png)

cache upstream responses

See Reference：

[keep-alive in Nginx](https://www.nginx.com/blog/http-keepalives-and-web-performance/)


