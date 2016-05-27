
[TOC]


# 文档

1. [Mysql内存碎片整理](http://www.365mini.com/page/mysql-optimize-table.htm)
2. [查看Mysql索引页碎片](http://www.2cto.com/database/201203/124967.html)
3. [Mysql存储过程自动维护分区方法](http://blog.chinaunix.net/uid-24086995-id-127389.html)





# 大数据量基础

**基础概念：**
1. 大数据量，数据量很大，一般在千万级或者亿级别
2. 大数据： 对大数据量进行分析和挖掘，通常设计数据仓库，数据挖掘，人工智能方向

	因此大数据量问题的本质是：要操作的数据量基础太大，而引发问题：
	- 慢
	- 多次操作的叠加导致数据库崩溃


**如何处理**

**分字诀**
1.  用和不用分开，常用和不常用分开
2.  分区 分库 分表
3.  对于文件存放的数据：拆文件
4.   考虑分批处理


**合理使用缓存**
**数据库优化**
1. 合理设计数据库结构
2. 合理构建索引
3. 数据库集群

**优化算法**
1. 优化操作数据的算法
2. 优化sql
3. 考虑使用临时表或者中间表

**合理使用NoSql**
MongoDB, Redis, HBase等等

**使用大数据来解决问题**

# MySql分区

**分区是什么？**
	将一个表分成多个区块进行操作和存储，但是对应用是透明的。逻辑上一个表，物理上多个分区，每个分区都是一个独立的对象，可以进行独立的处理。

**分区的优点**
1.  逻辑上进行数据分割，分割数据能有不同的物理文件路径
2. 可以存储更多的数据，突破单个文件的最大限制
3. 提升性能
4. 删除分区的时候可以快速删除数据(*例如想快速删除多年前的数据*)
5.  通过物理上跨多个磁盘来分散数据结构，提高磁盘IO性能
6. 聚合函数(`sum`,`avg`) 等可以并行处理
7. 对分区进行单独的备份和恢复

**支持分区的引擎**
	MyISAM , InnoDb  都支持分区， 不支持MERGE 和 CSV等来创建分区。 同一个分区表中的所有
	分区必须是**同一个存储引擎**

**使用分区**

1. 是否支持分区
	```sql
		-- 新版本
		show plugins;
		-- 如果partition是Active表示支持分区
		-- 老版本
		show variables like '%partition%'
	```
2. 分区类型 :
	- RANGE分区
	- LIST 基于给定的连续区间的`列值`，把多行分配给分区
	- RANGE 类似于RANGE, LIST 是列值匹配一个离散值集合中的某个值来进行选择
	- HASH 用户自定义的返回值
	- KEY分区： 类似于HASH， 由Mysql服务器提供HASH函数

3.  tips:

	- 如果存在pk,uk 分区的列必须从pk或者uk中去取
	- 没有pk 和 uk 则可以随便指定一个列
	- 5.5 之前RANGE LIST HASH要求分区键必须是int， 5.5之后支持非整形的RANGE和LIST

4. 分区命名
	分区的名字基础上遵循其他 Mysql 标识符应当遵循的原则， 例如用于表和数据库名字的标识符。但是应当注意，分区的名字是**不区分大小写**的。
	无论使用何种分区，分区在创建时就自动的顺序编号，从0开始记录

**创建分区**

```sql
	CREATE TABLE tbl_users (
		uuid INT NOT NULL,
		customerId VARCHAR(20),
		pwd VARCHAR(20),
		showName VARCHAR(100),
		trueName VARCHAR(100),
		registerTime VARCHAR(100)
	)
	--使用uuid进行分区
	PARTITION BY RANGE (uuid) (
		PARTITION p0 VALUES LESS THAN (5),
		PARTITION p1 VALUES LESS THAN (10),
		PARTITION p2 VALUES LESS THAN (15),
		PARTITION p3 VALUES LESS THAN MAXVALUE
	);

	--准备测试数据
	insert into tbl_users values(1,'test01','111','carl','A','');
	insert into tbl_users values(6,'test06','111','carl','B','');
	insert into tbl_users values(11,'test01','111','carl','C','');
	insert into tbl_users values(16,'test01','111','carl','D','');
	insert into tbl_users values(50,'test01','111','carl','E','');
	insert into tbl_users values(13,'test01','111','carl','F','');

	CREATE TABLE tbl_users7(
			uuid INT NOT NULL,
			customerId VARCHAR(20),
			pwd VARCHAR(20),
			showName VARCHAR(100),
			trueName VARCHAR(100),
			registerTime Date
		)
		PARTITION BY RANGE(YEAR(registerTime))
			SUBPARTITION BY HASH(TO_DAYS(registerTime))
		(
			PARTITION p0 VALUES LESS THAN (2008)(
			SUBPARTITION s0,
			SUBPARTITION s1
		),
		PARTITION p1 VALUES LESS THAN (2015)(
			SUBPARTITION s2,
			SUBPARTITION s3
		),
		PARTITION p2 VALUES LESS THAN MAXVALUE(
			SUBPARTITION s4,
			SUBPARTITION s5
		)
		);
```



 检测分区是否创建成功：
（1）到存放数据的地方查看文件，路经配置在/usr/bin/mysql_config里面的ldata。
（2）可以通过使用形如：`SELECT * FROM information_schema.partitions WHERE table_schema='carl' AND table_name='tbl_users' \G;` 的语句来查看分区信息
（3）可以通过形如`select * from tbl_users partition(p0);`的语句来查看分区上的数据
（4）可以使用形如`explain partitions select * from tbl_users where uuid=2;`的语句来
查看MySQL会操作的分区


#google_report demo
-  1.**准备数据**

```sql
DROP TABLE IF EXISTS `google_report`;
CREATE TABLE `google_report` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `account_id` bigint(20) DEFAULT NULL,
  `channel_code` varchar(20) DEFAULT NULL,
  `channel_cost_currency` varchar(10) DEFAULT NULL,
  `check_in_date` datetime DEFAULT NULL,
  `clicks` bigint(20) DEFAULT NULL,
  `cost_used` double DEFAULT NULL,
  `country` varchar(50) DEFAULT NULL,
  `date` datetime DEFAULT NULL,
  `device_type` varchar(10) DEFAULT NULL,
  `eligible_impressions` bigint(20) DEFAULT NULL,
  `final_cost` decimal(19,6) DEFAULT NULL,
  `final_cost_currency` varchar(10) DEFAULT NULL,
  `hotel_name` varchar(50) DEFAULT NULL,
  `impressions` bigint(20) DEFAULT NULL,
  `length_of_stay` tinyint(4) DEFAULT NULL,
  `passport` varchar(50) DEFAULT NULL,
  `position` double DEFAULT NULL,
  `price_bucket` int(11) DEFAULT NULL,
  `price_bucket_rank` int(11) DEFAULT NULL,
  `property` varchar(255) DEFAULT NULL,
  `provider_hotel_code` varchar(255) DEFAULT NULL,
  `rank` int(11) DEFAULT NULL,
  `room_bundle` varchar(10) DEFAULT NULL,
  `slot` varchar(10) DEFAULT NULL,
  `slot_position` double DEFAULT NULL,
  PRIMARY KEY (`id`,`date`),
  KEY `IDX_GOOGLE_REPORT_DATE` (`date`),
  KEY `IDX_GOOGLE_REPORT_id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
使用python测试插入数据
```python
__author__ = 'carl'
# -*- coding: UTF-8 -*-

import MySQLdb
import ConfigParser
import itertools
import logging
import time
import traceback
import random
import datetime

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                    datefmt='%a, %d %b %Y %H:%M:%S')


def create_conn():
    conn = MySQLdb.connect(host="localhost", user="root", passwd="", db="arch", charset="utf8")
    return conn


def gen_params(fileds):
    columns = fileds.keys()
    _params = [fileds[key] for key in columns]
    return _params


def gen_sql(fileds, tablename):
    columns = fileds.keys()
    _prefix = "".join(['INSERT INTO `', tablename, '`'])
    _fields = ",".join(["".join(['`', column, '`']) for column in columns])
    _values = ",".join(["%s" for i in range(len(columns))])
    _sql = "".join([_prefix, "(", _fields, ") VALUES (", _values, ")"])
    return _sql


def get_account_id(i):
    return i % 20;


def get_channel_code(index):
    account_id = get_account_id(index)
    if account_id < 15:
        return "google"
    else:
        return "trivago"


def init_google_report(total, conn=None):
    batch_size = 10000
    factor = total / batch_size
    remain = total % batch_size
    count = 0
    for i in range(factor):
        list = [{
                    "account_id": get_account_id(index),
                    "channel_code": get_channel_code(index),
                    "channel_cost_currency": get_channel_cost_currency(index),
                    "check_in_date": get_date(index, total),
                    "clicks": random.randint(1, 10),
                    "cost_used": random.randint(1, 10),
                    "country": get_country(index),
                    "date": get_date(index, total),
                    "device_type": get_device_type(index),
                    "eligible_impressions": random.randint(10, 100),
                    "final_cost": round(random.uniform(10, 100), 1),
                    "final_cost_currency": get_channel_cost_currency(index),
                    "hotel_name": get_hotel_name(index),
                    "impressions": random.randint(10, 100),
                    "length_of_stay": random.randint(1, 7),
                    "passport": get_passport(index),
                    "position": random.randint(1, 4),
                    "price_bucket": random.randint(1, 10),
                    "price_bucket_rank": random.randint(1, 10),
                    "property": random.randint(1, 10),
                    "provider_hotel_code": '',
                    "rank": random.randint(1, 10),
                    "room_bundle": random.randint(1, 5),
                    "slot": random.randint(1, 5),
                    "slot_position": random.randint(1, 5)
                } for index in range(count, count + batch_size, 1)]

        # 将list一次性插入数据库
        sql = gen_sql(list[0], "google_report")
        ret = []
        logging.info("now insert sql is : %s" % sql)
        for item in list:
            ret.append(gen_params(item))
        cursor = conn.cursor()
        cursor.executemany(sql, ret)
        close(cursor)
        conn.commit()
        count = count + batch_size
        logging.info("insert %d hotels" % count)


def get_channel_cost_currency(index):
    account_id = get_account_id(index)
    if (account_id > 10):
        return "USD"
    else:
        return "EUR"


def get_hotel_name(index):
    name = index % 500 + 1
    return "hotel%d" % name


def get_passport(index):
    name = index % 500 + 1
    return "passport%d" % name


COUNTRYIES = ["CA", "UK", "US", "HK", "TW", "SH"]


def get_country(index):
    length = COUNTRYIES.__len__()
    return COUNTRYIES[index % length]


def get_device_type(index):
    account_id = get_account_id(index)
    if account_id % 3 == 0:
        return "mobile"
    else:
        return "pc"


def get_date(index, total):
    f = total / 365
    r = index / f
    start = get_time_stamp("2015-01-01")
    end = start + (DAYS_INTERVAL * r)
    return get_time_str(end)


def get_time_str(time_stamp):
    time_array = time.localtime(time_stamp)
    other_style_time = time.strftime(TIME_TYPE, time_array)
    return other_style_time


def get_time_stamp(s):
    time_array = time.strptime(s, TIME_TYPE)
    return int(time.mktime(time_array))


DAYS_INTERVAL = 86400
TIME_TYPE = "%Y-%m-%d"


def close(db):
    if db is not None:
        db.close()


if __name__ == '__main__':
    conn = None
    try:
        conn = create_conn()
        total = 1000000
        init_google_report(total, conn)
        close(conn)
    except Exception, e:
        close(conn)
        print e


```

- **2.使用Range分区创建表**
```sql
DROP TABLE IF EXISTS `google_report_copy`;
CREATE TABLE `google_report_copy` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `account_id` bigint(20) DEFAULT NULL,
  `channel_code` varchar(20) DEFAULT NULL,
  `channel_cost_currency` varchar(10) DEFAULT NULL,
  `check_in_date` datetime DEFAULT NULL,
  `clicks` bigint(20) DEFAULT NULL,
  `cost_used` double DEFAULT NULL,
  `country` varchar(50) DEFAULT NULL,
  `date` datetime DEFAULT NULL,
  `device_type` varchar(10) DEFAULT NULL,
  `eligible_impressions` bigint(20) DEFAULT NULL,
  `final_cost` decimal(19,6) DEFAULT NULL,
  `final_cost_currency` varchar(10) DEFAULT NULL,
  `hotel_name` varchar(50) DEFAULT NULL,
  `impressions` bigint(20) DEFAULT NULL,
  `length_of_stay` tinyint(4) DEFAULT NULL,
  `passport` varchar(50) DEFAULT NULL,
  `position` double DEFAULT NULL,
  `price_bucket` int(11) DEFAULT NULL,
  `price_bucket_rank` int(11) DEFAULT NULL,
  `property` varchar(255) DEFAULT NULL,
  `provider_hotel_code` varchar(255) DEFAULT NULL,
  `rank` int(11) DEFAULT NULL,
  `room_bundle` varchar(10) DEFAULT NULL,
  `slot` varchar(10) DEFAULT NULL,
  `slot_position` double DEFAULT NULL,
  PRIMARY KEY (`id`,`date`),
  KEY `IDX_GOOGLE_REPORT_DATE` (`date`),
  KEY `IDX_GOOGLE_REPORT_id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE(TO_DAYS (`date`))
(
PARTITION p20150101 VALUES LESS THAN (TO_DAYS('2015-01-01')),
PARTITION p20150201 VALUES LESS THAN (TO_DAYS('2015-02-01')),
PARTITION p20150301 VALUES LESS THAN (TO_DAYS('2015-03-01')),
PARTITION p20150401 VALUES LESS THAN (TO_DAYS('2015-04-01')),
PARTITION p20150501 VALUES LESS THAN (TO_DAYS('2015-05-01')),
PARTITION p20150601 VALUES LESS THAN (TO_DAYS('2015-06-01')),
PARTITION p20150701 VALUES LESS THAN (TO_DAYS('2015-07-01')),
PARTITION p20150801 VALUES LESS THAN (TO_DAYS('2015-08-01')),
PARTITION p20150901 VALUES LESS THAN (TO_DAYS('2015-09-01')),
PARTITION p20151001 VALUES LESS THAN (TO_DAYS('2015-10-01')),
PARTITION p20151101 VALUES LESS THAN (TO_DAYS('2015-11-01')),
PARTITION p20151201 VALUES LESS THAN (TO_DAYS('2015-12-01')),
PARTITION p20160101 VALUES LESS THAN (TO_DAYS('2016-01-01')),
PARTITION p20160201 VALUES LESS THAN (TO_DAYS('2016-02-01'))
);
```


2. **测试性能**

对一个月的数据做聚合，刚好在一个partition中
```
explain SELECT SUM(t.clicks),SUM(t.final_cost),sum(rank),`date` FROM google_report_copy t WHERE `date` >= '2015-03-01' and `date` < '2015-04-01' group by `date`;

explain SELECT SUM(t.clicks),SUM(t.final_cost),sum(rank), `date` FROM google_report t WHERE `date` >= '2015-03-01' and `date` < '2015-04-01' group by `date`;

```

对一个月的数据做聚合，刚好在一个partition中
```
explain SELECT SUM(t.clicks),SUM(t.final_cost),sum(rank),`date` FROM google_report_copy t WHERE `date` >= '2015-03-01' and `date` < '2015-06-05' group by `date`;

explain SELECT SUM(t.clicks),SUM(t.final_cost),sum(rank), `date` FROM google_report t WHERE `date` >= '2015-03-01' and `date` < '2015-06-05' group by `date`;

```

# 分库分表

1. **为什么要分库分表**
数据库的复制能解决访问问题,并不能解决大规模的并发写入问题,由于无法进行分布式部署,而一台服务器的资源(CPU、磁盘、内存、IO等)是有限的,最终数据库所能承载的数据量、数据处理能力都将遭遇瓶颈。
要解决这个问题就要考虑对数据库进行分库分表了,它有如下好处:
- 解决磁盘系统最大文件限制,比如常见的有:
`FAT16`(最大分区2GB,最大文件2GB)
`FAT32`(最大分区32GB,最大容量2TB,最大文件32G)
`NTFS`(最大分区2TB,最大容量,最大文件2TB)
`EXT3`(最大文件大小: 2TB,最大文件极限: 仅受文件系统大小限制,最大分区/
 文件系统大小: 4TB,最大文件名长度: 255 字符)
 
- 减少增量数据写入时的锁对查询的影响,减少长时间查询造成的表锁,影响写入
操作等锁竞争的情况,节省排队的时间开支,增加呑吐量。

- 由于单表数量下降,常见的查询操作由于减少了需要扫描的记录,使得单表单次
查询所需的检索行数变少,减少了磁盘IO,时延变短。

2. **什么是分库**
分库又叫垂直切分,就是把原本**存储于一个库的表拆分存储到多个库上**,通常是将
表按照**功能模块、关系密切程度**划分出来,部署到不同的库上。
如果数据库是因为表太多而造成海量数据,并且项目的各项业务逻辑划分清晰、低
耦合,那么规则简单明了、容易实施的首选就是分库。
分库的优点是:实现简单,库与库之间界限分明,便于维护,缺点是不利于频繁跨
库操作,单表数据量大的问题解决不了。
3. **什么是分表**
分表又叫水平切分,是按照一定的**业务规则或逻辑,将一个表的数据拆分成多份**,
分别存储在多个表结构一样的表中,这多个表可以存在一到多个库中。
分表又分成垂直分表和水平分表:
**垂直分表**:将本来可以在同一个表的内容,人为划分为多个表。(所谓的本来,是
指按照关系型数据库的第三范式要求,是应该在同一个表的。)
**水平分表**,也被称为数据分片:是把一个表复制成同样表结构的不同表,然后把数
据按照一定的规则划分,分别存储到这些表中,从而保证单表的容量不会太大,提升性
能;当然这些结构一样的表,可以放在一个或多个数据库中。
**分表的优点是**:能解决分库的不足点,但是缺点是实现起来比较复杂,特别是分表
规则的划分,程序的编写,以及后期的数据库拆分移植维护。
一般都是先分库再分表,两者结合使用,取长补短,这样能发挥扩展的最大优势,
但是缺点是架构很大,很复杂,应用程序的编写也比较复杂。
4. **思路**
基本的思路就是分析业务功能,以及表间的聚合关系,把关系紧密的表放在一起。
分库的粒度指的是在做切分时允许几级的关联表放在一起,这个问题对应用程序实
现有着很大的影响。关联打断的越多,则受影响的join操作越多,应用程序为此做出的妥
协就越大,但单表的路由会越简单,与业务的关联性会越小,就越容易使用统一机制处
理。
实际的粒度掌控需要结合“业务紧密程度”和“表的数据量”两个因素综合考虑,
一般来说:若划归到一起的表关系紧密,且数据量并不大,增速也非常缓慢,则适宜放在
一起,不需要再进行水平切分;若划归到一起的表的数据量巨大且增速迅猛,则势必要在
分库的基础上再进行分表,这就意味着原单一的库还可能会被拆分成多个库,这会导致更
多的复杂性,一开始最好就要考虑进去。
5. **如何分表**
 - 对于垂直分表,通常是按照业务功能的使用频次,把主要的、热门的字段放在一起做为主要表;然后把不常用的,按照各自的业务属性进行聚集,拆分到不同的次要表中;主要表和次要表的关系一般都是一对一的。
- 对于水平分表,通常是按照具体的业务规则和数据的格式,选择能够把数据进行合理拆分的业务数据做为拆分标准,以此来对数据进行拆分。
- 常见的一些拆分方式:按业务属性、按时间、按区间、Hash、按数据的活跃度、按数据量等,不管采用什么方式,都要结合具体的业务场景进行分析和考量。
当然这个过程中要考虑很多问题,比如:预估的数据量大小、数据量增长速度、分表的数量多少、在多表中数据的均衡、多表负载的均衡、扩容、访问表的导航信息等
分表的表现又分成:单库单表、单库多表、多库多表几种。
6. **分库分表后的问题**
 - 分布式事务的问题,数据的完整性和一致性问题
 - 数据操作的维度问题
例如保存交易记录,是按照用户的纬度分表保存,还是按照产品的纬度来分表保存。
 - 跨库联合查询的问题,可能需要两次查询
- 跨节点的count、order by、group by以及聚合函数问题,可能需要分别在各个节点上得到结果,然后在应用程序端进行合并
- 额外的数据管理负担,如:访问数据表的导航定位
- 额外的数据运算压力,如:需要在多个节点执行,然后再合并计算
- 程序复杂度上升
- 后期维护难度上升,包括:程序的维护和升级、数据库的扩容、数据的迁移等
7. **水平分表的实现-1**
水平分表的实现面临一系列问题:切分策略、库节点路由、表路由、全局主键生成、跨节点排序
/分组/分页/表关联等操作、多数据源事务处理、数据库扩容等。
**部分相关开源产品一览(排名不分先后)**
- MySQL Fabric:官方产品,非代理方式,目前不太稳定,性能也不够好,但很有前景,综合了HA和水平
分表的功能,是未来的首选。
- Atlas:360开源,代理方式,基于MySQL-Proxy二次开发的,主要支持两个特性:分表和读写分离,但是
分表的话只支持单库多表,事实上是不支持分布式分表的
- Cobar:阿里开源,代理方式,支持分布式分表,但是不支持单库分多表,不支持读写分离,事务支持也
比较麻烦
- TDDL (Taobao Distributed Data Layer ):阿里部分开源,非代理方式,提供分库分表对应用的透明
化,实现异构数据库之间的数据复制,具有主备,读写分离,动态数据库配置等功能。但复杂度相对较
高,公布的文档少,核心部分不开源,还需要依赖Diamond
- MySQL Proxy:官方提供,基于MySQL协议接口,主要提供负载平衡,读写分离,failover等,但性能较
差,不支持大数据量的分库分表
- Amoeba:支持分数据库实例,每个数据相同的表,不支持事务;类似MySQL Proxy,相对更简单
- Hibernate Shards:支持分数据库实例,比较复杂,需事先规划数据规模,对HQL的支持非常有限
- mybatis shardbatis:主要通过插件机制来实现分表,但是插件机制控制不到多数据源的连接;离开插
件层又失去了对sql进行集中解析和路由的机会
8. **水平分表的实现-2**
n 现状——靠天靠地,不如靠自己
n 基本的实现思路
- **解析路由**:根据业务功能指定;根据SQL解析等,不管采用何种方式,要获得需要访问的数据源,以及分别要访问的表
- **分别在数据源和表上去执行功能**
- 如果涉及到返回结果集的话,就需要做结果集的合并,并按照需要进行二次处理,比如:排序、分页等
- 如果需要事务的话,就得考虑是使用分布式事务,还是自行实现两阶段提交,或者采用补偿性业务处理的方式等
- **可实现的层面**
a: DAO层
b: Spring数据访问封装层,介于DAO与JDBC之间
c: JDBC驱动层
d: 介于应用服务器与数据库之间的代理服务器


