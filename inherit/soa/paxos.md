# Paxos

分布式一致性协议，世界上唯一的分布式算法
[TOC]

## 一、问题的引入
**问题**

**1. Paxos究竟在解决什么问题？**

Paxos用来确定一个不可变变量的取值:

		取值可以是任意的二进制数据
		一旦确定将不可更改，并且可以被获取到（这就是所谓的不可变性，可读取性）



**2.  Paxos如何在分布式存储系统中应用？**

		数据本身是可变的，并且采用多个副本进行存储
		但是对多个副本的更新序列【op1,op2,...,opn】是相同的，不变的
		可以用Paxos来确定不可变变量opi的取值（即第i个操作是什么）
		每确定完opi之后，让各个数据副本执行opi，依次类推。

google的Chubby，Megastore和Spanner都采用了Paxos来对数据副本的更新序列来达成一致


## 二、思路的演变

### 2.1 系统的介绍

我们的目标是设计一个系统，来存储名称为var的变量

	1.系统内部由多个Acceptor组成，负责存储和管理var变量。
	2.外部有多个proposer机器任意并发调用API，向系统提交不同的var取值，var的取值可以是任意的二进制数据
	3.系统对外部的API接口为
	propose(var, V) => <ok, f> or <error>


 系统需要保证var的取值满足一致性

	1. 如果var的取值没有确定，则var的取值设置为null
	2. 一旦var的取值被确定，则不可以更改。并却可以一直获取到这个值。


系统需要满足容错的特性：

	1. 可以容忍任意proposer机器出现故障。
	2. 可以容忍少数Acceptor故障（半数以下）


以下问题暂时不考虑：
	1. 网络分化
	2. acceptor故障会丢失var的信息


### 2.2 系统难点

1. 管理多个Proposer的并发执行
2. 保证var变量的不可变性
3. 容忍任意的Proposer机器出现故障
4. 容忍半数的Acceptor机器出现故障


### 2.3 方案一：互斥锁，单Acceptor

使用类似于互斥锁的机制来管理并发的proposer的执行，实现如下：

**Acceptor设计如下：**

	1.保存变量var和一个互斥锁lock
	2.prepare():加互斥锁，给予var的互斥访问权，并且返回var的当前取值
	3.accept(var, V):如果已经加锁，且没有取值则设置为var为V，并且释放锁


**propose(var, V)的二阶段实现：**

**第一阶段：**
		1.通过Acceptor获取互斥锁访问权和当前var的取值
		2.如果获取到，进入第二阶段，否则，返回error，锁已经被占用

**第二阶段：** 根据当前var的取值f选择：

	1. 如果f的值为null，表示没有取值，通过Acceptor的accept(var, V) 提交数据V。
	2. 如果f不为空，直接释放访问权，返回<ok, f>

**总结：**
	通过互斥锁来简单保证var取值的一致性

**问题：**

		假如proposer在释放互斥访问权之前发生故障，会导致系统陷入死锁，因此不能保证容错性第一点，为了解决这个问题，考虑方案二


### 2.4 方案二：引入抢占式访问权。

**基本思路：**

**1. 引入抢占式访问权：**

	acceptor可以让某个proposer获取到的访问权失效，不再接受它的访问
2. proposer向Acceptor申请访问权的时候指定编号epoch（越大的epoch越新），获取到访问权之后，才能向acceptor提交取值

3.Acceptor采用喜新厌旧的原则

	一旦受到更大的epoch申请，马上让旧的epoch的访问失效，不再接受他们的提交的取值
4. 新epoch可以抢占旧的epoch，让旧epoch失效，旧epoch的proposer将无法运行，新epoch的proposer将开始运行。
5. 为了保证一致性，不同的epoch的proposer采用了后者认同前者的原则

		在肯定的那个旧的epoch无法生成确定性取值时，新的epoch会提交自己的value。不会冲突
		一旦旧epoch形成确定性取值，新的epoch肯定可以获取到此取值，并且会认同此取值，不会破坏。


**Acceptor实现：**

		1. Acceptor保存的状态有当前var的取值<accepted_epoch, accepted_value>
		最新发放访问权的epoch(lastest_prepared_epoch)

		2. prepare(epoch) : 只接收比latest_prepare_epoch更大的epoch，并给予访问权，记录当前latest_prepared_epoch = epoch：返回当前var的取值。
		3. accept(var, prepared_epoch, V)：验证lastest_prepared_epoch = prepared_epoch，并且设置var的取值<accepted_epoch, accepted_value> = <prepared_epoch, v>


**propose(var, V)的二阶段实现：**


**第一阶段：** 获取epoch轮次的访问权和当前var的取值

		1.简单选择当前时间戳为epoch，通过accpetor的prepare(epoch)方法来获取epoch轮次的访问权和当前var的取值
		2.如果不能获取，返回<error>


**第二阶段：** 采用后者认同前者的原则执行

		1.在肯定旧epoch无法生成确定性取值时候，新的epoch会提交自己的value，不会冲突
		2.一旦旧epoch形成确定性取值，新的epoch肯定可以获取到这个值，并且不会破坏

		3.如果var的取值为null，则可以肯定旧epoch无法生成确定性取值，则通过accept(var , epoch ,V) 提交数据V。成功后返回<ok, V>	，如果accept失败，返回error，此时，被新的epoch抢占或者acceptor故障。

		4.如果var的取值非空，那么知道此时的值已经是确定性取值，此时应该认同且不再更改，返回<ok, accepted_value>



### 2.5 方案三- paxos

Acceptor的实现保持不变，仍采用喜新厌旧的思路
paxos采用少数服从多数的思路
一旦某个epoch的取值f被半数以上accept接受，则认为此var取值被确定为f，不再更改


