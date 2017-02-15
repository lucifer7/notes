## 分布式系统 Unique ID 生成
### 1. Requirements
#### 1.1 保证生成的 ID 全局唯一
#### 1.2 今后数据在多个 Shards 之间迁移不会受到 ID 生成方式的限制
#### 1.3 生成的 ID 中最好能带上时间信息, 例如 ID 的前 k 位是 Timestamp, 这样能够直接通过对 ID 的前 k 位的排序来对数据按时间排序
#### 1.4 生成的 ID 最好不大于 64 bits
#### 1.5 生成 ID 的速度有要求. 例如, 在一个高吞吐量的场景中, 需要每秒生成几万个 ID (Twitter 最新的峰值到达了 143,199 Tweets/s, 也就是 10万+/秒)
#### 1.6 整个服务最好没有单点

### 2. Implementation
#### 2.1 Tweeter, Snowflake
[Project URL](https://github.com/twitter/snowflake)

##### 2.1.1 Generation
- 10 bits 的机器号, 在 ID 分配 Worker 启动的时候, 从一个 Zookeeper 集群获取 (保证所有的 Worker 不会有重复的机器号)
- 41 bits 的 Timestamp: 每次要生成一个新 ID 的时候, 都会获取一下当前的 Timestamp, 然后分两种情况生成 sequence number:
- 如果当前的 Timestamp 和前一个已生成 ID 的 Timestamp 相同 (在同一毫秒中), 就用前一个 ID 的 sequence number + 1 作为新的 sequence number (12 bits); 如果本毫秒内的所有 ID 用完, 等到下一毫秒继续 (这个等待过程中, 不能分配出新的 ID)
- 如果当前的 Timestamp 比前一个 ID 的 Timestamp 大, 随机生成一个初始 sequence number (12 bits) 作为本毫秒内的第一个 sequence number

#### 2.2 基于 Redis 的ID生成器
[Project URl](https://github.com/hengyunabc/redis-id-generator)

#### 2.2.1 Detail
using LUA script, generate on each node