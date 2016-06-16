### Java Clone
#### Assignment
For primitive type, assigns real value
For reference type, assigns reference

#### Clone
java中的克隆为逐域复制。

在Java中想要支持clone方法，需要首先实现Cloneable接口

谨慎使用克隆

1. Shallow Copy
使用默认的clone方法
对于原始数据域进行值拷贝
对于引用类型仅拷贝引用
执行快，效率高
不能做到数据的100%分离。
如果一个对象只包含原始数据域或者不可变对象域，推荐使用浅拷贝。

2. Deep Copy
解决数据分离

需要重写clone方法，不仅仅只调用父类的方法，还需调用属性的clone方法
做到了原对象与克隆对象之间100%数据分离
如果是对象存在引用类型的属性，建议使用深拷贝
深拷贝比浅拷贝要更加耗时，效率更低

3. Copy Constructor
public Class (Class org) {
    //copy fields
    return cp;
}

4. 序列化
使用Serializable实现深拷贝


See More Codes in Ref URL:
[探究JAVA中的克隆 deep copy and shallow copy](http://droidyue.com/blog/2016/05/15/dive-into-java-clone/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)