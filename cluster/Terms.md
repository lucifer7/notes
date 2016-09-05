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

1. CPS（Cyber-Physical Systems）
物联网(us)
OR: IoT(Internet of Things)

1. 多跳网络（multi-hop），以及自组织网络（ad-hoc）

Solutions see : https://www.iron.io/spikability-applications-ability-to/

1. Definition of Modern Web Browser
- successfully rendering a site
- all essentials functioning well
- without any browser-specific hacks, forks or workarounds

1. network bandwidth and latency
带宽与延迟

### 2.0 Internet Association
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
congestion prevents or limits useful communication
choke point: incoming traffic exceeds outgoing bandwidth

### 3.0 SQL
predicate 谓词
OR 断言
e.g. BETWEEN EXISTS IN

spatial index
空间索引，用于图等

### 4.0 Java Performance
1. 