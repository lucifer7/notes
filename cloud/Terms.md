## terms

### 1.0 Computer Science
1. spatial and temporal locality    
空间局部性   
时间局部性

1. Fluent Interface
- implemented by using method cascading (concretely method chaining)
- relay the instruction context of a subsequent call
流接口

1. Application Spikability
DEF: An Application's Ability to Handle Unknown and/or Inconsistent Load    
how an application can handle spiky behavior    
峰值处理能力？     
Solutions see : https://www.iron.io/spikability-applications-ability-to/

1. CPS（Cyber-Physical Systems）
物联网(us)
OR: IoT(Internet of Things)

1. Actor模型
Share Nothing 
类似消息队列 
相关： event sourcing, CQRS

1. Arity
The number of arguments to a function is called it's arity, and is used to help identify the functions. 
函数的参数数量称为元数 (arity)，用于帮助标识函数。

1. thrashing
（系统）抖动, memory page 

1. cgroups
control groups
a Linux kernel feature
limits, accounts for, and isolates the resource usage(CPU, memory, disk i/o, network, etc.) of a collection of processes.

### 2.0 Internet Association
1. 多跳网络（multi-hop），以及自组织网络（ad-hoc）

1. Definition of Modern Web Browser
- successfully rendering a site
- all essentials functioning well
- without any browser-specific hacks, forks or workarounds

1. network bandwidth and latency
带宽与延迟

1. diminishing marginal returns
边际收益递减

1. Chunked Transfer Encoding
a data transfer mechanism, brought in HTTP 1.1(which is supported by almost every browser)
data send in a series of chunks


1. 增量 Incremental
存量 Stock

1. 美国Fintech < -- > 中国互联网金融

1. REST: PUT <--> POST
语义有区别，
put: 幂等
post: 非幂等

1. 缓存击穿
Cache stampedes / dogpile effect / dog-piling

? Definition: Unexpected increase in requests after cache invalid

雪崩？
解决：
返回旧值、加锁某个线程去拿新值、单开一个线程拿值

1. Congestive collapse  
拥塞崩溃？
congestion prevents or limits useful communication
choke point: incoming traffic exceeds outgoing bandwidth

1. Protocol Stack  
Or network stack, is an implementation of a computer networking protocol suite(or protocol family).

Suite is the definition of the Communication Protocols, stack is the software implementation.


### 3.0 SQL
predicate  
谓词 OR 断言  
e.g. BETWEEN EXISTS IN

spatial index
空间索引，用于图等

### 4.0 Java Performance
1. 

### 5.0 Software Engineering
1. MVP原则
 Minimum Viable Product，最小可用产品
 
2. OTT应用
 over the top, eg. app store, wechat 
 
3. Generic
Non-trivial   非平凡，大型程序

4. KISS principle
Keep it simple, stupid
   
5.  