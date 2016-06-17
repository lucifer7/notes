### 1. Future and Promises
Futures and promises are pretty similar concepts, the difference is that a future is a read-only container for a result that does not yet exist, while a promise can be written (normally only once). The Java 8 CompletableFuture and the Guava SettableFuture can be thought of as promises, because their value can be set ("completed"), but they also implement the Future interface, therefore there is no difference for the client.

Future:
read-only container
Promise:
can be written(normally only once)

[test this](http://blog.zhouhaocheng.cn/posts/41)

[More detail](http://marsishandsome.github.io/gen/posts/Scala/%E8%A7%A3%E8%AF%BBFuture%E5%92%8CPromise.html)

### 2. 悲观锁与乐观锁
悲观锁
synchronized

乐观锁
CAS (Compare and Swap)
> CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。

Problem:

可能导致ABA问题

线程1准备用CAS将变量的值由A替换为B，在此之前，线程2将变量的值由A替换为C，又由C替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。但实际上这时的现场已经和最初不同了

解决：

用版本戳version来对记录或对象标记，避免并发操作带来的问题

[CAS and ABA in JAVA](http://www.cnblogs.com/549294286/p/3766717.html)

[无锁队列的实现](http://coolshell.cn/articles/8239.html)

#### 2.1 operations like CAS
Fetch And Add，一般用来对变量做 +1 的原子操作

Test-and-set，写值到某个内存位置并传回其旧值。汇编指令BST

Test and Test-and-set，用来低低Test-and-Set的资源争夺情况


