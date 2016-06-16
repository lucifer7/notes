## Hierarchy Data Database
### 数据库中存储树形结构 Tree
一般比较普遍的就是四种方法：（具体见 SQL Anti-patterns这本书）
Adjacency List：每一条记录存parent_id
Path Enumerations：每一条记录存整个tree path经过的node枚举
Nested Sets：每一条记录存 nleft 和 nright
Closure Table：维护一个表，所有的tree path作为记录进行保存。

各种方法的常用操作代价见下图
TODO: add link to pic

作者：卢钧轶
链接：https://www.zhihu.com/question/20417447/answer/15078011
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

See Ref Book: SQL anti-patterns

More tree types:
Modified Preorder Tree Traversal：预排序遍历树

### Implement for trees
MySQL does not support recursive query
Oracle use START WITH, CONNECT BY
SQL Server use CTE

[Adjacency list vs. nested sets: MySQL](https://explainextended.com/2009/09/29/adjacency-list-vs-nested-sets-mysql/)
> nested sets defeats adjacency list on most query

#### 1.1 Modified Path Enumeration Tree
>width = 4, id string type
root 0000

select parent
>select * from tree where id =  id.substr(0, id.length - 4)

select ancestor

select tree map
>select * from tree where id like '0000%'
order by len(id), id

#### 1.2 Path Enumeration Tree
id, path(1/5/6/)


#### 1.3  Modified Preorder tree
> id, lno, rno

1. select parent
>select * from tree where lno = (:lno - 1)

1. select ancestor
>select * from tree where lno < (:lno) and rno > (:rno)

1. select children
>select * from tree where lno > (:lno) and rno < (:rno)

1. select tree map
 ...?


## Java Implementation
### Ref url
TODO: imple tree, redBlackTree

[Java tree implementation](http://www.quesucede.com/page/show/id/java-tree-implementation#app-class)

[树型的数据库结构Schema](http://blog.csdn.net/monkey_d_meng/article/details/6647488)

JAVA 红黑树
[Java集合干货系列-（四）TreeMap源码解析](http://tengj.top/2016/04/16/javajh4treemap/)