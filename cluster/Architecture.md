[TOC]

## Architecture

### 1. Scalable Computing/Analysis
#### 1.1 Three architecture ?

##### 1.1.1 Shared-Nothing versus Shared-Everything
- Patterns for Distributed or Parallel computing
- big problems, break it down, computing unit
- which pattern to use
trader-off between compute and communication costs

##### 1.1.2 Shared-Memory
1. single machine, multiple processes
Apply in:
- GPU-based parallel-computing architecture
- Deep Learning
2. multiple machines
RDMA (Remote direct memory access)
data-center network
warehouse-scale computing

##### 1.1.3 Shared-Storage
- in 80's, NFS/DFS/Andrew
- today, SAN/NAS/Virtualized storage

### 2. Core elements in architecture
架构核心要素，思维导图
![架构核心五要素](http://images.cnitblog.com/blog/90573/201404/131651286223938.jpg)

Ref Book:
[大型网站技术架构](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00F3Z26G8?ie=UTF8&SubscriptionId=AKIAJOMEZLLKFEWYT4PQ&camp=2025&creative=165953&creativeASIN=B00F3Z26G8&linkCode=xm2&tag=z08-23)

### 3. Server Architecture
#### 3.1 服务器三大体系结构
Three architecture for business server

1. Symmetric Multi-Processing (SMP)  **主流服务器架构**
对称多处理架构：多个CPU对称工作，无主次或从属关系
Share same physical memory, access to any address takes same time
Thus is also called: 一致存储访问结构
Uniform Memory Access (UMA)  
Feature: sharing
Extension: more memory, faster CPU, more CPU, more i/o, more device
Defect: extension limited
Best practice: 2-4 CPUs

2. Massively Parallel Processing (MPP)
海量并行处理结构: 多个SMP服务器（节点）互联而成
完全无共享结构(Share Nothing)
Thus：扩展能力最好，目前512个节点，数千个CPU
MPP vs. NUMA
| Criteria | MPP | NUMA |
|--|:--|:--|
| 节点互联机制 | I/O | inside same physical server |
| 内存访问机制 | access only local | access to all, remote delays |
| Usage | data warehouse, i/o | OLTP, little data, fast processing |

3. Non-Uniform Memory Access (NUMA)
非一致存储访问结构
Feature: 多个CPU模块，每个模块多个CPU
独立内存，I/O
通过互联模块（eg. crossbar switch) 连接和信息交互
支持上百个CPU
Defects：远地内存延时太多
系统性能无法线性增加

[服务器三大体系SMP、NUMA、MPP介绍](http://server.51cto.com/sCollege-198840.htm)
[Transitioning from SMP to MPP, the why and the how](https://blogs.technet.microsoft.com/dataplatforminsider/2014/07/30/transitioning-from-smp-to-mpp-the-why-and-the-how/)
[Multi core basics: AMP and SMP](http://www.embedded.com/design/mcus-processors-and-socs/4429496/Multicore-basics)

#### 3.2 多核系统三大架构
1. SMP
每个CPU内核运行一个独立的操作系统或同一操作系统的独立实例（instantiation）

1. Asymmetric Multi-Processing (AMP)
非对称多处理架构
一个操作系统的实例可以同时管理所有CPU内核，且应用并不绑定某一个内核
AMP vs. SMP
| Criteria | AMP | SMP |
|---|:--|:--|
| 耦合 | 松耦合多CPU系统 | 紧耦合多CPU系统 |
| OS | one CPU, one OS | multi CPU, share one OS |
| Apply | less | more |

1. Bound Multi-Processing (BMP)
混合多处理架构
一个操作系统的实例可以同时管理所有CPU内核，但每个应用被锁定于某个指定的核心
