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

###