## CPU
### 1. 流水线
五级流水线处理器
![CPU 五级流水线处理器](https://pic3.zhimg.com/1e3667161c3307d95cf5d863dbe435fe_b.png)
- 级别更多：
- 单条指令时间变长
- 吞吐率提高(ThroughOut)
- 以空间换时间   ？
### 2. Cache

```
Core1       Core2
Register    Register
L1 Cache    L1 Cache            -> divided into data cache and instruction cache ? for Von Neuman or Harvard? 
Bus interface and L2 Cache      -> shared by multi cores
```


### 3. Memory
DRAM vs. SRAM

 Criteria  | DRAM(Dynamic random access memory) | SRAM(Static random access memory)
--|--|--
Bit stored in | separate capacitor in integrated circuit | bistable latching circuitry 双稳态锁存电路？
Refresh |  Need refresh, 电容漏电, thus called dynamic |    No need for periodical refresh
Applications  |  Main memory(eg. DDR3)           |            L2, L3 cache
Typical size |   1G-2G| 4G-16G   


```
WIN7 下查看CPU缓存：
Run as administrator
    > wmic cpu get L2CacheSize, L2CacheSpeed, L3CacheSize, L3CacheSpeed
Linux 下查看CPU:
    $ lscpu
```


## 磁盘
### 1. 机械硬盘
磁盘的读/写原理和效率

磁盘上数据必须用一个三维地址唯一标示：柱面号、盘面号、块号(磁道上的盘块)。

- 查找时间(seek time) Ts: 完成上述步骤(1)所需要的时间。这部分时间代价最高，最大可达到0.1s左右。
- 等待时间(latency time) Tl: 完成上述步骤(3)所需要的时间。由于盘片绕主轴旋转速度很快，一般为7200转/分(电脑硬盘的性能指标之一, 家用的普通硬盘的转速一般有5400rpm(笔记本)、7200rpm几种)。因此一般旋转一圈大约0.0083s。
- 传输时间(transmission time) Tt: 数据通过系统总线传送到内存的时间，一般传输一个字节(byte)大概0.02us=2*10^(-8)s

磁盘读取数据是以盘块(block)为基本单位的。位于同一盘块中的所有数据都能被一次性全部读取出来。而磁盘IO代价主要花费在查找时间Ts上。因此我们应该尽量将相关信息存放在同一盘块，同一磁道中。或者至少放在同一柱面或相邻柱面上，以求在读/写信息时尽量减少磁头来回移动的次数，避免过多的查找时间Ts。

通常：(估计，待核)

```
平均寻道时间   10ms
7200转硬盘的等待时间    8.3ms
传输时间    很短
```


## JVM
> [JVM执行篇：使用HSDIS插件分析JVM代码执行细节](http://www.infoq.com/cn/articles/zzm-java-hsdis-jvm)

## Networking
OSI七层模型
![OSI](http://hi.csdn.net/attachment/201201/5/0_1325744597WM32.gif)

