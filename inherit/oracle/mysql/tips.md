


[TOC]




# Mysql 的 基础类型

## 整数类型

整数问题比较简单，只需要判断范围尽可能选择小的范围既可
1. **bigint**
从 -2^63 (-9223372036854775808) 到 2^63-1 (9223372036854775807) 的整型数据（所有数字）。存储大小为 8 个字节。
>P.S. bigint已经有长度了，在mysql建表中的length，只是用于显示的位数

2. **int**
从 -2^31 (-2,147,483,648) 到 2^31 – 1 (2,147,483,647) 的整型数据（所有数字）。存储大小为 4 个字节。int 的 SQL-92 同义字为 integer。
3. **smallint**
从 `-2^15 (-32,768)` 到 `2^15 – 1 (32,767)` 的整型数据。存储大小为 2 个字节。
4. **tinyint**
从 0 到 255 的整型数据。存储大小为 1 字节。
5. **注释**
在支持整数值的地方支持 bigint 数据类型。但是，bigint 用于某些特殊的情况，当整数值超过 int 数据类型支持的范围时，就可以采用 bigint。在 SQL Server 中，int 数据类型是主要的整数数据类型。
在数据类型优先次序表中，bigint 位于 smallmoney 和 int 之间。
只有当参数表达式是 bigint 数据类型时，函数才返回 bigint。SQL Server 不会自动将其它整数数据类型（tinyint、smallint 和 int）提升为 bigint。
int(M) 在 integer 数据类型中，M 表示最大显示宽度。在 int(M) 中，M 的值跟 int(M) 所占多少存储空间并无任何关系。和数字位数也无关系 int(3)、int(4)、int(8) 在磁盘上都是占用 4 btyes 的存储空间。

## 浮点数
MySQL浮点型和定点型可以用类型名称后加（M，D）来表示，M表示该值的总共长度，D表示小数点后面的长度，M和D又称为精度和标度，如float(7,4)的可显示为-999.9999，MySQL保存值时进行四舍五入，如果插入999.00009，则结果为999.0001。
(1) FLOAT和DOUBLE在不指定精度时，默认会按照实际的精度来显示
(2) 而DECIMAL在不指定精度时，默认整数为10，小数为0。


# 查询编码字符集 

1. 查询字段:

```sql
    -- 查询server的
    select table_name, table_collation from information_schema.tables where table_schema = database();
    
    -- 查询字段的
    select *
    from information_schema.columns
    where table_schema = database() 
      and collation_name is not null
      and collation_name not like 'utf8%';
```

下面要修改字符集:
```sql
    -- 修改某一个字段的
    ALTER TABLE account CHANGE notify_email notify_email VARCHAR(100) CHARACTER SET utf8 COLLATE utf8_general_ci;
```



# mysql 锁（实战检测）

myIsam 和 innodb都支持表锁，可以通过`LOCK TABLES USER WRITE;`加锁，通过`UNLOCK TABLES;`解锁

可以通过命令`SHOW STATUS LIKE '%lock%';`查看锁状态

```sql
| Com_lock_tables               | 0      |
| Com_unlock_tables             | 0      |
| Innodb_row_lock_current_waits | 0      |
| Innodb_row_lock_time          | 108733 |
| Innodb_row_lock_time_avg      | 36244  |
| Innodb_row_lock_time_max      | 50873  |
| Innodb_row_lock_waits         | 3      |
| Key_blocks_not_flushed        | 0      |
| Key_blocks_unused             | 214340 |
| Key_blocks_used               | 2      |
| Qcache_free_blocks            | 1      |
| Qcache_total_blocks           | 4      |
| Table_locks_immediate         | 226    |
| Table_locks_waited            | 3
```
1. Table_locks_waited 表示被表锁锁住的总次数
2. Innodb_row_lock_current_waits当前行锁等待个数
3. Innodb_row_lock_waits 表示被行锁锁住的总次数


>1. `set autocommit=0` 取消自动提交，以下都假设取消自动提交且事务级别是3 repeatable read
>2. 一个session中可以看到当前session未commit的数据，看不到其他session提交或者未提交的数据
>3. 一个session中除非当前调用commit方法，才能看到其他session已经提交的数据

下一个话题:<span class="red-bold">悲观锁和乐观锁</span>

1. **并发控制机制**
    最常用的处理多用户并发访问的方法是加锁。当一个用户锁住数据库中的某个对象时，其他用户就不能再访问该对象。加锁对并发访问的影响体现在锁的粒度上。比如，放在一个表上的锁限制对整个表的并发访问；放在数据页上的锁限制了对整个数据页的访问；放在行上的锁只限制对该行的并发访问。可见行锁粒度最小，并发访问最好，页锁粒度最大，表锁介于2者之间。

	-<span class="red-bold">悲观锁</span>：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。[1]      悲观锁假定其他用户企图访问或者改变你正在访问、更改的对象的概率是很高的，因此在悲观锁的环境中，在你开始改变此对象之前就将该对象锁住，并且直到你提交了所作的更改之后才释放锁。悲观的缺陷是不论是页锁还是行锁，加锁的时间可能会很长，这样可能会长时间的限制其他用户的访问，也就是说悲观锁的并发访问性不好。

	-<span class="red-bold">乐观锁</span>：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。[1] 乐观锁不能解决脏读的问题。    乐观锁则认为其他用户企图改变你正在更改的对象的概率是很小的，因此乐观锁直到你准备提交所作的更改时才将对象锁住，当你读取以及改变该对象时并不加锁。可见乐观锁加锁的时间要比悲观锁短，乐观锁可以用较大的锁粒度获得较好的并发访问性能。但是如果第二个用户恰好在第一个用户提交更改之前读取了该对象，那么当他完成了自己的更改进行提交时，数据库就会发现该对象已经变化了，这样，第二个用户不得不重新读取该对象并作出更改。这说明在乐观锁环境中，会增加并发用户读取对象的次数。
    *比如redis中的Watch机制*就是一种乐观锁

	从数据库厂商的角度看，使用`乐观的`页锁是比较好的，尤其在影响很多行的批量操作中可以放比较少的锁，从而降低对资源的需求提高数据库的性能。再考虑`聚集索引`。在数据库中记录是按照聚集索引的物理顺序存放的。如果使用页锁，当两个用户同时访问更改位于同一数据页上的相邻两行时，其中一个用户必须等待另一个用户释放锁，这会明显地降低系统的性能。interbase和大多数关系数据库一样，采用的是乐观锁，而且读锁是共享的，写锁是排他的。可以在一个读锁上再放置读锁，但不能再放置写锁；你不能在写锁上再放置任何锁。锁是目前解决多用户并发访问的有效手段。

2. 乐观锁应用
   - 使用自增长的整数表示数据版本号。更新时检查版本号是否一致，比如数据库中数据版本为6，更新提交时version=6+1,使用该version值(=7)与数据库version+1(=7)作比较，如果相等，则可以更新，如果不等则有可能其他程序已更新该记录，所以返回错误。

   - 使用时间戳来实现.

   - 注：对于以上两种方式,`Hibernate`自带实现方式：在使用乐观锁的字段前加annotation: @Version, Hibernate在更新时自动校验该字段。

3. 悲观锁应用
    需要使用数据库的锁机制，比如SQL SERVER 的TABLOCKX（排它表锁） 此选项被选中时，SQL  Server  将在整个表上置排它锁直至该命令或事务结束。这将防止其他进程读取或修改表中的数据。

    SqlServer中使用
    ```sql
    Begin Tran
    select top 1 @TrainNo=T_NO
             from Train_ticket   with (UPDLOCK)   where S_Flag=0

          update Train_ticket
             set T_Name=user,
                 T_Time=getdate(),
                 S_Flag=1
             where T_NO=@TrainNo
    commit
    ```
    我们在查询的时候使用了with (UPDLOCK)选项,在查询记录的时候我们就对记录加上了更新锁,表示我们即将对此记录进行更新. 注意更新锁和共享锁是不冲突的,也就是其他用户还可以查询此表的内容,但是和更新锁和排它锁是冲突的.所以其他的更新用户就会阻塞.




# mysql 的事务隔离级别

说道事务隔离级别，必须先说事务中经常出现的问题:
1. **脏读**：脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
例如：
　　张三的工资为5000,事务A中把他的工资改为8000,但事务A尚未提交。
　　与此同时，
　　事务B正在读取张三的工资，读取到张三的工资为8000。
　　随后，
　　事务A发生异常，而回滚了事务。张三的工资又回滚为5000。
　　最后，
　　事务B读取到的张三工资为8000的数据即为脏数据，事务B做了一次脏读。

2. **不可重复读**：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
例如：
　　在事务A中，读取到张三的工资为5000，操作没有完成，事务还没提交。
　　与此同时，
　　事务B把张三的工资改为8000，并提交了事务。
　　随后，
　　在事务A中，再次读取张三的工资，此时工资变为8000。在一个事务中前后两次读取的结果并不致，导致了不可重复读。

3. **幻读**：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。
例如：
　　目前工资为5000的员工有10人，事务A读取所有工资为5000的人数为10人。
　　此时，
　　事务B插入一条工资也为5000的记录。
　　这是，事务A再次读取工资为5000的员工，记录为11人。此时产生了幻读。

4. **提醒**
不可重复读的重点是<span class="red-bold">修改</span>：
同样的条件，你读取过的数据，再次读取出来发现值不一样了
幻读的重点在于<span class="red-bold">新增或者删除</span>：
同样的条件，第 1 次和第 2 次读出来的记录数不一样


下面实战演示，mysql中如何出现幻读:

打开2个事务

```sql
	begin transaction;
    select count(*) from CARL_USER; --4
    --此时调用另一个事务的插入操作
    update  CARL_USER set name ='111'; --update操作会导致幻读
    --在去查询
    select count(*) from CARL_USER; --5 返回5,出现幻读
```

```sql
    -- 查看事务隔离等级
    SELECT @@tx_isolation;
	insert into CARL_USER(name) VALUES('11');
```

mysql 中的事务中分为以下几个隔离级别:


1. 未提交读（READ UNCOMMITTED）。另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据（脏读）。
2. 提交读（READ COMMITTED）。本事务读取到的是最新的数据（其他事务提交后的）。问题是，在同一个事务里，前后两次相同的SELECT会读到不同的结果（不重复读）。
3. 可重复读（REPEATABLE READ）。在同一个事务里，SELECT的结果是事务开始时时间点的状态，因此，同样的SELECT操作读到的结果会是一致的。但是，会有幻读现象（稍后解释）。
4. 串行化（SERIALIZABLE）。读操作会隐式获取共享锁，可以保证不同事务间的互斥。


# mysql 函数:
## `find_in_set`(包含递归查询)

直接看下面2个demo:
```sql
--其中bid_pos 是字符串1,2,4
SELECT * FROM bidding_pos_constraint WHERE id = 1 and  find_in_set(1,bid_pos);
--find_in_set(str,list)就是返回str在str_list中的索引位置
SELECT * FROM bidding_pos_constraint WHERE id = 1 and  find_in_set('1',bid_pos);
--好像改成字符串后也很好使
```

下面来观看**官网**的注释:

- FIND_IN_SET(str,strlist)

	Returns a value in the range of 1 to N if the string str is in the string list strlist **consisting** of N substrings. A string list is a string composed of substrings separated by “,” characters. If the first argument is a constant string and the second is a column of type SET, the FIND_IN_SET() function is optimized to use bit arithmetic. Returns 0 if str is not in strlist or if strlist is the empty string. Returns NULL if either argument is NULL. *This function does not work properly if the first argument contains a comma (“,”) character.*

	注意：当第一参数包含"`,`"号的时候不可用,可见这个分隔符很关键

- 性能问题:
	同样，个人感觉find_in_set也会带来性能问题, 对任意的索引使用函数是无法使用索引效果的

```sql
	explain SELECT * FROM account where find_in_set(id,'1,3,4');
	--此时应该使用后者
	explain SELECT * FROM account where id in (1,3,4);
	--但是有些需求适合这个函数,同样需要考虑性能问题
```

<span class="red-bold">实战</span>使用find_in_set完成递归查询:

```sql
--准备测试数据
DROP TABLE IF EXISTS `t_areainfo`;
CREATE TABLE `t_areainfo` (
 `id` int(11) NOT '0' AUTO_INCREMENT,
 `level` int(11) DEFAULT '0',
 `name` varchar(255) DEFAULT '0',
 `parentId` int(11) DEFAULT '0',
 `status` int(11) DEFAULT '0',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=65 DEFAULT CHARSET=utf8;

INSERT INTO `t_areainfo` VALUES ('1', '0', '中国', '0', '0');
INSERT INTO `t_areainfo` VALUES ('2', '0', '华北区', '1', '0');
INSERT INTO `t_areainfo` VALUES ('3', '0', '华南区', '1', '0');
INSERT INTO `t_areainfo` VALUES ('4', '0', '北京', '2', '0');
INSERT INTO `t_areainfo` VALUES ('5', '0', '海淀区', '4', '0');
INSERT INTO `t_areainfo` VALUES ('6', '0', '丰台区', '4', '0');
INSERT INTO `t_areainfo` VALUES ('7', '0', '朝阳区', '4', '0');
INSERT INTO `t_areainfo` VALUES ('8', '0', '北京XX区1', '4', '0');
INSERT INTO `t_areainfo` VALUES ('9', '0', '北京XX区2', '4', '0');
INSERT INTO `t_areainfo` VALUES ('10', '0', '北京XX区3', '4', '0');
INSERT INTO `t_areainfo` VALUES ('11', '0', '北京XX区4', '4', '0');
INSERT INTO `t_areainfo` VALUES ('12', '0', '北京XX区5', '4', '0');
INSERT INTO `t_areainfo` VALUES ('13', '0', '北京XX区6', '4', '0');
INSERT INTO `t_areainfo` VALUES ('14', '0', '北京XX区7', '4', '0');
INSERT INTO `t_areainfo` VALUES ('15', '0', '北京XX区8', '4', '0');
INSERT INTO `t_areainfo` VALUES ('16', '0', '北京XX区9', '4', '0');
INSERT INTO `t_areainfo` VALUES ('17', '0', '北京XX区10', '4', '0');
INSERT INTO `t_areainfo` VALUES ('18', '0', '北京XX区11', '4', '0');
INSERT INTO `t_areainfo` VALUES ('19', '0', '北京XX区12', '4', '0');
INSERT INTO `t_areainfo` VALUES ('20', '0', '北京XX区13', '4', '0');
INSERT INTO `t_areainfo` VALUES ('21', '0', '北京XX区14', '4', '0');
INSERT INTO `t_areainfo` VALUES ('22', '0', '北京XX区15', '4', '0');
INSERT INTO `t_areainfo` VALUES ('23', '0', '北京XX区16', '4', '0');
INSERT INTO `t_areainfo` VALUES ('24', '0', '北京XX区17', '4', '0');
INSERT INTO `t_areainfo` VALUES ('25', '0', '北京XX区18', '4', '0');
INSERT INTO `t_areainfo` VALUES ('26', '0', '北京XX区19', '4', '0');
INSERT INTO `t_areainfo` VALUES ('27', '0', '北京XX区1', '4', '0');
INSERT INTO `t_areainfo` VALUES ('28', '0', '北京XX区2', '4', '0');
INSERT INTO `t_areainfo` VALUES ('29', '0', '北京XX区3', '4', '0');
INSERT INTO `t_areainfo` VALUES ('30', '0', '北京XX区4', '4', '0');
INSERT INTO `t_areainfo` VALUES ('31', '0', '北京XX区5', '4', '0');
INSERT INTO `t_areainfo` VALUES ('32', '0', '北京XX区6', '4', '0');
INSERT INTO `t_areainfo` VALUES ('33', '0', '北京XX区7', '4', '0');
INSERT INTO `t_areainfo` VALUES ('34', '0', '北京XX区8', '4', '0');
INSERT INTO `t_areainfo` VALUES ('35', '0', '北京XX区9', '4', '0');
INSERT INTO `t_areainfo` VALUES ('36', '0', '北京XX区10', '4', '0');
INSERT INTO `t_areainfo` VALUES ('37', '0', '北京XX区11', '4', '0');
INSERT INTO `t_areainfo` VALUES ('38', '0', '北京XX区12', '4', '0');
INSERT INTO `t_areainfo` VALUES ('39', '0', '北京XX区13', '4', '0');
INSERT INTO `t_areainfo` VALUES ('40', '0', '北京XX区14', '4', '0');
INSERT INTO `t_areainfo` VALUES ('41', '0', '北京XX区15', '4', '0');
INSERT INTO `t_areainfo` VALUES ('42', '0', '北京XX区16', '4', '0');
INSERT INTO `t_areainfo` VALUES ('43', '0', '北京XX区17', '4', '0');
INSERT INTO `t_areainfo` VALUES ('44', '0', '北京XX区18', '4', '0');
INSERT INTO `t_areainfo` VALUES ('45', '0', '北京XX区19', '4', '0');
INSERT INTO `t_areainfo` VALUES ('46', '0', 'xx省1', '1', '0');
INSERT INTO `t_areainfo` VALUES ('47', '0', 'xx省2', '1', '0');
INSERT INTO `t_areainfo` VALUES ('48', '0', 'xx省3', '1', '0');
INSERT INTO `t_areainfo` VALUES ('49', '0', 'xx省4', '1', '0');
INSERT INTO `t_areainfo` VALUES ('50', '0', 'xx省5', '1', '0');
INSERT INTO `t_areainfo` VALUES ('51', '0', 'xx省6', '1', '0');
INSERT INTO `t_areainfo` VALUES ('52', '0', 'xx省7', '1', '0');
INSERT INTO `t_areainfo` VALUES ('53', '0', 'xx省8', '1', '0');
INSERT INTO `t_areainfo` VALUES ('54', '0', 'xx省9', '1', '0');
INSERT INTO `t_areainfo` VALUES ('55', '0', 'xx省10', '1', '0');
INSERT INTO `t_areainfo` VALUES ('56', '0', 'xx省11', '1', '0');
INSERT INTO `t_areainfo` VALUES ('57', '0', 'xx省12', '1', '0');
INSERT INTO `t_areainfo` VALUES ('58', '0', 'xx省13', '1', '0');
INSERT INTO `t_areainfo` VALUES ('59', '0', 'xx省14', '1', '0');
INSERT INTO `t_areainfo` VALUES ('60', '0', 'xx省15', '1', '0');
INSERT INTO `t_areainfo` VALUES ('61', '0', 'xx省16', '1', '0');
INSERT INTO `t_areainfo` VALUES ('62', '0', 'xx省17', '1', '0');
INSERT INTO `t_areainfo` VALUES ('63', '0', 'xx省18', '1', '0');
INSERT INTO `t_areainfo` VALUES ('64', '0', 'xx省19', '1', '0');

--创建自定义函数
DROP FUNCTION IF EXISTS queryChildrenAreaInfo;
CREATE FUNCTION `queryChildrenAreaInfo` (areaId INT)
RETURNS VARCHAR(4000)
BEGIN
DECLARE sTemp VARCHAR(4000);
DECLARE sTempChd VARCHAR(4000);

SET sTemp = '$';
SET sTempChd = cast(areaId as char);

WHILE sTempChd is not NULL DO
SET sTemp = CONCAT(sTemp,',',sTempChd);
SELECT group_concat(id) INTO sTempChd FROM t_areainfo where FIND_IN_SET(parentId,sTempChd)>0;
END WHILE;
return sTemp;
END;

--直接使用
select queryChildrenAreaInfo(1);
select * from t_areainfo where FIND_IN_SET(id, queryChildrenAreaInfo(1)); 
```







