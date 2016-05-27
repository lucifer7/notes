# Java 多线程 - 基础篇


[TOC]

## 帮助文档
[java并发编程中文](http://ifeve.com/java-7-concurrency-cookbook/)
[并发笔记](http://ifeve.com/java-concurrency-thread-directory/)

## 一、基础概念

###1.1 JAVA 内存模型

**一、首先，来关注JVM一般虚拟机模型**
某些虚拟机问题不考虑，比如说有的虚拟机实现有专门的方法区（永久代）

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-3.png)

1. A local variable may be of a primitive type, in which case it is totally kept on the thread stack.

2. A local variable may also be a reference to an object. In that case the reference (the local variable) is stored on the thread stack, but the object itself if stored on the heap.

3. An object may contain methods and these methods may contain local variables. These local variables are also stored on the thread stack, even if the object the method belongs to is stored on the heap.

4. An object's member variables are stored on the heap along with the object itself. That is true both when the member variable is of a primitive type, and if it is a reference to an object.

5. Static class variables are also stored on the heap along with the class definition.

Objects on the heap can be accessed by all threads that have a reference to the object. When a thread has access to an object, it can also get access to that object's member variables. If two threads call a method on the same object at the same time, they will both have access to the object's member variables, but each thread will have its own copy of the local variables.

然而堆是被多线程所共有的，存在多线程并发问题，以下代码会出现上图问题。
```java
public class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 =
            MySharedObject.sharedInstance;

        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}


public class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance =
        new MySharedObject();


    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);

    public long member1 = 12345;
    public long member1 = 67890;
}
```

**二、硬件架构**

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-4.png)

一个现代计算机通常由两个或者多个CPU。其中一些CPU还有多核。从这一点可以看出，在一个有两个或者多个CPU的现代计算机上同时运行多个线程是可能的。每个CPU在某一时刻运行一个线程是没有问题的。这意味着，如果你的Java程序是多线程的，在你的Java程序中每个CPU上一个线程可能同时（并发）执行。

每个CPU都包含一系列的寄存器，它们是CPU内内存的基础。CPU在寄存器上执行操作的速度远大于在主存上执行的速度。这是因为CPU访问寄存器的速度远大于主存。

每个CPU可能还有一个CPU缓存层。实际上，绝大多数的现代CPU都有一定大小的缓存层。CPU访问缓存层的速度快于访问主存的速度，但通常比访问内部寄存器的速度还要慢一点。一些CPU还有多层缓存，但这些对理解Java内存模型如何和内存交互不是那么重要。只要知道CPU中可以有一个缓存层就可以了。

一个计算机还包含一个主存。所有的CPU都可以访问主存。主存通常比CPU中的缓存大得多。

通常情况下，当一个CPU需要读取主存时，它会将主存的部分读到CPU缓存中。它甚至可能将缓存中的部分内容读到它的内部寄存器中，然后在寄存器中执行操作。当CPU需要将结果写回到主存中去时，**它会将内部寄存器的值刷新到缓存中，然后在某个时间点将值刷新回主存。**

当CPU需要在缓存层存放一些东西的时候，存放在缓存中的内容通常会被刷新回主存。CPU缓存可以在某一时刻将数据局部写到它的内存中，和在某一时刻局部刷新它的内存。它不会再某一时刻读/写整个缓存。通常，在一个被称作“cache lines”的更小的内存块中缓存被更新。一个或者多个缓存行可能被读到缓存，一个或者多个缓存行可能再被刷新回主存


**三、JVM和硬件架构**

堆和栈是绑定在主存的，部分堆和栈有时会存在寄存器和cpu缓存
As already mentioned, the Java memory model and the hardware memory architecture are different. The hardware memory architecture does not distinguish between thread stacks and heap. On the hardware, both the thread stack and the heap are located in **main memory**. Parts of the thread stacks and heap **may sometimes be present in CPU caches and in internal CPU registers.** This is illustrated in this diagram:

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)


这会导致如下2个问题：

- Visibility of thread updates (writes) to shared variables.(可见性)
- Race conditions when reading, checking and writing shared variables.(竞态条件)

**可见性问题：**

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-6.png)
可以发现一个线程在CPU Cache Memory中修改了obj.count的值为2，但是没有刷入主存，导致，右边的那个线程读到的值依旧是obj.count=1

**解决这个问题**你可以使用Java中的`volatile`关键字。volatile关键字可以保证直接从主存中读取一个变量，如果这个变量被修改后，总是会被写回到主存中去。


**Race Conditions**

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-7.png)

二个线程同时对obj.count执行+1操作，然后你懂的，本来应该+2,现在结果是+1
To solve this problem you can use a `Java synchronized block.` A synchronized block guarantees that only one thread can enter a given critical section of the code at any given time. Synchronized blocks also guarantee that all variables accessed inside the synchronized block will be read in from main memory, and when the thread exits the synchronized block, all updated variables will be flushed back to main memory again, regardless of whether the variable is declared volatile or not.
解决这个问题可以使用Java同步块。一个同步块可以保证在同一时刻仅有一个线程可以进入代码的临界区。同步块还可以保证代码块中所有被访问的变量将会从主存中读入，当线程退出同步代码块时，所有被更新的变量都会被刷新回主存中去，不管这个变量是否被声明为volatile。


### 1.2 并发编程模型

**一、Parallel Workers**

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-2.png)


**二、Assembly Line**
![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-5.png)

类似于工厂中生产线上的工人们那样组织工作者。每个工作者只负责作业中的部分工作。当完成了自己的这部分工作时工作者会将作业转发给下一个工作者。每个工作者在自己的线程中运行，并且不会和其他工作者共享状态。有时也被成为无共享并行模型。

通常使用非阻塞的IO来设计使用流水线并发模型的系统。非阻塞IO意味着，一旦某个工作者开始一个IO操作的时候（比如读取文件或从网络连接中读取数据），这个工作者不会一直等待IO操作的结束。IO操作速度很慢，所以等待IO操作结束很浪费CPU时间。此时CPU可以做一些其他事情。当IO操作完成的时候，IO操作的结果（比如读出的数据或者数据写完的状态）被传递给下一个工作者。

有了非阻塞IO，就可以使用IO操作确定工作者之间的边界。工作者会尽可能多运行直到遇到并启动一个IO操作。然后交出作业的控制权。当IO操作完成的时候，在流水线上的下一个工作者继续进行操作，直到它也遇到并启动一个IO操作。


采用流水线并发模型的系统有时候也称为反应器系统或事件驱动系统。系统内的工作者对系统内出现的事件做出反应，这些事件也有可能来自于外部世界或者发自其他工作者。事件可以是传入的HTTP请求，也可以是某个文件成功加载到内存中等。在写这篇文章的时候，已经有很多有趣的反应器/事件驱动平台可以使用了，并且不久的将来会有更多。比较流行的似乎是这几个：

Vert.x
AKKa
Node.JS(JavaScript)
我个人觉得Vert.x是相当有趣的（特别是对于我这样使用Java/JVM的人来说）

**Actor**
在Actor模型中每个工作者被称为actor。Actor之间可以直接异步地发送和处理消息。Actor可以被用来实现一个或多个像前文描述的那样的作业处理流水线。下图给出了Actor模型：

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-7.png)

**Channel**

工作者之间不直接进行通信。相反，它们在不同的通道中发布自己的消息（事件）。其他工作者们可以在这些通道上监听消息，发送者无需知道谁在监听。
![enter image description here](http://tutorials.jenkov.com/images/java-concurrency/concurrency-models-8.png)

**优点**:
The assembly line concurrency model has several advantages compared to the parallel worker model. I will cover the biggest advantages in the following sections.

- **No Shared State**
The fact that workers share no state with other workers means that they can be implemented without having to think about all the concurrency problems that may arise from concurrent access to shared state. This makes it much easier to implement workers. You implement a worker as if it was the only thread performing that work - essentially a singlethreaded implementation.
工作者之间无需共享状态，意味着实现的时候无需考虑所有因并发访问共享对象而产生的并发性问题。这使得在实现工作者的时候变得非常容易。在实现工作者的时候就好像是单个线程在处理工作-基本上是一个单线程的实现。


- **Stateful Workers**
Since workers know that no other threads modify their data, the workers can be stateful. By stateful I mean that they can keep the data they need to operate in memory, only writing changes back the eventual external storage systems. A stateful worker can therefore often be faster than a stateless worker.
当工作者知道了没有其他线程可以修改它们的数据，工作者可以变成有状态的。对于有状态，我是指，它们可以在内存中保存它们需要操作的数据，只需在最后将更改写回到外部存储系统。因此，有状态的工作者通常比无状态的工作者具有更高的性能。

- **Better Hardware Conformity**
Singlethreaded code has the advantage that it often conforms better with how the underlying hardware works. First of all, you can usually create more optimized data structures and algorithms when you can assume the code is executed in single threaded mode.
**Second**, singlethreaded stateful workers can cache data in memory as mentioned above. When data is cached in memory there is also a higher probability that this data is also cached in the CPU cache of the CPU executing the thread. This makes accessing cached data even faster.
I refer to it as hardware conformity when code is written in a way that naturally benefits from how the underlying hardware works. Some developers call this mechanical sympathy. I prefer the term hardware conformity because computers have very few mechanical parts, and the word "sympathy" in this context is used as a metaphor for "matching better" which I believe the word "conform" conveys reasonably well. Anyways, this is nitpicking. Use whatever term you prefer.
单线程代码在整合底层硬件的时候往往具有更好的优势。
**首先**，当能确定代码只在单线程模式下执行的时候，通常能够创建更优化的数据结构和算法。
**其次**，像前文描述的那样，单线程有状态的工作者能够在内存中缓存数据。在内存中缓存数据的同时，也意味着数据很有可能也缓存在执行这个线程的CPU的缓存中。这使得访问缓存的数据变得更快。
我说的硬件整合是指，以某种方式编写的代码，使得能够自然地受益于底层硬件的工作原理。有些开发者称之为mechanical sympathy。我更倾向于硬件整合这个术语，因为计算机只有很少的机械部件，并且能够隐喻“更好的匹配（match better）”，相比“同情（sympathy）”这个词在上下文中的意思，我觉得“conform”这个词表达的非常好。当然了，这里有点吹毛求疵了，用自己喜欢的术语就行。

- **Job Ordering is Possible**
It is possible to implement a concurrent system according to the assembly line concurrency model in a way that guarantees job ordering. Job ordering makes it much easier to reason about the state of a system at any given point in time. Furthermore, you could write all incoming jobs to a log. This log could then be used to rebuild the state of the system from scratch in case any part of the system fails. The jobs are written to the log in a certain order, and this order becomes the guaranteed job order. Here is how such a design could look:

**缺点：**

		The main disadvantage of the assembly line concurrency model is that the execution of a job is often spread out over multiple workers, and thus over multiple classes in your project. Thus it becomes harder to see exactly what code is being executed for a given job.It may also be harder to write the code. Worker code is sometimes written as callback handlers. Having code with many nested callback handlers may result in what some developer call callback hell. Callback hell simply means that it gets hard to track what the code is really doing across all the callbacks, as well as making sure that each callback has access to the data it needs.With the parallel worker concurrency model this tends to be easier. You can open the worker code and read the code executed pretty much from start to finish. Of course parallel worker code may also be spread over many different classes, but the execution sequence is often easier to read from the code.
		通常情况下，这个答案取决于你的系统打算做什么。如果你的作业本身就是并行的、独立的并且没有必要共享状态，你可能会使用并行工作者模型去实现你的系统。虽然许多作业都不是自然并行和独立的。对于这种类型的系统，我相信使用流水线并发模型能够更好的发挥它的优势，而且比并行工作者模型更有优势。你甚至不用亲自编写所有流水线模型的基础结构。像Vert.x这种现代化的平台已经为你实现了很多。我也会去为探索如何设计我的下一个项目，使它运行在像Vert.x这样的优秀平台上。我感觉Java EE已经没有任何优势了。

**三、Functional Parallelism**

第三种并发模型是函数式并行模型，这是也最近（2015）讨论的比较多的一种模型。函数式并行的基本思想是采用函数调用实现程序。函数可以看作是”代理人（agents）“或者”actor“，函数之间可以像流水线模型（AKA 反应器或者事件驱动系统）那样互相发送消息。某个函数调用另一个函数，这个过程类似于消息发送。

函数都是通过拷贝来传递参数的，所以除了接收函数外没有实体可以操作数据。这对于避免共享数据的竞态来说是很有必要的。同样也使得函数的执行类似于原子操作。每个函数调用的执行独立于任何其他函数的调用。

一旦每个函数调用都可以独立的执行，它们就可以分散在不同的CPU上执行了。这也就意味着能够在多处理器上并行的执行使用函数式实现的算法。

Java7中的`java.util.concurrent`包里包含的ForkAndJoinPool能够帮助我们实现类似于函数式并行的一些东西。而Java8中并行streams能够用来帮助我们并行的迭代大型集合。记住有些开发者对`ForkAndJoinPool`进行了批判（你可以在我的ForkAndJoinPool教程里面看到批评的链接）。

函数式并行里面最难的是确定需要并行的那个函数调用。跨CPU协调函数调用需要一定的开销。某个函数完成的工作单元需要达到某个大小以弥补这个开销。如果函数调用作用非常小，将它并行化可能比单线程、单CPU执行还慢。

我个人认为（可能不太正确），你可以使用反应器或者事件驱动模型实现一个算法，像函数式并行那样的方法实现工作的分解。使用事件驱动模型可以更精确的控制如何实现并行化（我的观点）。
此外，将任务拆分给多个CPU时协调造成的开销，仅仅在该任务是程序当前执行的唯一任务时才有意义。但是，如果当前系统正在执行多个其他的任务时（比如web服务器，数据库服务器或者很多其他类似的系统），将单个任务进行并行化是没有意义的。不管怎样计算机中的其他CPU们都在忙于处理其他任务，没有理由用一个慢的、函数式并行的任务去扰乱它们。使用流水线（反应器）并发模型可能会更好一点，因为它开销更小（在单线程模式下顺序执行）同时能更好的与底层硬件整合。



### 1.3 Amdahl定律

$$speedup\leq\frac{1}{F+\frac{1-F}{N}}$$

N: cpu个数
F: 必须串行的部分

1. F 越大，串行比例越大，N越大也作用不大，即当程序串行化部分比例较大时，再多的cpu也无济于事
2. F 固定，最大值逼近1/F
3. F 较小的时候，增加N是有意义的，因为1-F较大，减少N，`(1-F)/N`变化明显
4. 对N求一阶偏导数>0 ，说明增加cpu能够提速
5. 对N求二阶偏导数<0，说明cpu增加的效果越来越不明显

### 1.4 JVM的重排序

重排序是编译器运行时环境优化性能而采取的指令进行重排序的一种手段。
1. 数据依赖性
	

| 名称        | 代码示例          | 说明  |
| ------------- |:-------------:| -----:|
| 写后读	      | a = 1;b = a; | 写一个变量之后，再读这个位置 |
| 写后写	      | a = 1;b = a; | 写一个变量之后，再写这个变量。 |
| 读后写	      | a = b;b = 1; | 读一个变量之后，再写这个变量。 |

2. **as-if-serial**
不管如何重排序，不能改变程序的结果.`as-if-serial`语义把单线程程序保护了起来，遵守`as-if-serial`语义的编译器，`runtime` 和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的。`as-if-serial`语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。<br/>
3. **demo**
	```java
    	class ReorderExample {
            int a = 0;
            boolean flag = false;

            public void writer() {
                a = 1;                   //1
                flag = true;             //2
            }

            Public void reader() {
                if (flag) {                //3
                    int i =  a * a;        //4
                    ……
                }
            }
            }
    ```
    - (1) and (2)
    - (3) and (4):程序可能在 `if(flag)`之前算出`int i = a*a` 的值先缓存起来，如果flag成立，直接赋值，因此也可能存在重排序问题
    - volatile 关键字修饰的变量，对其操作会停止冲排序，为了能够保证该变量的写操作对所有线程可见
4. **Happen before**

    >Java存储模型有一个happens-before原则，就是如果动作B要看到动作A的执行结果（无论A/B是否在同一个线程里面执行），那么A/B就需要满足happens-before关系。

    >在介绍happens-before法则之前介绍一个概念：JMM动作（Java Memeory Model Action），Java存储模型动作。一个动作（Action）包括：变量的读写、监视器加锁和释放锁、线程的start()和join()。后面还会提到锁的的。
	>
	> - 同一个线程中的每个Action都happens-before于出现在其后的任何一个Action。

	> - 对一个监视器的解锁happens-before于每一个后续对同一个监视器的加锁。

	> - 对volatile字段的写入操作happens-before于每一个后续的同一个字段的读操作。

	> - Thread.start()的调用会happens-before于启动线程里面的动作。

	> - Thread中的所有动作都happens-before于其他线程检查到此线程结束或者Thread.join（）中返回或者Thread.isAlive()==false。

	> - 一个线程A调用另一个另一个线程B的interrupt（）都happens-before于线程A发现B被A中断（B抛出异常或者A检测到B的isInterrupted（）或者interrupted()）。

	> - 一个对象构造函数的结束happens-before与该对象的finalizer的开始

	> - 如果A动作happens-before于B动作，而B动作happens-before与C动作，那么A动作happens-before于C动作。



### 1.5 volatile
**volatile的语义:**
1. Java 存储模型不会对valatile指令的操作进行重排序：这个保证对volatile变量的操作时按照指令的出现顺序执行的。

2. volatile变量不会被缓存在寄存器中（只有拥有线程可见）或者其他对CPU不可见的地方，每次总是从主存中读取volatile变量的结果。也就是说对于volatile变量的修改，其它线程总是可见的，并且不是使用自己线程栈内部的变量。也就是在happens-before法则中，对一个valatile变量的写操作后，其后的任何读操作理解可见此写操作的结果。

**volatile的适用性:**

1. 写入变量不依赖此变量的值，或者只有一个线程修改此变量

2. 变量的状态不需要与其它变量共同参与不变约束

3. 访问变量不需要加锁

**Demos:**

1. 状态标识
	```java
    volatile boolean shutdownRequested;
    public void shutdown() { shutdownRequested = true; }

    public void doWork() {
        while (!shutdownRequested) {
            // do stuff
        }
    }
    ```
2. 一次性安全发布
	```java
    public class UserManager {
    public volatile String lastUser;

    public boolean authenticate(String user, String password) {
        boolean valid = passwordIsValid(user, password);
        if (valid) {
            User u = new User();
            activeUsers.add(u);
            lastUser = user;
        }
        return valid;
    }
    ```
3. “volatile bean” 模式
	```java
    @ThreadSafe
    public class Person {
        private volatile String firstName;
        private volatile String lastName;
        private volatile int age;

        public String getFirstName() { return firstName; }
        public String getLastName() { return lastName; }
        public int getAge() { return age; }

        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }

        public void setLastName(String lastName) {
            this.lastName = lastName;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
    ```

4. 开销较低的读－写锁策略
	```java
    @ThreadSafe
    public class CheesyCounter {
        // Employs the cheap read-write lock trick
        // All mutative operations MUST be done with the 'this' lock held
        @GuardedBy("this") private volatile int value;

        public int getValue() { return value; }

        public synchronized int increment() {
            return value++;
        }
    }
    ```



## 二、多线程的优点和缺点

### 优点

**一、资源利用率更好**-尽可能的减少cpu空闲的时间，完成一个任务并不需要这么多的cpu
想象一下，一个应用程序需要从本地文件系统中读取和处理文件的情景。比方说，从磁盘读取一个文件要5秒，处理一个文件需要2秒。处理两个文件则需要：

- 5秒读取文件A
- 2秒处理文件A
- 5秒读取文件B
- 2秒处理文件B

总共需要`14`秒
从磁盘中读取文件的时候，大部分的CPU时间用于等待磁盘去读取数据。在这段时间里，**CPU非常的空闲**。它可以做一些别的事情。通过改变操作的顺序，就能够更好的使用CPU资源。看下面的顺序：

- 5秒读取文件A
- 5秒读取文件B + 2秒处理文件A
- 2秒处理文件B

总共需要`12`秒
CPU等待第一个文件被读取完。然后开始读取第二个文件。当第二文件在被读取的时候，CPU会去处理第一个文件。记住，在等待磁盘读取文件的时候，**CPU大部分时间是空闲的**。

总的说来，CPU能够在等待IO的时候做一些其他的事情。这个不一定就是磁盘IO。它也可以是网络的IO，或者用户输入。通常情况下，网络和磁盘的IO比CPU和内存的IO慢的多。

**二、程序设计更简单**
在单线程应用程序中，如果你想编写程序手动处理上面所提到的读取和处理的顺序，你必须记录每个文件读取和处理的状态。相反，你可以启动两个线程，每个线程处理一个文件的读取和操作。线程会在等待磁盘读取文件的过程中被阻塞。在等待的时候，其他的线程能够使用CPU去处理已经读取完的文件。其结果就是，磁盘总是在繁忙地读取不同的文件到内存中。这会带来磁盘和CPU利用率的提升。而且每个线程只需要记录一个文件，因此这种方式也很容易编程实现。

**三、程序响应更快**
将一个单线程应用程序变成多线程应用程序的另一个常见的目的是实现一个响应更快的应用程序。设想一个服务器应用，它在某一个端口监听进来的请求。当一个请求到来时，它去处理这个请求，然后再返回去监听。
服务器的流程如下所述：
```java
while(server is active){
    listen for request ...
    process request ...
}
```
如果一个请求需要占用大量的时间来处理，在这段时间内新的客户端就无法发送请求给服务端。只有服务器在监听的时候，请求才能被接收。
另一种设计是，监听线程把请求传递给工作者线程(worker thread)，然后立刻返回去监听。而工作者线程则能够处理这个请求并发送一个回复给客户端。这种设计如下所述：
```
while(server is active){
    listen for request
    hand request to worker thread
}
```
这种方式，服务端线程迅速地返回去监听。因此，更多的客户端能够发送请求给服务端。这个服务也变得响应更快。

桌面应用也是同样如此。如果你点击一个按钮开始运行一个耗时的任务，这个线程既要执行任务又要更新窗口和按钮，那么在任务执行的过程中，这个应用程序看起来好像没有反应一样。相反，任务可以传递给工作者线程（word thread)。当工作者线程在繁忙地处理任务的时候，窗口线程可以自由地响应其他用户的请求。当工作者线程完成任务的时候，它发送信号给窗口线程。窗口线程便可以更新应用程序窗口，并显示任务的结果。对用户而言，这种具有工作者线程设计的程序显得响应速度更快。


### 缺点
从一个单线程的应用到一个多线程的应用并不仅仅带来好处，它也会有一些代价。不要仅仅为了使用多线程而使用多线程。而应该明确在使用多线程时能多来的好处比所付出的代价大的时候，才使用多线程。如果存在疑问，**应该尝试测量一下应用程序的性能和响应能力**，而不只是猜测。

**一、设计更复杂**
虽然有一些多线程应用程序比单线程的应用程序要简单，但其他的一般都更复杂。在多线程访问共享数据的时候，这部分代码需要特别的注意。线程之间的交互往往非常复杂。不正确的线程同步产生的错误非常难以被发现，并且重现以修复。

**二、上下文切换的开销**
当CPU从执行一个线程切换到执行另外一个线程的时候，它需要先存储当前线程的本地的数据，程序指针等，然后载入另一个线程的本地数据，程序指针等，最后才开始执行。这种切换称为“上下文切换”(“context switch”)。CPU会在一个上下文中执行一个线程，然后切换到另外一个上下文中执行另外一个线程。上下文切换并不廉价。如果没有必要，应该减少上下文切换的发生。

	如果**可运行的线程数**大于cpu的数量，那么操作系统最终会将某个正在运行的线程调度出来，从而使其他线程可以使用cpu。这将导致一次上下文切换，在这个过程中将保存当前运行线程的执行上下文，并将新调度的线程的执行上下文设置为当前上下文，这就是上下文的切换。
	1. 切换上下文是有开销的，而在线程调度过程中需要访问由操作系统和JVM共享的数据结构。应用程序、操作系统以及JVM都使用一组相同的CPU。在JVM和操作系统的代码中消耗越多的CPU时钟周期，应用程序可用的CPU时钟周期就越少。
	2. 当一个线程切入时，它所需要的数据不在当前处理器的本地缓存中，因此切换上下文导致一些缓存缺失，因而线程在首次调度运行时会更加缓慢。

[维基百科-上下文切换](http://en.wikipedia.org/wiki/Context_switch) 

**三、内存同步**
	在synchronized和volatile提供的可见性可能会使用一些特殊指令，即内存栅栏(Memory Barrier)。内存栅栏可以刷新缓存，使缓存无效，刷新硬件的写缓存。内存栅栏对性能可能会有影响，它会抑制一些编译器的优化操作，比如重排序。 不要担心非竞争同步的影响，编译器已经进行了优化。

**编译器的智能分析：下面至少4次调用锁，这是没有必要的，可以锁粒度粗化，只用一个锁**
```java
public String getStoogeNames(){
	List<String> stooges = new Vector<String>();
	stooges.add("1");
	stooges.add("2");
	stooges.add("3");
	return stooges.toString();
}
```

同步竞争可能会影响性能，某个共享内存的争夺。


	
	
	


**三、增加资源消耗**
线程在运行的时候需要从计算机里面得到一些资源。除了CPU，线程还需要一些内存来维持它本地的堆栈。它也需要占用操作系统中一些资源来管理线程。我们可以尝试编写一个程序，让它创建100个线程，这些线程什么事情都不做，只是在等待，然后看看这个程序在运行的时候占用了多少内存。
`jvm`虚拟机中一般只有堆是共享内存，其他大多数例如栈，程序计数器都会增加线程开销



## 三、线程通信基础

### 3.1 实现生产者和消费者模式各种方法

**一、使用基础的wait notify**

典型的第一种并行处理方案，使用一个公共的安全数据结构来满足条件

```
package com.carl.demo.base;

import com.carl.demo.util.ThreadUtils;
import com.google.common.collect.Lists;

import java.util.LinkedList;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 16-3-22.
 */
public class MyStore {

    private int capacity = 10;

    private LinkedList<String> table;

    private final Object putMonitor = new Object();


    public MyStore() {
        this.table = Lists.newLinkedList();
    }

    public MyStore(int capacity) {
        this.table = Lists.newLinkedList();
    }
    //、这2个都别做修改操作
    public void put(String item) {
        synchronized (putMonitor) {
            while (table.size() == capacity ) {
                try {
                    System.out.println("线程" + Thread.currentThread().getName() + "put 堵塞，此时有:" + table.size() + "个元素");
                    putMonitor.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            table.addLast(item);
            System.out.println("线程" + Thread.currentThread().getName() + "put成功，此时有:" + table.size() + "个元素");
            putMonitor.notifyAll();
        }
    }


    public void take() {
        synchronized (putMonitor) {
            while (table.size() == 0) {
                try {
                    System.out.println("线程" + Thread.currentThread().getName() + "take 堵塞，此时有:" + table.size() + "个元素");
                    putMonitor.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            table.removeFirst();
            System.out.println("线程" + Thread.currentThread().getName() + "take 成功，此时有:" + table.size() + "个元素");
            putMonitor.notifyAll();
        }
    }

    public static void main(String[] args) {
        final MyStore myStore = new MyStore();
        ExecutorService pool = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 9; i++) {
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        myStore.put("a");
                        ThreadUtils.sleep(200);
                    }
                }w
            });
        }

        for (int i = 0; i < 1; i++) {
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        myStore.take();
                        ThreadUtils.sleep(200);
                    }
                }
            });
        }

        pool.shutdown();
    }
}

```


## 四、锁

### 4.1 死锁
**一、什么是死锁**

```java
public class TreeNode {
    TreeNode parent   = null;
    List children = new ArrayList();

    public synchronized void addChild(TreeNode child){
        if(!this.children.contains(child)) {
            this.children.add(child);
            child.setParentOnly(this);
        }
    }

    public synchronized void addChildOnly(TreeNode child){
        if(!this.children.contains(child){
            this.children.add(child);
        }
    }

    public synchronized void setParent(TreeNode parent){
        this.parent = parent;
        parent.addChildOnly(this);
    }

    public synchronized void setParentOnly(TreeNode parent){
        this.parent = parent;
    }
}
```

如果有2个线程同时调用:
```
Thread 1: parent.addChild(child); //locks parent
          --> child.setParentOnly(parent);

Thread 2: child.setParent(parent); //locks child
          --> parent.addChildOnly()
```

首先线程1调用parent.addChild(child)。因为addChild()是同步的，所以线程1会对parent对象加锁以不让其它线程访问该对象。

然后线程2调用child.setParent(parent)。因为setParent()是同步的，所以线程2会对child对象加锁以不让其它线程访问该对象。

现在child和parent对象被两个不同的线程锁住了。接下来线程1尝试调用child.setParentOnly()方法，但是由于child对象现在被线程2锁住的，所以该调用会被阻塞。线程2也尝试调用parent.addChildOnly()，但是由于parent对象现在被线程1锁住，导致线程2也阻塞在该方法处。现在两个线程都被阻塞并等待着获取另外一个线程所持有的锁。

注意：像上文描述的，这两个线程需要同时调用parent.addChild(child)和child.setParent(parent)方法，并且是同一个parent对象和同一个child对象，才有可能发生死锁。上面的代码可能运行一段时间才会出现死锁。所以死锁是一种**随机事件**

**更复杂的死锁**
```
Thread 1  locks A, waits for B
Thread 2  locks B, waits for C
Thread 3  locks C, waits for D
Thread 4  locks D, waits for A
```
上面的锁刚好构成一个闭环

**二、如何避免死锁**

1. 加锁顺序: 所有线程保证固定的加锁顺序
2. 加锁时限，超时锁可以避免死锁
3. 死锁检测

**死锁检测**

死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

每当一个线程获得了锁，会在线程和锁相关的数据结构中（map、graph等等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。

当一个线程请求锁失败时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程A请求锁7，但是锁7这个时候被线程B持有，这时线程A就可以检查一下线程B是否已经请求了线程A当前所持有的锁。如果线程B确实有这样的请求，那么就是发生了死锁（线程A拥有锁1，请求锁7；线程B拥有锁7，请求锁1）。

当然，死锁一般要比两个线程互相持有对方的锁这种情况要复杂的多。线程A等待线程B，线程B等待线程C，线程C等待线程D，线程D又在等待线程A。线程A为了检测死锁，它需要递进地检测所有被B请求的锁。从线程B所请求的锁开始，线程A找到了线程C，然后又找到了线程D，发现线程D请求的锁被线程A自己持有着。这是它就知道发生了死锁。

下面是一幅关于四个线程（A,B,C和D）之间锁占有和请求的关系图。像这样的数据结构就可以被用来检测死锁。

![enter image description here](http://ifeve.com/wp-content/uploads/2013/03/deadlock-detection-graph.png)


- 一个可行的做法是**释放所有锁，回退，并且等待一段随机的时间后重试**。这个和简单的加锁超时类似，不一样的是只有死锁已经发生了才回退，而不会是因为加锁的请求超时了。虽然有回退和等待，但是如果有大量的线程竞争同一批锁，它们还是会重复地死锁（编者注：原因同超时类似，不能从根本上减轻竞争）。

- 一个更好的方案是给**这些线程设置优先级**，让一个（或几个）线程回退，剩下的线程就像没发生死锁一样继续保持着它们需要的锁。如果赋予这些线程的优先级是固定不变的，同一批线程总是会拥有更高的优先级。为避免这个问题，可以在死锁发生的时候设置随机的优先级。



### 4.2 饥饿和公平性

如果一个线程得不到CPU的运行时间（没有运行的机会），这种状态被称为饥饿，解决饥饿的方案被称之为“公平性” – 即所有线程均能公平地获得运行机会。

**原因：**

1. **高优先级的线程吞噬所有的低优先级的CPU时间**
	你能为每个线程设置独自的线程优先级，优先级越高的线程获得的CPU时间越多，线程优先级值设置在1到10之间，而这些优先级值所表示行为的准确解释则依赖于你的应用运行平台。对大多数应用来说，你最好是不要改变其优先级值。
2. **线程被永久堵塞在一个等待进入同步块**
	 Java的同步代码区也是一个导致饥饿的因素。Java的同步代码区对哪个线程允许进入的次序没有任何保障。这就意味着理论上存在一个试图进入该同步区的线程处于被永久堵塞的风险，因为其他线程总是能持续地先于它获得访问，这即是“饥饿”问题，而一个线程被“饥饿致死”正是因为它得不到CPU运行时间的机会。
3. **线程等待一个本身处于永久等待完成的对象**(比如调用这个对象的`wait`方法)
	如果多个线程处在wait()方法执行上，而对其调用notify()不会保证哪一个线程会获得唤醒，任何线程都有可能处于继续等待的状态。因此存在这样一个风险：一个等待线程从来得不到唤醒，因为其他等待线程总是能被获得唤醒。

**思路：**
1. 使用锁，而不是同步块
2. 公平锁
3. 性能方面


**如何实现一个普通的锁**

```java
package com.carl.demo.concurrency;

/**
 * Created by carl on 16-3-25.
 */
public class SimpleLock {
    private Thread lockingThread = null;
    private final Object monitor = new Object();
    private boolean isLocked = false;


    public void lock() throws InterruptedException {
        synchronized (monitor) {
            while (isLocked) {
                monitor.wait();
            }
            lockingThread = Thread.currentThread();
            isLocked = true;
        }
    }


    public void unlock() {
        synchronized (monitor) {
            if (Thread.currentThread() != lockingThread) {
                throw new IllegalMonitorStateException("Calling thread can't unlock this");
            }
            isLocked = false;
            lockingThread = null;
            monitor.notify();
        }
    }

}

```



**如何实现一个公平锁**

```java
package com.carl.demo.concurrency;

import com.carl.demo.util.ThreadUtils;

import java.util.LinkedList;

/**
 * Created by carl on 16-3-25.
 */
public class FairLock {
    private boolean isLocked = false;
    private Thread lockingThread = null;
    private LinkedList<LockObject> waitingThreads = new LinkedList<>();

    public void lock() throws InterruptedException {
        LockObject lockObject = new LockObject();
        boolean isLockedForThisThread = true;
        //加入队列中
        synchronized (this) {
            waitingThreads.add(lockObject);
        }

        while (isLockedForThisThread) {
            synchronized (this) {
                //没有锁而且是队列中第一个位置才允许go，否则都去等待
                isLockedForThisThread = isLocked || waitingThreads.getFirst() != lockObject;
                if (
                        !isLocked
                                &&
                                waitingThreads.getFirst() == lockObject
                        ) {
                    isLocked = true;
                    waitingThreads.remove(lockObject);
                    lockingThread = Thread.currentThread();
                    return;
                }
            }
            try {
                System.out.println(Thread.currentThread().getName()+" is waiting");
                lockObject.doWait();
            } catch (InterruptedException e) {
                synchronized (this) {
                    waitingThreads.remove(lockObject);
                }
                throw e;
            }
        }

    }


    public void unlock() {
        synchronized (this) {
            if (lockingThread != Thread.currentThread()) {
                throw new IllegalMonitorStateException("Calling thread has not locked this lock");
            }
            isLocked = false;
            lockingThread = null;
            if (waitingThreads.size() > 0) {
                waitingThreads.getFirst().doNotify();
            }
        }
    }


    static class MyThread extends Thread {
        private FairLock lock;
        private String name ;

        MyThread(FairLock lock, String name) {
            super(name);
            this.lock = lock;
            this.name = name;
        }

        @Override
        public void run() {
            try {
                lock.lock();
                ThreadUtils.sleep(1000);
                System.out.println(Thread.currentThread().getName());
                lock.unlock();
            } catch (InterruptedException e) {
            }
        }
    }

    public static void main(String[] args) {
        final FairLock lock = new FairLock();
        new MyThread(lock, "线程1").start();
        ThreadUtils.sleep(100L);
        new MyThread(lock, "线程2").start();
        ThreadUtils.sleep(100L);
        new MyThread(lock, "线程3").start();
        ThreadUtils.sleep(100L);
        new MyThread(lock, "线程4").start();
        ThreadUtils.sleep(100L);
        new MyThread(lock, "线程5").start();
        ThreadUtils.sleep(100L);
        new MyThread(lock, "线程6").start();
        ThreadUtils.sleep(100L);
    }


}



package com.carl.demo.concurrency;

/**
 * Created by carl on 16-3-25.
 */
public class LockObject {
    private boolean isNotified = false;

    public void doWait() throws InterruptedException {
        synchronized (this) {
            //只要没有被唤醒一直处于等待状态
            while (!isNotified) {
                wait();
            }
            isNotified = false;
        }
    }

    public void doNotify() {
        synchronized (this) {
            isNotified = true;
            notify();
        }
    }

}
```

> 1. 第一个线程进入时，isLocked为false，将isLocked设置为true
> 2. 第二个线程进入时，isLocked为true，waitting
> 3. 第三个线程进入时，isLocked为true，waitting
> 4. 只有队列中第一个位置中的线程会被解锁
> 5. 将每个线程单独锁定，可以指定解锁顺序
> 6. 不支持可以重入


### 4.3 嵌套管程锁死


```java
public class Lock{
        protected MonitorObject monitorObject = new MonitorObject();
        protected boolean isLocked = false;

        public void lock() throws InterruptedException{
            synchronized(this){
                while(isLocked){
                    synchronized(this.monitorObject){
                        this.monitorObject.wait();
                    }
                }
                isLocked = true;
            }
        }

        public void unlock(){
            
            synchronized(this){
                
                this.isLocked = false;
                
                synchronized(this.monitorObject){
                    
                    this.monitorObject.notify();
                    
                }
                
            }
            
        }
        
    }

```

	线程1获得A对象的锁。
	线程1获得对象B的锁（同时持有对象A的锁）。
	线程1决定等待另一个线程的信号再继续。
	线程1调用B.wait()，从而释放了B对象上的锁，但仍然持有对象A的锁。
	
	线程2需要同时持有对象A和对象B的锁，才能向线程1发信号。
	线程2无法获得对象A上的锁，因为对象A上的锁当前正被线程1持有。
	线程2一直被阻塞，等待线程1释放对象A上的锁。
	
	线程1一直阻塞，等待线程2的信号，因此，不会释放对象A上的锁，
		而线程2需要对象A上的锁才能给线程1发信号……


假设有2把锁A，B A嵌套B，有两个方法类似于以上代码，方法1，方法2

**1.锁的顺序一致**
**2.方法1在被嵌套的锁B中有wait方法**
**3.方法2要获取锁A，但是方法1的wait方法只释放了锁B**
**4.方法1要被唤醒需要方法2执行完，啧啧，挂了**


### 4.4 可重入锁 Reentrant Lock

**一、简易锁**

一个简单的锁可以实现synchronized类似的功能，像最开始讲到的，它可以保证可以保证在同一时刻仅有一个线程可以进入代码的临界区。
可以保证CPU Memory Cache中的值一旦修改就写入RAM么?

代码如开始所示，用自旋锁解决假唤醒问题（没有来信号，由于意外就醒了过来）

```java
package com.carl.demo.concurrency;

/**
 * Created by carl on 16-3-25.
 */
public class SimpleLock {
    private Thread lockingThread = null;
    private final Object monitor = new Object();
    private boolean isLocked = false;


    public void lock() throws InterruptedException {
        synchronized (monitor) {
            while (isLocked) {
                monitor.wait();
            }
            lockingThread = Thread.currentThread();
            isLocked = true;
        }
    }


    public void unlock() {
        synchronized (monitor) {
            if (Thread.currentThread() != lockingThread) {
                throw new IllegalMonitorStateException("Calling thread can't unlock this");
            }
            isLocked = false;
            lockingThread = null;
            monitor.notify();
        }
    }

}
```

**二、可重入锁**

```java
package com.carl.demo.concurrency;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Created by carl on 16-3-25.
 */
public class ReentrantLock {
    private boolean isLocked = false;
    private Thread lockingThread = null;
    private int lockCount = 0;

    public void lock() throws InterruptedException {
        synchronized (this) {
            Thread callingThread = Thread.currentThread();
            while (isLocked && lockingThread != callingThread) {
                System.out.println("---------------------------当前锁有:"+lockCount+"层");
                wait();
            }
            isLocked = true;
            lockCount++;
            lockingThread = Thread.currentThread();
        }
    }


    public void unlock() {
        synchronized (this) {
            if (Thread.currentThread() == this.lockingThread) {
                lockCount--;
            }
            if (lockCount == 0) {
                isLocked = false;
                notify();
            }
        }
    }

    static int count = 0;

    static class MyThread extends Thread {

        private ReentrantLock lock;

        MyThread(ReentrantLock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            try {
                lock.lock();
                lock.lock();
                lock.lock();
                lock.lock();
            } catch (InterruptedException e) {
                e.printStackTrace();
                return;
            }
            try {
                for (int i = 0; i < 100; i++) {
                    System.out.println(count++);
                }
            } finally {
                lock.unlock();
                lock.unlock();
                lock.unlock();
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(10);
        final ReentrantLock lock = new ReentrantLock();
        for (int i = 0; i < 10; i++) {
            pool.execute(new MyThread(lock));
        }
        pool.shutdown();
    }

}
```


### 4.5 读写锁

下面实现一个最简易的读写锁，需要满足以下需求

**问题一：** 写锁饥饿问题，当有大量读请求时，写请求若没有设置优先级（即是设置了优先级）也不一定能保证写锁的优先问题，**写锁是要比读锁拥有更高的优先级**


**问题二：** 读写锁分离问题，读锁和读锁不互斥，读锁和写锁互斥，写锁和写锁互斥

```java
package com.carl.demo.concurrency;

import com.carl.demo.util.ThreadUtils;

/**
 * Created by carl on 16-3-25.
 */
public class ReadWriteLock {

    private int readers;
    private int writers;
    private int writeRequests;


    public void lockRead() throws InterruptedException {
        synchronized (this) {
            while (writers > 0 || writeRequests > 0) {
                wait();
            }
            readers++;
        }

    }

    public void lockWrite() throws InterruptedException {
        synchronized (this) {
            writeRequests++;
            while (writers > 0 || readers > 0) {
                wait();
            }
            writeRequests--;
            writers++;
        }
    }

    public void unlockRead() {
        synchronized (this) {
            readers--;
            notifyAll();
        }
    }

    public void unlockWrite() {
        synchronized (this) {
            writers--;
            notifyAll();
        }
    }


    static class MyReadThread extends Thread {
        private ReadWriteLock readWriteLock;

        private String name;

        MyReadThread(ReadWriteLock readWriteLock, String name) {
            super(name);
            this.readWriteLock = readWriteLock;
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " begin reading");
            try {
                readWriteLock.lockRead();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is reading");
            ThreadUtils.sleep(2000L);
            readWriteLock.unlockRead();
        }
    }

    static class MyWriteThread extends Thread {
        private ReadWriteLock readWriteLock;

        private String name;

        MyWriteThread(ReadWriteLock readWriteLock, String name) {
            super(name);
            this.readWriteLock = readWriteLock;
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " begin writing");
            try {
                readWriteLock.lockWrite();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is writing");
            ThreadUtils.sleep(2000L);
            readWriteLock.unlockWrite();
        }
    }


    public static void main(String[] args) {

        final ReadWriteLock lock = new ReadWriteLock();
        new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 20; i++) {
                    new MyReadThread(lock, "r" + i).start();
                }
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                new MyWriteThread(lock, "w1").start();
                new MyWriteThread(lock, "w2").start();
            }
        }.start();
    }
}


```


> 下面来分离实现，假设有多个读取线程在访问 r1 r2 r3 r4 ......，有少数写锁w1,w2在访问
> **第一种情况：** 此时写锁r1获取优先，此时没有写锁 ,write=0，否则会等待到write执行完毕为0，此时写锁r1执行完毕，read+1，其他读线程r2,r3,r4...可以继续访问，如果是w1，w2会等待，但是关键是**writeRequest+1会执行**，这会堵塞其他的读锁，从而满足了需求1，写锁拥有更高的优先级。
> **第二种情况：** 此时读锁w1获取优先，此时w2,r1,r2.....等都会堵塞，这满足了需求2，各种互斥


### 4.6 可重入读写锁

四个问题：
**1 读锁重入（本身读锁就是不互斥的）**
**2 写锁重入**
**3 读锁可获得写锁**
**4 写锁可以重入写锁**

```java
package com.carl.demo.concurrency;

import java.util.HashMap;
import java.util.Map;

/**
 * Created by carl on 16-3-30.
 */
public class FinalReentranReadWriteLock {
    private Map<Thread, Integer> readingThreads = new HashMap<Thread, Integer>();


    private int writeAccesses = 0;

    private int writeRequests = 0;

    private Thread writingThread = null;


    public synchronized void lockRead()

            throws InterruptedException {

        Thread callingThread = Thread.currentThread();

        while (!canGrantReadAccess(callingThread)) {

            wait();

        }


        readingThreads.put(callingThread,

                (getReadAccessCount(callingThread) + 1));

    }


    private boolean canGrantReadAccess(Thread callingThread) {
        // 拥有写锁可以重入读锁-》写锁的降级
        if (isWriter(callingThread)) return true;

        if (hasWriter()) return false;

        if (isReader(callingThread)) return true;

        if (hasWriteRequests()) return false;

        return true;

    }


    public synchronized void unlockRead() {

        Thread callingThread = Thread.currentThread();

        if (!isReader(callingThread)) {

            throw new IllegalMonitorStateException(

                    "Calling Thread does not" +

                            " hold a read lock on this ReadWriteLock");

        }

        int accessCount = getReadAccessCount(callingThread);

        if (accessCount == 1) {

            readingThreads.remove(callingThread);

        } else {

            readingThreads.put(callingThread, (accessCount - 1));

        }

        notifyAll();

    }


    public synchronized void lockWrite()

            throws InterruptedException {

        writeRequests++;

        Thread callingThread = Thread.currentThread();

        while (!canGrantWriteAccess(callingThread)) {

            wait();

        }

        writeRequests--;

        writeAccesses++;

        writingThread = callingThread;

    }


    public synchronized void unlockWrite()

            throws InterruptedException {

        if (!isWriter(Thread.currentThread())) {

            throw new IllegalMonitorStateException(

                    "Calling Thread does not" +

                            " hold the write lock on this ReadWriteLock");

        }

        writeAccesses--;

        if (writeAccesses == 0) {

            writingThread = null;

        }

        notifyAll();

    }


    private boolean canGrantWriteAccess(Thread callingThread) {
        //可以读锁可以重入写锁-》读锁的升级
        if (isOnlyReader(callingThread)) return true;

        if (hasReaders()) return false;

        if (writingThread == null) return true;

        //判断是否可以重入写锁
        if (!isWriter(callingThread)) return false;

        return true;

    }


    private int getReadAccessCount(Thread callingThread) {

        Integer accessCount = readingThreads.get(callingThread);

        if (accessCount == null) return 0;

        return accessCount.intValue();

    }


    private boolean hasReaders() {

        return readingThreads.size() > 0;

    }


    private boolean isReader(Thread callingThread) {

        return readingThreads.get(callingThread) != null;

    }


    private boolean isOnlyReader(Thread callingThread) {

        return readingThreads.size() == 1 &&

                readingThreads.get(callingThread) != null;

    }


    private boolean hasWriter() {

        return writingThread != null;

    }


    private boolean isWriter(Thread callingThread) {

        return writingThread == callingThread;

    }


    private boolean hasWriteRequests() {

        return this.writeRequests > 0;

    }

}


```

关键代码
```
private boolean canGrantReadAccess(Thread callingThread) {
        // 拥有写锁可以重入读锁-》写锁的降级
        if (isWriter(callingThread)) return true;

        if (hasWriter()) return false;

        if (isReader(callingThread)) return true;

        if (hasWriteRequests()) return false;

        return true;

    }

 private boolean canGrantWriteAccess(Thread callingThread) {
        //可以读锁可以重入写锁-》读锁的升级
        if (isOnlyReader(callingThread)) return true;

        if (hasReaders()) return false;

        if (writingThread == null) return true;

        //判断是否可以重入写锁
        if (!isWriter(callingThread)) return false;

        return true;
    }
```

### 4.7 重入锁死

这种问题只是针对于不可重入锁:
	如果一个线程在两次调用lock()间没有调用unlock()方法，那么第二次调用lock()就会被阻塞，这就出现了重入锁死。这种死法和嵌套锁死非常类似，相同顺序的锁，啧啧 ！

避免重入锁死有两个选择：

1. 编写代码时避免再次获取已经持有的锁
2. 使用可重入锁

至于哪个选择最适合你的项目，得视具体情况而定。**可重入锁通常没有不可重入锁那么好的表现，而且实现起来复杂**，但这些情况在你的项目中也许算不上什么问题。无论你的项目用锁来实现方便还是不用锁方便，可重入特性都需要根据具体问题具体分析。



## 五、多线程常用工具

### 5.1 Semphore

Semaphore（信号量） 是一个线程同步结构，用于在线程间传递信号，以避免出现信号丢失（译者注：下文会具体介绍），或者像锁一样用于保护一个关键区域。自从5.0开始，jdk在java.util.concurrent包里提供了Semaphore 的官方实现，因此大家不需要自己去实现Semaphore。但是还是很有必要去熟悉如何使用`Semaphore`及其背后的原理



```java
package com.carl.demo.concurrency;

import com.carl.demo.util.ThreadUtils;

/**
 * Created by carl
 */
public class MyCountDownLatch {
    private int signals = 2;

    public synchronized void countDown() {
        if (signals > 0)
            signals--;
        if(signals == 0){
            notifyAll();
        }
    }


    public synchronized void await() throws InterruptedException {
        while (this.signals != 0) {
            wait();
        }
    }

    static class MyThread extends Thread {
        private MyCountDownLatch myCountDownLatch;

        private MyThread(MyCountDownLatch myCountDownLatch) {
            this.myCountDownLatch = myCountDownLatch;
        }

        @Override
        public void run() {
            System.out.println("2s后我要解开锁");
            ThreadUtils.sleep(2000L);
            myCountDownLatch.countDown();
        }
    }

    public static void main(String[] args) throws Exception {
        final MyCountDownLatch myCountDownLatch = new MyCountDownLatch();
        System.out.println("哎哟，卡一下");
        new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 1; i++) {
                    new MyThread(myCountDownLatch).start();
                }
            }
        }.start();
        myCountDownLatch.await();
        System.out.println("过了.....");
    }
}

```


### 5.2 堵塞队列

阻塞队列与普通队列的区别在于，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其他的线程往空的队列插入新的元素。同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其他的线程使队列重新变得空闲起来，如从队列中移除一个或者多个元素，或者完全清空队列，下图展示了如何通过阻塞队列来合作：

![enter image description here](http://tutorials.jenkov.com/images/java-concurrency-utils/blocking-queue.png)


```java
package com.carl.demo.concurrency;

import com.carl.demo.util.ThreadUtils;
import com.google.common.collect.Lists;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import java.util.LinkedList;

/**
 * Created by carl on 16-3-31.
 */
public class MyBlockingQueue<T> {
    private LinkedList<T> queue = Lists.newLinkedList();
    private int limit = 10;
    private ReentrantLock lock;
    private final Condition notEmpty;
    private final Condition notFull;


    public MyBlockingQueue(int limit) {
        this.limit = limit;
        lock = new ReentrantLock(true);
        notEmpty = lock.newCondition();
        notFull = lock.newCondition();
    }

    public void put(T t) throws InterruptedException {
        lock.lockInterruptibly();
        System.out.println(ThreadUtils.getCurrentThreadName() + " put .." + t + "-start");
        try {
            while (limit == queue.size()) {
                System.out.println(ThreadUtils.getCurrentThreadName() + " put is waiting");
                notFull.await();
            }
            enqueue(t);
            System.out.println(ThreadUtils.getCurrentThreadName() + " put .." + t + "-success,now is " + queue.size());
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lockInterruptibly();
        System.out.println(ThreadUtils.getCurrentThreadName() + " take-start");
        try {
            while (queue.size() == 0) {
                System.out.println(ThreadUtils.getCurrentThreadName() + " take is waiting");
                notEmpty.await();
            }
            T ret = dequeue();
            System.out.println(ThreadUtils.getCurrentThreadName() + " take-success,now is " + queue.size());
            return ret;
        } finally {
            lock.unlock();
        }
    }

    private void enqueue(T t) throws InterruptedException {
        this.queue.addLast(t);
        // 一次加入只唤醒一个
        notEmpty.signal();
    }


    private T dequeue() throws InterruptedException {
        T ret = this.queue.remove(0);
        // 移除后有空的了
        notFull.signal();
        return ret;
    }


    public static void main(String[] args) throws Exception {
        final MyBlockingQueue<String> queue = new MyBlockingQueue(7);
        ExecutorService pool = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 9; i++) {
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        try {
                            queue.put("a");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        ThreadUtils.sleep(200);
                    }
                }
            });
        }

        for (int i = 0; i < 1; i++) {
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        try {
                            queue.take();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        ThreadUtils.sleep(200);
                    }
                }
            });
        }

        pool.shutdown();
    }
}

```








## 六、线程池

### 6.1 基础篇

线程池（Thread Pool）对于限制应用程序中同一时刻运行的线程数很有用。因为每启动一个新线程都会有相应的性能开销，每个线程都需要给栈分配一些内存等等。

我们可以把并发执行的任务传递给一个线程池，来替代为每个并发执行的任务都启动一个新的线程。只要池里有空闲的线程，任务就会分配给一个线程执行。在线程池的内部，任务被插入一个阻塞队列（Blocking Queue ），线程池里的线程会去取这个队列里的任务。当一个新任务插入队列时，一个空闲线程就会成功的从队列中取出任务并且执行它。


线程池经常应用在多线程服务器上。每个通过网络到达服务器的连接都被包装成一个任务并且传递给线程池。线程池的线程会并发的处理连接上的请求。以后会再深入有关 Java 实现多线程服务器的细节。

Java 5 在 java.util.concurrent 包中自带了内置的线程池，所以你不用非得实现自己的线程池。你可以阅读我写的 java.util.concurrent.ExecutorService 的文章以了解更多有关内置线程池的知识。不过无论如何，知道一点关于线程池实现的知识总是有用的。







## 七、 无堵塞算法


### 7.1 CAS

在研究无锁之前，我们需要首先了解一下CAS原子操作——Compare & Set，或是 Compare & Swap，现在几乎所image有的CPU指令都支持CAS的原子操作，X86下对应的是 CMPXCHG 汇编指令。
大家应该还记得操作系统里面关于“原子操作”的概念，一个操作是原子的(atomic)，如果这个操作所处的层(layer)的更高层不能发现其内部实现与结构。原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序是不可以被打乱，或者切割掉只执行部分。有了这个原子操作这个保证我们就可以实现无锁了。


CAS原子操作在维基百科中的代码描述如下:
```
: int compare_and_swap(int* reg, int oldval, int newval)
   2: {
   3:   ATOMIC();
   4:   int old_reg_val = *reg;
   5:   if (old_reg_val == oldval)
   6:      *reg = newval;
   7:   END_ATOMIC();
   8:   return old_reg_val;
   9: }
```

它可以变种为直接返回是否更新成功
``` c
 1: bool compare_and_swap (int *accum, int *dest, int newval)
   2: {
   3:   if ( *accum == *dest ) {
   4:       *dest = newval;
   5:       return true;
   6:   }
   7:   return false;
   8: }
```


除了CAS还有以下几种原子操作：

Fetch And Add，一般用来对变量做 +1 的原子操作。
``` c
   1: << atomic >>
   2: function FetchAndAdd(address location, int inc) {
   3:     int value := *location
   4:     *location := value + inc
   5:     return value
   6: }
```

 
Test-and-set，写值到某个内存位置并传回其旧值。汇编指令BST。
```c
 1: #define LOCKED 1
   2:  
   3: int TestAndSet(int* lockPtr) {
   4:     int oldValue;
   5:  
   6:     // Start of atomic segment
   7:     // The following statements should be interpreted as pseudocode for
   8:     // illustrative purposes only.
   9:     // Traditional compilation of this code will not guarantee atomicity, the
  10:     // use of shared memory (i.e. not-cached values), protection from compiler
  11:     // optimization, or other required properties.
  12:     oldValue = *lockPtr;
  13:     *lockPtr = LOCKED;
  14:     // End of atomic segment
  15:  
  16:     return oldValue;
  17: }
```
  
 
Test and Test-and-set，用来实现多核环境下互斥锁，
```c
	1: boolean locked := false // shared lock variable
   2: procedure EnterCritical() {
   3:   do {
   4:     while (locked == true) skip // spin until lock seems free
   5:   } while TestAndSet(locked) // actual atomic locking
   6: }
```
   
### 7.2 堵塞与非堵塞

（1）一个阻塞并发算法一般分下面两步：

- 执行线程请求的操作
- 阻塞线程直到可以安全地执行操作

很多算法和并发数据结构都是阻塞的。例如，`java.util.concurrent.BlockingQueue`的不同实现都是阻塞数据结构。如果一个线程要往一个阻塞队列中插入一个元素，队列中没有足够的空间，执行插入操作的线程就会阻塞直到队列中有了可以存放插入元素的空间。

![enter image description here](https://camo.githubusercontent.com/720fd34b938c645d6d42b41e5d694fb9d2d71a61/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d312e706e67)



（2）非阻塞并发算法
一个非阻塞并发算法一般包含下面两步：

- 执行线程请求的操作
- 通知请求线程操作不能被执行
Java也包含几个非阻塞数据结构。`AtomicBoolean,AtomicInteger,AtomicLong,AtomicReference`都是非阻塞数据结构的例子。

![enter image description here](https://camo.githubusercontent.com/3f3647ee8e4af4140e5414eb296d8f2d4e83a3fb/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d322e706e67)


阻塞算法和非阻塞算法的主要不同在于上面**两部分描述的它们的行为的第二步**。换句话说，它们之间的不同在于当请求操作不能够执行时阻塞算法和非阻塞算法会怎么做。

	阻塞算法会阻塞线程知道请求操作可以被执行。
	而非阻塞算法会通知请求线程操作不能够被执行，并返回。
	
	1.一个使用了阻塞算法的线程可能会阻塞直到有可能去处理请求:
	通常，其它线程的动作使第一个线程执行请求的动作成为了可能。 如果，由于某些原因线程被阻塞在程序某处，因此不能让第一个线程的请求动作被执行，第一个线程会阻塞——可能一直阻塞或者直到其他线程执行完必要的动作。
	
	例如，如果一个线程产生往一个已经满了的阻塞队列里插入一个元素，这个线程就会阻塞，直到其他线程从这个阻塞队列中取走了一些元素。如果由于某些原因，从阻塞队列中取元素的线程假定被阻塞在了程序的某处，那么，尝试往阻塞队列中添加新元素的线程就会阻塞，要么一直阻塞下去，要么直到从阻塞队列中取元素的线程最终从阻塞队列中取走了一个元素。
	2.一个使用了非堵塞算法的线程会通知请求线程该操作不能够执行


### 7.3 非阻塞并发数据结构

在一个多线程系统中，线程间通常通过一些数据结构”交流“。例如可以是任何的数据结构，从变量到更高级的俄数据结构（队列，栈等）。为了确保正确，并发线程在访问这些数据结构的时候，这些数据结构必须由一些并发算法来保证。这些并发算法让这些数据结构成为并发数据结构。

如果某个算法确保一个并发数据结构是阻塞的，它就被称为是一个阻塞算法。这个数据结构也被称为是一个阻塞，并发数据结构。

如果某个算法确保一个并发数据结构是非阻塞的，它就被称为是一个非阻塞算法。这个数据结构也被称为是一个非阻塞，并发数据结构。

每个并发数据结构被设计用来支持一个特定的通信方法。使用哪种并发数据结构取决于你的通信需要。在接下里的部分，我会引入一些非阻塞并发数据结构，并讲解它们各自的适用场景。通过这些并发数据结构工作原理的讲解应该能在非阻塞数据结构的设计和实现上一些启发。

**一、 浅谈使用valatile**

（1）volatile的简单结构

**注意：此时要保证最多只有一个写线程**

	当多个线程在一个共享变量上执行一个 read-update-write 的顺序操作时才会发生竞态条件。如果你只有一个线程在执行一个 raed-update-write 的顺序操作，其他线程都在执行读操作，将不会发生竞态条件。


```java
public class SingleWriterCounter{
    private volatile long count = 0;

    /**
     *Only one thread may ever call this method
     *or it will lead to race conditions
     */
     public void inc(){
         this.count++;
     }

     /**
      *Many reading threads may call this method
      *@return
      */
      public long count(){
          return this.count;
      }
}
```
	
多个线程访问同一个`Counter`实例，只要仅有一个线程调用`inc()`方法，这里，我不是说在某一时刻一个线程，我的意思是，仅有相同的，单个的线程被允许去调用inc()>方法。多个线程可以调用count()方法。这样的场景将不会发生任何竞态条件。

下图，说明了线程是如何访问count这个volatile变量的。

![enter image description here](https://camo.githubusercontent.com/74cd8d994d0be5bee55c55c72b3ce86a673cf86f/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d332e706e67)


（2）基于volatile变量更高级的数据结构
使用多个volatile变量去创建数据结构是可以的，构建出的数据结构中每一个volatile变量仅被一个单个的线程写，被多个线程读。每个volatile变量可能被一个不同的线程写（但仅有一个）。使用像这样的数据结构多个线程可以使用这些volatile变量以一个非阻塞的方法彼此发送信息。


```java
public class DoubleWriterCounter{
    private volatile long countA = 0;
    private volatile long countB = 0;

    /**
     *Only one (and the same from thereon) thread may ever call this method,
     *or it will lead to race conditions.
     */
     public void incA(){
         this.countA++;
     }

     /**
      *Only one (and the same from thereon) thread may ever call this method, 
      *or it will  lead to race conditions.
      */
      public void incB(){
          this.countB++;
      }

      /**
       *Many reading threads may call this method
       */
      public long countA(){
          return this.countA;
      }

     /**
      *Many reading threads may call this method
      */
      public long countB(){
          return this.countB;
      }
}
```


	如你所见，DoubleWriterCoounter现在包含两个volatile变量以及两对自增和读方法。
	
	在某一时刻，仅有一个单个的线程可以调用inc()，仅有一个单个的线程可以访问incB()。不过不同的线程可以同时调用incA()和incB()。countA()和countB()可以被多个线程调用。这将不会引发竞态条件。
	
	DoubleWriterCoounter可以被用来比如线程间通信。countA和countB可以分别用来存储生产的任务数和消费的任务数。下图，展示了两个线程通过类似于上面的一个数据结构进行通信的。

![enter image description here](https://camo.githubusercontent.com/97a2b4ea28a6d8fb79460d847acf047603c3b975/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d342e706e67)



**二、实现CAS**

>如果你确实需要多个线程区写同一个共享变量，volatile变量是不合适的。你将会需要一些类型的排它锁（悲观锁）访问这个变量。下面代码演示了使用Java中的同步块进行排他访问的。



```java
public class SynchronizedCounter{
    long count = 0;

    public void inc(){
        synchronized(this){
            count++;
        }
    }

    public long count(){
        synchronized(this){
            return this.count;
        }
    }
}
```


我们可以使用一种Java的原子变量来代替这两个同步块。在这个例子是AtomicLong。下面是SynchronizedCounter类的AtomicLong实现版本。


```java
import java.util.concurrent.atomic.AtomicLong;

public class AtomicLong{
    private AtomicLong count = new AtomicLong(0);

    public void inc(){
        boolean updated = false;
        while(!updated){
            long prevCount = this.count.get();
            updated = this.count.compareAndSet(prevCount, prevCount + 1);
        }
    }

    public long count(){
        return this.count.get();
    }
}
```

**为什么称它为乐观锁**

	上一部分展现的代码被称为乐观锁（optimistic locking）。乐观锁区别于传统的锁，传统锁有时也被称为悲观锁。传统的锁会使用同步块或其他类型的锁阻塞对临界区域的访问。一个同步块或锁可能会导致线程挂起。
	
	乐观锁允许所有的线程在不发生阻塞的情况下创建一份共享内存的拷贝。这些线程接下来可能会对它们的拷贝进行修改，并企图把它们修改后的版本写回到共享内存中。如果没有其它线程对共享内存做任何修改， CAS操作就允许线程将它的变化写回到共享内存中去。如果，另一个线程已经修改了共享内存，这个线程将不得不再次获得一个新的拷贝，在新的拷贝上做出修改，并尝试再次把它们写回到共享内存中去。
	
	称之为“乐观锁”的原因就是，线程获得它们想修改的数据的拷贝并做出修改，在乐观的假在此期间没有线程对共享内存做出修改的情况下。当这个乐观假设成立时，这个线程仅仅在无锁的情况下完成共享内存的更新。当这个假设不成立时，线程所做的工作就会被丢弃，但同样不使用锁。
	
	乐观锁使用于共享内存竞用不是非常高的情况。如果共享内存上的内容非常多，仅仅因为更新共享内存失败，就用浪费大量的CPU周期用在拷贝和修改上。但是，如果在共享内存上有大量的内容，无论如何，你都要把你的代码设计的产生的争用更低。

**乐观锁是非阻塞的**

	我们这里提到的乐观锁机制是非阻塞的。如果一个线程获得了一份共享内存的拷贝，当尝试修改时，发生了阻塞，其它线程去访问这块内存区域不会发生阻塞。
	
	对于一个传统的加锁/解锁模式，当一个线程持有一个锁时，其它所有的线程都会一直阻塞直到持有锁的线程再次释放掉这个锁。如果持有锁的这个线程被阻塞在某处，这个锁将很长一段时间不能被释放，甚至可能一直不能被释放。


**不可替换的数据结构**

	简单的CAS乐观锁可以用于共享数据结果，这样一来，整个数据结构都可以通过一个单个的CAS操作被替换成为一个新的数据结构。尽管，使用一个修改后的拷贝来替换真个数据结构并不总是可行的。
	
	假设，这个共享数据结构是队列。每当线程尝试从向队列中插入或从队列中取出元素时，都必须拷贝这个队列然后在拷贝上做出期望的修改。我们可以通过使用一个AtomicReference来达到同样的目的。拷贝引用，拷贝和修改队列，尝试替换在AtomicReference中的引用让它指向新创建的队列。
	
	然而，一个大的数据结构可能会需要大量的内存和CPU周期来复制。这会使你的程序占用大量的内存和浪费大量的时间再拷贝操作上。这将会降低你的程序的性能，特别是这个数据结构的竞用非常高情况下。更进一步说，一个线程花费在拷贝和修改这个数据结构上的时间越长，其它线程在此期间修改这个数据结构的可能性就越大。如你所知，如果另一个线程修改了这个数据结构在它被拷贝后，其它所有的线程都不等不再次执行 拷贝-修改 操作。这将会增大性能影响和内存浪费，甚至更多。
	
	接下来的部分将会讲解一种实现非阻塞数据结构的方法，这种数据结构可以被并发修改，而不仅仅是拷贝和修改。


### 7.4 ABA问题

**概念：**

	ABA：如果另一个线程修改V值假设原来是A，先修改成B，再修改回成A。当前线程的CAS操作无法分辨当前V值是否发生过变化。

**共享预期的修改**

	用来替换拷贝和修改整个数据结构，一个线程可以共享它们对共享数据结构预期的修改。一个线程向对修改某个数据结构的过程变成了下面这样：
	
	检查是否另一个线程已经提交了对这个数据结构提交了修改
	如果没有其他线程提交了一个预期的修改，创建一个预期的修改，然后向这个数据结构提交预期的修改
	执行对共享数据结构的修改
	移除对这个预期的修改的引用，向其它线程发送信号，告诉它们这个预期的修改已经被执行
	如你所见，第二步可以阻塞其他线程提交一个预期的修改。因此，第二步实际的工作是作为这个数据结构的一个锁。如果一个线程已经成功提交了一个预期的修改，其他线程就不可以再提交一个预期的修改直到第一个预期的修改执行完毕。
	
	如果一个线程提交了一个预期的修改，然后做一些其它的工作时发生阻塞，这时候，这个共享数据结构实际上是被锁住的。其它线程可以检测到它们不能够提交一个预期的修改，然后回去做一些其它的事情。很明显，我们需要解决这个问题。


**可完成的预期修改**

	为了避免一个已经提交的预期修改可以锁住共享数据结构，一个已经提交的预期修改必须包含足够的信息让其他线程来完成这次修改。因此，如果一个提交了预期修改的线程从未完成这次修改，其他线程可以在它的支持下完成这次修改，保证这个共享数据结构对其他线程可用。

下图说明了上面描述的非阻塞算法的蓝图：

![enter image description here](https://camo.githubusercontent.com/c4d52cd702c9ad3124a62a76ee9493da43d07562/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d352e706e67)


>修改必须被当做一个或多个CAS操作来执行。因此，如果两个线程尝试去完成同一个预期修改，仅有一个线程可以所有的CAS操作。一旦一条CAS操作完成后，再次企图完成这个CAS操作都不会“得逞”。


**A-B-A问题**

	上面演示的算法可以称之为A-B-A问题。A-B-A问题指的是一个变量被从A修改到了B，然后又被修改回A的一种情景。其他线程对于这种情况却一无所知。
	
	如果线程A检查正在进行的数据更新，拷贝，被线程调度器挂起，一个线程B在此期可能可以访问这个共享数据结构。如果线程对这个数据结构执行了全部的更新，移除了它的预期修改，这样看起来，好像线程A自从拷贝了这个数据结构以来没有对它做任何的修改。然而，一个修改确实已经发生了。当线程A继续基于现在已经过期的数据拷贝执行它的更新时，这个数据修改已经被线程B的修改破坏。
	
下图说明了上面提到的A-B-A问题：

![enter image description here](https://camo.githubusercontent.com/e8ddaa9c30791bc7ed8f09c92585cb11b724558c/687474703a2f2f7475746f7269616c732e6a656e6b6f762e636f6d2f696d616765732f6a6176612d636f6e63757272656e63792f6e6f6e2d626c6f636b696e672d616c676f726974686d732d362e706e67)



**A-B-A问题的解决方案**

	A-B-A通常的解决方法就是不再仅仅替换指向一个预期修改对象的指针，而是指针结合一个计数器，然后使用一个单个的CAS操作来替换指针 + 计数器。这在支持指针的语言像C和C++中是可行的。因此，尽管当前修改指针被设置回指向 “不是正在进行的修改”（no ongoing modification），指针 + 计数器的计数器部分将会被自增，使修改对其它线程是可见的。
	
	在Java中，你不能将一个引用和一个计数器归并在一起形成一个单个的变量。不过Java提供了AtomicStampedReference类，利用这个类可以使用一个CAS操作自动的替换一个引用和一个标记（stamp）。

下面是原话：
注意：在非阻塞算法方面，我并不是一位专家，所以，下面的模板可能错误。不要基于我提供的模板实现自己的非阻塞算法。这个模板意在给你一个关于非阻塞算法大致是什么样子的一个idea。如果，你想实现自己的非阻塞算法，首先学习一些实际的工业水平的非阻塞算法的时间，在实践中学习更多关于非阻塞算法实现的知识。


```
mport java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicStampedReference;

public class NonblockingTemplate{
    public static class IntendedModification{
        public AtomicBoolean completed = new AtomicBoolean(false);
    }

    private AtomicStampedReference<IntendedModification> ongoinMod = new AtomicStampedReference<IntendedModification>(null, 0);
    //declare the state of the data structure here.

    public void modify(){
        while(!attemptModifyASR());
    }


    public boolean attemptModifyASR(){
        boolean modified = false;

        IntendedMOdification currentlyOngoingMod = ongoingMod.getReference();
        int stamp = ongoingMod.getStamp();

        if(currentlyOngoingMod == null){
            //copy data structure - for use
            //in intended modification

            //prepare intended modification
            IntendedModification newMod = new IntendModification();

            boolean modSubmitted = ongoingMod.compareAndSet(null, newMod, stamp, stamp + 1);

            if(modSubmitted){
                 //complete modification via a series of compare-and-swap operations.
                //note: other threads may assist in completing the compare-and-swap
                // operations, so some CAS may fail
                modified = true;
            }
        }else{
             //attempt to complete ongoing modification, so the data structure is freed up
            //to allow access from this thread.
            modified = false;
        }

        return modified;
    }
}
```


### 7.5 非堵塞算法的优点


**非阻塞算法是不容易实现的**
	正确的设计和实现非阻塞算法是不容易的。在尝试设计你的非阻塞算法之前，看一看是否已经有人设计了一种非阻塞算法正满足你的需求。
	
	Java已经提供了一些非阻塞实现（比如 ConcurrentLinkedQueue），相信在Java未来的版本中会带来更多的非阻塞算法的实现。
	
	除了Java内置非阻塞数据结构还有很多开源的非阻塞数据结构可以使用。例如，LAMX Disrupter和Cliff Click实现的非阻塞 HashMap。查看我的Java concurrency references page查看更多的资源。

**使用非阻塞算法的好处**
非阻塞算法和阻塞算法相比有几个好处。下面让我们分别看一下：

1. **选择**

	非阻塞算法的第一个好处是，给了线程一个选择当它们请求的动作不能够被执行时做些什么。不再是被阻塞在那，请求线程关于做什么有了一个选择。有时候，一个线程什么也不能做。在这种情况下，它可以选择阻塞或自我等待，像这样把CPU的使用权让给其它的任务。不过至少给了请求线程一个选择的机会。
	
	在一个单个的CPU系统可能会挂起一个不能执行请求动作的线程，这样可以让其它线程获得CPU的使用权。不过即使在一个单个的CPU系统阻塞可能导致死锁，线程饥饿等并发问题。

2. **没有死锁**

	非阻塞算法的第二个好处是，一个线程的挂起不能导致其它线程挂起。这也意味着不会发生死锁。两个线程不能互相彼此等待来获得被对方持有的锁。因为线程不会阻塞当它们不能执行它们的请求动作时，它们不能阻塞互相等待。非阻塞算法任然可能产生活锁（live lock），两个线程一直请求一些动作，但一直被告知不能够被执行（因为其他线程的动作）。

3. **没有线程挂起**

	挂起和恢复一个线程的代价是昂贵的。没错，随着时间的推移，操作系统和线程库已经越来越高效，线程挂起和恢复的成本也不断降低。不过，线程的挂起和户对任然需要付出很高的代价。
	
	无论什么时候，一个线程阻塞，就会被挂起。因此，引起了线程挂起和恢复过载。由于使用非阻塞算法线程不会被挂起，这种过载就不会发生。这就意味着CPU有可能花更多时间在执行实际的业务逻辑上而不是上下文切换。
	
	在一个多个CPU的系统上，阻塞算法会对阻塞算法产生重要的影响。运行在CPUA上的一个线程阻塞等待运行在CPU B上的一个线程。这就降低了程序天生就具备的并行水平。当然，CPU A可以调度其他线程去运行，但是挂起和激活线程（上下文切换）的代价是昂贵的。需要挂起的线程越少越好。

4. **降低线程延迟**

	在这里我们提到的延迟指的是一个请求产生到线程实际的执行它之间的时间。因为在非阻塞算法中线程不会被挂起，它们就不需要付昂贵的，缓慢的线程激活成本。这就意味着当一个请求执行时可以得到更快的响应，减少它们的响应延迟。
	
	非阻塞算法通常忙等待直到请求动作可以被执行来降低延迟。当然，在一个非阻塞数据数据结构有着很高的线程争用的系统中，CPU可能在它们忙等待期间停止消耗大量的CPU周期。这一点需要牢牢记住。非阻塞算法可能不是最好的选择如果你的数据结构哦有着很高的线程争用。不过，也常常存在通过重构你的程序来达到更低的线程争用。