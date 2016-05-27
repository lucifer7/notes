# java 并发编程读书笔记
[TOC]
## 第一章 简介
## 第二章 线程安全性
### 2.1 什么是线程安全性

**官方定义：**

		当多个线程在访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调方法中不需要额外的同步或者协同，这个类都能表现出正确的行为，那么称为这个类是线程安全的。

**个人理解：**

说了这么多，什么是正确的行为，关于这个问题没有所谓的标准答案，因此，部分人认为只要达到`代码可信性`就无限接近正确性，即(`we konw it when we see it`)，举个例子，我们访问某个公共变量i，此时读取到的值就是内存中的值，此时就是正确的行为。

因此，线程安全，就是多个线程在访问一个类的时候，不用其他额外的工作，这个类都是`正确的`。

	

### 2.2 原子性

	详情见多线程基础，一个操作序列才有原子性问题，其中的某个操作甚至依赖之前的操作。
	
	假定有2个操作A和B，如果从执行A的线程来看，当另一线程执行B时，要么将B全部执行完，要么完全不执行B，那么A和B对彼此来说就是原子的。

### 2.3 加锁机制 活跃性 性能



## 第三章 对象的共享

### 3.1 可见性

先看一段代码：

```java
package com.carl.demo.concurrency;

/**
 * Created by carl on 16-4-13.
 */
public class Novisibility {
    private static boolean ready;
    private static int number;

    private static class ReaderThread extends Thread {
        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }
            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new ReaderThread().start();
        number = 42;
        ready = true;
    }
}

```

**上述代码存在2个线程安全问题:**

1.重排序问题
```
number = 42;
ready = true;
```
这2个操作可能发生重排序，没有设置number的值的时候，已经修改ready为true了

2.可见性问题

ReaderThread线程可能永远看不到ready的值发生改变，因此堵塞
`System.out.println(number);`的时候可能读取到的不是正确的值



#### 3.1.1 失效数据
	类似于上面例子中失效的数据number,ready等，这不会太郁闷，但是可能造成其他的悲催问题，比如令人困惑的故障，意料之外的异常，被破坏的数据结构，不精确的计算以及死循环等等

#### 3.1.2 非原子性的64位操作

**最低安全性：**
		
		失效数据就算悲剧，想想的话也只是之前的一个数据。
		
`最低安全性`适用于大多数变量，但是依旧存在例外，**非volatile的64位数值变量**，就是double和long，JVM允许将64位的读操作或写操作分解成为2个操作，这样的结果是相当令人困惑的。可以用`volatile`或者`volatile`或者`volatile`或者锁保护起来。
	
#### 3.1.3 加锁与可见性

	加锁同时保证了互斥行为和可见性

#### 3.1.4 volatile变量

	不要用这伙来控制状态的可见性，常用用法：用来检查某个状态标记以判断是否退出循环。
	volitile 只能保证可见性，而加锁同时保证了可见性与互斥行为。


### 3.2 发布与逸出行为
### 3.3 线程封闭
#### 3.3.1 Ad-hoc线程封闭
#### 3.3.2 栈封闭
#### 3.3.3 ThreadLocal


###3.4 不变性

不可变对象都是线程安全的！，一个不可变对象需要满足以下条件：

1. 对象创建后不能修改
2. 所有field都是final
3. 对象被正确创建（this引用没有逸出）

测试atomic refernce

```java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceTest {
    
    public static void main(String[] args){

        // 创建两个Person对象，它们的id分别是101和102。
        Person p1 = new Person(101);
        Person p2 = new Person(102);
        // 新建AtomicReference对象，初始化它的值为p1对象
        AtomicReference ar = new AtomicReference(p1);
        // 通过CAS设置ar。如果ar的值为p1的话，则将其设置为p2。
        ar.compareAndSet(p1, p2);

        Person p3 = (Person)ar.get();
        System.out.println("p3 is "+p3);
        System.out.println("p3.equals(p1)="+p3.equals(p1));
    }
}

class Person {
    volatile long id;
    public Person(long id) {
        this.id = id;
    }
    public String toString() {
        return "id:"+id;
    }
}
```




## 第四章 对象的组合

看的蛋疼，直接跳过。。。


## 第五章 基础构建模块

### 5.1 同步容器类

#### 5.1.1 同步容器类的问题

**常见的复合操作：**

1. 迭代：反复的访问元素，直到遍历所有元素
2. 跳转：根据指定顺序找到当前元素的下一个元素 
3. 若没有则添加


在同步容器中，这些复合操作没有客户端加锁也是线程安全的，但是其他线程并发的修改容器时，可能会有bug，这时候，我们需要额外的锁，这时候的锁就很重要了，是哪把锁，如下demo。

```java
 public static Object getLast(Vector list) {
        synchronized (list){
            int lastIndex = list.size()-1;
            return list.get(lastIndex);
        }
    }
```


#### 5.1.2 迭代器与ConcurrentModificationException

```java
package com.carl.demo.concurrency;

import org.apache.poi.ss.formula.functions.T;

import java.util.ArrayList;
import java.util.List;
import java.util.Vector;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.CountDownLatch;

/**
 * Created by carl on 16-4-13.
 */
public class SafeVectorAct {

    public static Object getLast(Vector list) {
        synchronized (list) {
            int lastIndex = list.size() - 1;
            return list.get(lastIndex);
        }
    }

    // Vector ArrayList
    private static List<String> vector = new CopyOnWriteArrayList<>();

    static {
        for (int i = 0; i < 100; i++) {
            vector.add(i + "");
        }
    }

    static CountDownLatch latch = new CountDownLatch(1);

    private static class ChangeThread extends Thread {
        @Override
        public void run() {
            try {
                latch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            vector.remove(vector.size() - 1);
        }
    }

    public static void main(String[] args) throws Exception {
        new ChangeThread().start();
        int i = 0;
        for (String s : vector) {
            if ((i++) == 10) {
                latch.countDown();
            }
            System.out.println(s);
        }
    }

}

```

通过实验发现，只有`CopyOnWriteArrayList`这种集合才不会出异常，这很正常，明白了。
但是这种及时失败的容器并不是一种完备的处理机制。
为了避免这种情况，有2种思路：
1. 加锁，缺失是会引起性能问题，而且上面代码中由于跟CountDownLatch复合操作，很容易引起死锁，即使好吧，没有死锁风险，如何许多线程都在等待锁被释放，那么将极大的降低吞吐量和响应时间。
2. clone，但这种方式也取决于多个因素，包括容器的大小，在每个元素中执行的工作，迭代操作相对于其他操作的频率等等。


#### 5.1.3 隐藏的迭代器，toString()操作


### 5.2 并发容器
	
	
		BlockingQueue，ConcurrentHashMap等等。

#### 5.2.1 ConcurrentHashMap

利用锁分段技术优化了常用操作，先看一段代码明白什么是锁分段技术：

```java
package com.carl.demo.concurrency;

/**
 * Created by carl on 16-4-13.
 */
public class StripedMap {
    private static final int N_LOCKS = 16;

    //表示是散列筒
    private final Node[] buckets;
    //表示所有的锁
    private final Object[] locks;

    private static class Node {
        private Node next;
        private String key;
        private Object value;
    }

    public StripedMap(int numBuckets) {
        buckets = new Node[numBuckets];
        locks = new Object[N_LOCKS];
        for (int i = 0; i < locks.length; i++) {
            locks[i] = new Object();
        }
    }


    private final int hash(Object key) {
        return Math.abs(key.hashCode() % buckets.length);
    }

    public Object get(Object key) {
        // 先算出是哪个桶子中的数据
        int hash = hash(key);
        //
        synchronized (locks[hash % N_LOCKS]) {
            System.out.println("从这个桶中继续寻找数据.....");
            for (Node m = buckets[hash]; m != null; m = m.next) {
                if (m.key.equals(key)) {
                    return m.value;
                }
            }
        }
        return null;
    }


    public void clear() {
        for (int i = 0; i < buckets.length; i++) {
            synchronized (locks[i % N_LOCKS]) {
                buckets[i] = null;
            }
        }
    }
}

```


通过上述类似于`ConcurrentHashMap`思路的实现，对`get`操作进行优化，假设hash tables有n个bucket,有16个锁，在某个bucket k 上的对象都用 k%16这个锁，来同步。通过更细的锁粒度进行了优化。但像clear这样的操作，呃，则访问了16把锁，而且不要求同时获得。

是对一组独立对象的锁进行分解，锁与锁之间是相互独立的。


**下面来讨论ConcurrentHashMap在哪些部分做了优化**

1. get ,constains这样的操作只涉及一把锁
2. 迭代的时候不需要对容器加锁。它返回的迭代器具有弱一致性，这种迭代器可以容忍并发额修改，当创建迭代器的时会遍历已有的元素，但不保证在迭代器后将修改操作反映给容器。



**缺点：**

1 `size`和`isEmpty`返回的计算值，可能已经过期了，只是一个估计值。但是在并发操作中，这个值不重要，因为并发操作中的值总是在不断修改。

**总结：**
	与传统的`HashTable`相比，这样的容器具有更大的优点和更少的缺点。




#### 5.2.2 额外的原子Map操作


```java
	public interface ConcurrentMap<K,V> extends Map<K,V>{
		// 仅当K没有相应的映射才插入值
		V putIfAbsent(K key, V value);
		
		// 仅当当前值是value的时候才删除
		boolean remove(K key, V value);

		// 当前值是oldValue,换成newValue
		boolean replace(K key, V oldValue, V newValue );

		// 当前值是非null时，换成newValue
		boolean replace(K key, V newValue );
	}
```


### 5.3 堵塞队列和生产者-消费者模式

堵塞队列提供了可堵塞的put和take方法，以及定时的offer和poll方法。
	
	put和take方法的设计简化了消费者程序的编码，而且某些情况是非常合适的，例如，在服务器应用程序中，没有任何客户端请求服务。

	还可以使用offer方法，如果数据项不能被添加到队列中，那么将返回一个失败状态。这样你就能够创建更多灵活的策略来处理负荷过重的情况。

	
#### 5.3.1 SynchronousQueue详解

[synchronousqueue](http://ifeve.com/java-synchronousqueue/)

`SynchronousQueue`是一个没有数据缓冲的`BlockingQueue`,生产者线程对其的插入操作put必须等待消费者的移除操作take

下面是自定义实现，以方便理解:

```java
package com.carl.demo.concurrency;

/**
 * Created by carl on 16-4-14.
 */
public class NativeSynchronousQueue<E> {
    boolean putting = false;
    E item = null;

    public synchronized E take() throws InterruptedException {
        while (item == null) {
            this.wait();
        }
        E e = item;
        item = null;
        notifyAll();
        return e;
    }

    /**
     * <pre>
     *      放入数据项
     * </pre>
     *
     * @param e item项
     * @throws InterruptedException
     */
    public synchronized void put(E e) throws InterruptedException {
        if (e == null) {
            return;
        }
        //如何正在put，等待
        while (putting) {
            wait();
        }
        putting = true;
        item = e;
        notifyAll();
        //还必须会取走，否则不能再加
        while (item != null) {
            wait();
        }
        putting = false;
        notifyAll();
    }


    public static void main(String[] args) throws Exception {
        final NativeSynchronousQueue<String> queue = new NativeSynchronousQueue<>();
        Thread thread = new Thread() {
            @Override
            public void run() {
                try {
                    System.out.println("子线程先睡一会");
                    Thread.sleep(100);
                    System.out.println("拿到的数据是：" + queue.take());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        thread.start();
        queue.put("1");
    }

}


```


#### 5.3.2 AQS

[AQS简单介绍](http://ifeve.com/java-special-troops-aqs/)
[AQS原论文-真大神](http://gee.cs.oswego.edu/dl/papers/aqs.pdf)
[AQS系列文章-深入浅出](http://www.blogjava.net/xylz/archive/2010/07/08/325587.html)



#### 5.3.3 双端队列与工作密取

	Java 6 中增加了两种容器类型，Deque和BlockingDeque，它们分别对Queue和BlockingQueue进行了扩展。


双端队列适用于工作密取型的设计(Working Stealing)。在生产者-消费者设计中，所有消费者共享一个工作队列，然后在工作密取型设计中，每个消费者都拥有自己的队列，如果一个消费者消费完了自己的队列，可以去其他的消费者的对尾获取新的任务，这种好处是进一步降低了队列上的竞争程度。
暂时理解到这里。。。



### 5.4 堵塞方法和中断方法

java通过传递的方法来实现中断`interrupt`，对于代码而言，通常有两种选择：

**1. 传递：** 避开这个异常是最明智的选择，此时有2种做法:

直接抛出异常:
```java
public void testInterrupt() throws InterruptedException {
        //dosomething
        Thread.sleep(1000);
    }
```

做点日志再抛出

```
public void testInterrupt() throws InterruptedException {
        //dosomething
        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            //do some log
            throw new InterruptedException(e.getMessage()); //...
        }

    }
```


**2.恢复中断:**

```java
static class TaskRunnable implements Runnable {
        BlockingQueue<Task> queue;

        public void run() {
            try {
                processTask(queue.take());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
```



### 5.5 同步工具类		

#### 5.5.1 闭锁的概念与使用

**闭锁：** 是一种同步工具类，可以延迟线程的进度到其终止状态。闭锁的作用相当于一扇门，在闭锁达到结束状态之前，这扇门一直是失败的，当达到结束状态之后，这扇门会允许所有的线程通过。


**例如：**

1. 确保某个计算在其所需要资源都会初始化之后才继续执行。二元闭锁（包括两个状态）可以用来表示“资源R已经被初始化”，而所有需要R的操作都必须先在这个闭锁上等待。
2. 确保某个服务在其依赖的所有其他服务都已经启动后再启动。每个服务都有一个二元闭锁。当启动服务S的时候，在所有依赖都成功后都会释放闭锁S，这样其他依赖S的服务才能继续执行
3. 等待某个操作的所有参与者都就绪再继续执行。

**上述第二点理解如下：**
	
	联系Dubbo中的依赖，呃，假设要添加依赖，我们会怎么样启动服务呢。
假设现在有服务A，B，C，D，E，F，G，其中A，B，C，D必须在先S启动前启动，E，F，G必须在S启动后启动，就可以使用二元闭锁。

```java
package com.carl.demo.concurrency.basic;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 16-4-15.
 */
public class CountDownLatchDemo {

    private static class MyThread extends Thread {
        private CountDownLatch start;
        private CountDownLatch end;
        private String name;

        private MyThread(CountDownLatch start, CountDownLatch end, String name) {
            this.name = name;
            this.start = start;
            this.end = end;
        }

        @Override
        public void run() {
            try {
                start.await();
                Thread.sleep(1000);
                System.out.println(name + "已经启动");
                end.countDown();
            } catch (InterruptedException e) {

            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newCachedThreadPool();
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch end = new CountDownLatch(4);
        for (int i = 0; i < 4; i++) {
            pool.execute(new MyThread(start, end, "服务" + (char) ('A' + i)));
        }
        //上述服务A，B，C，D启动后启动S
        start.countDown();
        //S启动成功之后启动E,F,G
        end.await();
        System.out.println("启动服务S");
        //下面写可以仿照上面
        System.out.println("启动E,F,G");
        pool.shutdown();
    }
}

```

#### 5.5.2 FutureTask Callable实现任务。。
#### 5.5.3 信号量与许可Semaphore
#### 5.5.4 Barrier

允许多个线程之间相互等待

```
package com.carl.demo.concurrency.basic;

import java.io.IOException;
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 16-4-15.
 */
public class CyclicBarrierTest {
    static class Runner implements Runnable {
        // 一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)
        private CyclicBarrier barrier;

        private String name;

        public Runner(CyclicBarrier barrier, String name) {
            super();
            this.barrier = barrier;
            this.name = name;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(1000 * (new Random()).nextInt(8));
                System.out.println(name + " 准备好了...");
                // barrier的await方法，在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。
                barrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(name + " 起跑！");
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        //如果将参数改为4，但是下面只加入了3个选手，这永远等待下去
        //Waits until all parties have invoked await on this barrier.
        CyclicBarrier barrier = new CyclicBarrier(3);

        ExecutorService executor = Executors.newFixedThreadPool(3);
        executor.submit(new Thread(new Runner(barrier, "1号选手")));
        executor.submit(new Thread(new Runner(barrier, "2号选手")));
        executor.submit(new Thread(new Runner(barrier, "3号选手")));

        executor.shutdown();
    }
}

```
	
	
### 5.5.5 构建缓存

```java
package net.jcip.examples;

import java.util.concurrent.*;

/**
 * Memoizer
 * <p/>
 * Final implementation of Memoizer
 *
 * @author Brian Goetz and Tim Peierls
 */
public class Memoizer <A, V> implements Computable<A, V> {
    private final ConcurrentMap<A, Future<V>> cache
            = new ConcurrentHashMap<A, Future<V>>();
    private final Computable<A, V> c;

    public Memoizer(Computable<A, V> c) {
        this.c = c;
    }

    public V compute(final A arg) throws InterruptedException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> eval = new Callable<V>() {
                    public V call() throws InterruptedException {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<V>(eval);
                f = cache.putIfAbsent(arg, ft);
                if (f == null) {
                    f = ft;
                    ft.run();
                }
            }
            try {
                return f.get();
            } catch (CancellationException e) {
                cache.remove(arg, f);
            } catch (ExecutionException e) {
                throw LaunderThrowable.launderThrowable(e.getCause());
            }
        }
    }
}
```


## 第六章 任务执行

### 6.1 在线程中执行任务

#### 6.1.1 串行的执行任务

```java
package net.jcip.examples;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * SingleThreadWebServer
 * <p/>
 * Sequential web server
 *
 * @author Brian Goetz and Tim Peierls
 */

public class SingleThreadWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            Socket connection = socket.accept();
            handleRequest(connection);
        }
    }

    private static void handleRequest(Socket connection) {
        // request-handling logic here
    }
}
```

以上代码是一个明显的串行服务器，在WEB请求的处理中包含了一组不同的运算和I/O操作。服务器必须处理套接字I/0以读取请求和写回响应，由于网络的原因，这些操作会阻塞，这会彻底的中断其他的请求，同时，服务器的资源利用率非常的低，因为当单线程在等待I/0操作完成时，CPU将处于空闲状态。


#### 6.1.2 显示创建线程

```java
 public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            new Thread(task).start();
        }
    }

    private static void handleRequest(Socket connection) {
        // request-handling logic here
    }
```


三个结论：

- 任务处理过程从主线程中分离，可以更快的处理新的请求
- 任务可以并行处理，从而能同时服务多个请求。如果有多个处理器，程序的吞吐量将会提高
- 任务的处理代码必须是线程安全的


#### 6.1.3 无限制创建线程不足

**线程生命周期的开销非常的高**

	线程的创建需要事件，延迟处理的请求，并且需要JVM和操作系统提供一些辅助操作。

**资源消耗**

	活跃的线程会消耗系统资源，尤其是内存，如果可运行的线程数目多于可用处理器的数量，那么有些线程将会闲置。大量空闲的线程会占用许多的内存，给垃圾回收期带来压力，而且大量的线程在竞争CPU的时候会产生其他的开销。
	如果你已经有足够的多的线程让CPU保持忙碌状态，那么再创建更多的线程反而会降低性能。

**稳定性**

	在可创建的线程数量上有一个限制。这个限制值将随着平台的不同而不同，受到多个参数制约，包括JVM的启动参数，Thread在构造函数中请求的栈的大小，以及底层的线程。如果你破坏了这些限制是非常危险的。
	在高负载状况下实在的太危险了。


### 6.2 Executor 框架

`Executor`框架基于生产者消费者模式，提交任务的操作相当于生产者（生产工作单元），执行任务的线程相当于消费者（消费工作单元）。

```java
	public interface Executor{
		void execute( Runnable command );
	}
```

而且这进行了解藕，将请求处理任务的提交和任务的实际执行解藕，只需要采取不同Executor的实现就可以完全改变服务器的行为。



#### 6.2.1 基于Executor的Web服务器

```java
package net.jcip.examples;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.*;

/**
 * TaskExecutionWebServer
 * <p/>
 * Web server using a thread pool
 *
 * @author Brian Goetz and Tim Peierls
 */
public class TaskExecutionWebServer {
    private static final int NTHREADS = 100;
    private static final Executor exec
            = Executors.newFixedThreadPool(NTHREADS);

    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            exec.execute(task);
        }
    }

    private static void handleRequest(Socket connection) {
        // request-handling logic here
    }
}

```

#### 6.2.2 执行策略

	解藕之后你可以很方便指定执行的策略

1. 在（what）什么线程中执行任务
2. 任务按照什么顺序执行?FIFO、LIFO、优先级
3. 有多少个任务可以并发执行
4. 在队列中有多少个任务在等待执行
5. 如果系统中由于过载需要拒绝一个任务，应该选择哪一个
6. 执行一个任务之前或者之后需要进行哪些动作？


通过这些选择的组合定制最佳策略


#### 6.2.3 线程池

`Executors` 提供了生产线程池的静态工厂方法如下：

`newFixedThreadPool`：创建一个固定长度的线程池，每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程池的规模将不再发生变化（如果某个线程由于发生了未预期的Exception而结束，那么线程池将补充一个新的线程）。

`newCachedThreadPool`：创建一个可以**缓冲**的线程池，如果线程池的规模超过目前需求，将会回收空闲的线程，线程池的规模没有任何限制。

`newSingleThreadExecutor`: 一个单线程，创建某个工作线程来执行任务，如果这个线程由于异常而结束，会创建一个新的线程来替代，他能确保任务在队列中的顺序来串行执行。（例如FIFO、LIFO、优先级）

`newScheduleThreadPool`：创建一个固定长度的线程池，而且可以延迟或定时的方法来执行任务，类似于Timer


>通过使用Executor，可以实现各种调优，管理，监视，记录日志，错误报告和其他功能。


#### 6.2.4 Executor的生命周期

线程池如果没有关闭的话，JVM将无法自然结束，因此，`Executor`扩展了`ExecutorService`接口，添加了生命周期管理的方法。

`shutdown` ： 执行平缓的关闭过程，不再接受新的任务，同时等待现有任务结束
`shutdownNow`：将执行粗暴的关闭，不再接受新的任务，尝试取消所有的任务

在`ExecutorService`关闭后 新提交的任务将有**拒绝处理器**（`Rejected Execution Handler`）来处理。

等所有任务完成后，进入终止状态，可以`awaitTermination()`来等待终止状态，`isTerminated`来轮巡是否终止。

### 6.3 找出可以利用的并行性，很关键

#### 6.3.1 Future 和 Callable

```java
package net.jcip.examples;

import java.util.*;
import java.util.concurrent.*;
import static net.jcip.examples.LaunderThrowable.launderThrowable;

/**
 * FutureRenderer
 * <p/>
 * Waiting for image download with \Future
 *
 * @author Brian Goetz and Tim Peierls
 */
public abstract class FutureRenderer {
    private final ExecutorService executor = Executors.newCachedThreadPool();

    void renderPage(CharSequence source) {
        final List<ImageInfo> imageInfos = scanForImageInfo(source);
        Callable<List<ImageData>> task =
                new Callable<List<ImageData>>() {
                    public List<ImageData> call() {
                        List<ImageData> result = new ArrayList<ImageData>();
                        for (ImageInfo imageInfo : imageInfos)
                            result.add(imageInfo.downloadImage());
                        return result;
                    }
                };

        Future<List<ImageData>> future = executor.submit(task);
        renderText(source);

        try {
            List<ImageData> imageData = future.get();
            for (ImageData data : imageData)
                renderImage(data);
        } catch (InterruptedException e) {
            // Re-assert the thread's interrupted status
            Thread.currentThread().interrupt();
            // We don't need the result, so cancel the task too
            future.cancel(true);
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }

    interface ImageData {
    }

    interface ImageInfo {
        ImageData downloadImage();
    }

    abstract void renderText(CharSequence s);

    abstract List<ImageInfo> scanForImageInfo(CharSequence s);

    abstract void renderImage(ImageData i);
}

```


`FutureRender`使用了2个任务，其中一个负责文本的渲染，另一个负责下载图像，获取性能的提升取决于二者的速度比率，假如文本的渲染速度远大于下载图像，并行将毫无意义，反而会让代码变的复杂。

只有大量相互独立且同构的任务可以并发执行时，才能体现出将程序的工作负载到多个任务中带来的真正性能提升。


#### 6.3.2 CompletionService：Executor与BlockingQueue

	
```java
package net.jcip.examples;

import java.util.*;
import java.util.concurrent.*;
import static net.jcip.examples.LaunderThrowable.launderThrowable;

/**
 * Renderer
 * <p/>
 * Using CompletionService to render page elements as they become available
 *
 * @author Brian Goetz and Tim Peierls
 */
public abstract class Renderer {
    private final ExecutorService executor;

    Renderer(ExecutorService executor) {
        this.executor = executor;
    }

    void renderPage(CharSequence source) {
        final List<ImageInfo> info = scanForImageInfo(source);
        CompletionService<ImageData> completionService =
                new ExecutorCompletionService<ImageData>(executor);
        for (final ImageInfo imageInfo : info)
            completionService.submit(new Callable<ImageData>() {
                public ImageData call() {
                    return imageInfo.downloadImage();
                }
            });

        renderText(source);

        try {
            for (int t = 0, n = info.size(); t < n; t++) {
                Future<ImageData> f = completionService.take();
                ImageData imageData = f.get();
                renderImage(imageData);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }

    interface ImageData {
    }

    interface ImageInfo {
        ImageData downloadImage();
    }

    abstract void renderText(CharSequence s);

    abstract List<ImageInfo> scanForImageInfo(CharSequence s);

    abstract void renderImage(ImageData i);

}

```

**思路：**

	为每一幅图像的下载都创建一个独立任务，并在线程池中执行他们，从而将串行的下载过程转换为并行的过程：这将减少下载所有图像的总时间。此外，通过CompleteionService获取结果以及使每张图片都在下载完成后立刻显示出来，能使用户获取更加动态和更加响应性的用户界面。



#### 6.3.3 Future 定时

本身`Future`支持超时方法，也可以考虑`invokeAll`方法


## 第七章 取消和关闭