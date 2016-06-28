### Linux perf
软件性能诊断工具
Use PMU, tracepoint and special counter in kernel

#### Terms

硬件并行设计：
CPU 流水线 | processor pipeline
超标量 Superscalar
乱序执行 reordering ?
Based：相邻指令不依赖

分支预测 branch predict ?
预测最可能的下条指令
对switch case效果不佳

硬件中加入 PMU 
performance monitor unit
允许软件对某种硬件事件设置 counter， 超过预设值时中断

Trackpoint
散落在内核中的hook, 可被 trace/debug 工具使用，如 perf

#### Usages
> perf list
list all monitor event

Three types:
1. Hardware event, generated by PMU, eg. cache miss
1. Software event, generated by kernel, eg. context switch, tick
1. Tracepoint event, triggered by static tracepoint in kernel, eg. counter for slab 分配器

> perf stat
provide overview for program running detail

CPU bound 型 与 IO bound 型，调优不同

### Others
1. CPU affinity
CPU亲缘性 (soft and hard)
Cause: most popular architecture for server is SMP
and every CPU has its own cache (L1, L2)
TO: avoid L1/L2 cache invalid(失效) caused by context switch
