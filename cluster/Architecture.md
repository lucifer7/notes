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