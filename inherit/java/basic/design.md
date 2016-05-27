# 设计模式

[TOC]
## 帮助文档

## 一、设计模式有什么用？
## 二、单例模式

[来源](http://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-with-examples)

### 2.1 Eager initialization

	In eager initialization, the instance of Singleton Class is created at the time of class loading, this is the easiest method to create a singleton class but it has a drawback that instance is created even though client application might not be using it.



```java
package com.journaldev.singleton;
 
public class EagerInitializedSingleton {
     
    private static final EagerInitializedSingleton instance = new EagerInitializedSingleton();
     
    //private constructor to avoid client applications to use constructor
    private EagerInitializedSingleton(){}
 
    public static EagerInitializedSingleton getInstance(){
        return instance;
    }
}
```

### 2.2 Static block initialization

	Static block initialization implementation is similar to eager initialization, except that instance of class is created in the static block that provides option for exception handling.

```java
package com.journaldev.singleton;
 
public class StaticBlockSingleton {
 
    private static StaticBlockSingleton instance;
     
    private StaticBlockSingleton(){}
     
    //static block initialization for exception handling
    static{
        try{
            instance = new StaticBlockSingleton();
        }catch(Exception e){
            throw new RuntimeException("Exception occured in creating singleton instance");
        }
    }
     
    public static StaticBlockSingleton getInstance(){
        return instance;
    }
}
```


### 2.3 Lazy Initialization

	Lazy initialization method to implement Singleton pattern creates the instance in the global access method. Here is the sample code for creating Singleton class with this approach.

```java
package com.journaldev.singleton;
 
public class ThreadSafeSingleton {
 
    private static ThreadSafeSingleton instance;
     
    private ThreadSafeSingleton(){}
     
    public static synchronized ThreadSafeSingleton getInstance(){
        if(instance == null){
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
     
}
```


```java
public static ThreadSafeSingleton getInstanceUsingDoubleLocking(){
    if(instance == null){
        synchronized (ThreadSafeSingleton.class) {
            if(instance == null){
                instance = new ThreadSafeSingleton();
            }
        }
    }
    return instance;
}

```



### 2.4 Bill Pugh Singleton Implementation

	Prior to Java 5, java memory model had a lot of issues and above approaches used to fail in certain scenarios where too many threads try to get the instance of the Singleton class simultaneously. So Bill Pugh came up with a different approach to create the Singleton class using a inner static helper class. The Bill Pugh Singleton implementation goes like this;

```java

package com.journaldev.singleton;
 
public class BillPughSingleton {
 
    private BillPughSingleton(){}
     
    private static class SingletonHelper{
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }
     
    public static BillPughSingleton getInstance(){
        return SingletonHelper.INSTANCE;
    
```


>Notice the private inner static class that contains the instance of the singleton class. When the singleton class is loaded, SingletonHelper class is not loaded into memory and only when someone calls the getInstance method, this class gets loaded and creates the Singleton class instance.
This is the most widely used approach for Singleton class as it doesn’t require synchronization. I am using this approach in many of my projects and it’s easy to understand and implement also.

### 2.5 Enum Singleton

	To overcome this situation with Reflection, Joshua Bloch suggests the use of Enum to implement Singleton design pattern as Java ensures that any enum value is instantiated only once in a Java program. Since Java Enum values are globally accessible, so is the singleton. The drawback is that the enum type is somewhat inflexible; for example, it does not allow lazy initialization.

```java
package com.journaldev.singleton;
 
public enum EnumSingleton {
 
    INSTANCE;
     
    public static void doSomething(){
        //do something
    }
}
```



### 2.6 单例中的序列化问题(Serialization and Singleton)


用`readResolve()`方法:

	序列化操作提供了一个很特别的钩子（hook）-类中具有一个私有的被实例化的方法readresolve(),这个方法可以确保类的开发人员在序列化将会返回怎样的object上具有发言权。足够奇怪的，readresolve()并不是静态的，但是在序列化创建实例的时候被引用。我们在一分钟内就开始体验这个。下面的例子将说明readresolve（）怎样在我们的单例中起作用

```java
public final class MySingleton {  
 private MySingleton() { }  
 private static final MySingleton INSTANCE = new MySingleton();  
 public static MySingleton getInstance() { return INSTANCE; }  
 private Object readResolve() throws ObjectStreamException {  
  // instead of the object we're on,   
  // return the class variable INSTANCE  
  return INSTANCE;   
 }  
}  
```




