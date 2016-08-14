### Quotes
> Bad programmers worry about the code. Good programmers worry about data structures and their relationships.          by Linus


### Avoid return null value
Try to use below instead:
- throw exception if this is not expected
- return empty list or object if this is possible

Ref:
[Using and Avoiding Null Explained](https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained)
[Null Object Pattern]

### Thread-safe and Effective Singleton
1. Eager mode
> private static Singleton INSTANCE = new Singleton();

2. Lazy mode
> if (instance == null) instance = new Singleton();

3. Thread-safe
- ‘synchronized’ method
> synchronized newInstance() { ... }

- AtomicReference fast-path before a ‘synchronized’ section

- AtomicReference with a spinlock

- double-checked locking  (not reliable in Java)
> Double Check Lock 双重检查锁

- double-checked locking using a ‘volatile’ field

> Spin lock - Wiki (Try to avoid)
In software engineering, a spinlock is a lock which causes a thread trying to acquire it to simply wait in a loop ("spin") while repeatedly checking if the lock is available. Since the thread remains active but is not performing a useful task, the use of such a lock is a kind of busy waiting.

- *Initialization On Demand Holder*
static inner class

*Reference*
[Thread-safe and performance](http://literatejava.com/jvm/fastest-threadsafe-singleton-jvm/)
[initialization on demand holder](http://ifeve.com/initialization-on-demand-holder-idiom/)

### Ref Doc Url
> [图说设计模式](http://design-patterns.readthedocs.io/zh_CN/latest/index.html)
