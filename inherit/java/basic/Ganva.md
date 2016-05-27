

[TOC]
# Ganva
ganva是由google推出的工具包，值得学习，包罗万象
maven依赖
```html
  <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>19.0</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.findbugs</groupId>
            <artifactId>jsr305</artifactId>
            <version>3.0.0</version>
  </dependency>
```

## 官方文档
[教程](http://ifeve.com/google-guava/)
[消息总线](http://www.cnblogs.com/peida/p/EventBus.html)

## 关于Java注解
**一、元注解：**

　　元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：
　　　　1. `@Target`
　　　　2. `@Retention`
　　　　3. `@Documented`
　　　　4. `@Inherited`
　　这些类型和它们所支持的类在java.lang.annotation包中可以找到。下面我们看一下每个元注解的作用和相应分参数的使用说明。

- **@Target：**

　　　@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

　　作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）

　　取值(ElementType)有：

　　　　1.CONSTRUCTOR:用于描述构造器
　　　　2.FIELD:用于描述域
　　　　3.LOCAL_VARIABLE:用于描述局部变量
　　　　4.METHOD:用于描述方法
　　　　5.PACKAGE:用于描述包
　　　　6.PARAMETER:用于描述参数
　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

- **@Retention：**

　　@Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

　　作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）

　　取值（RetentionPoicy）有：

　　　　1. `SOURCE`:在源文件中有效（即源文件保留）
　　　　2. `CLASS`:在class文件中有效（即class保留）
　　　　3.`RUNTIME`:在运行时有效（即运行时保留）

　　Retention meta-annotation类型有唯一的value作为成员，它的取值来自java.lang.annotation.RetentionPolicy的枚举类型值。具体实例如下：

- **@Documented:**

　　@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

- **@Inherited：**

　　@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

　　注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

　　当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

**二、 自定义注解**

使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。

　　定义注解格式：
　　public @interface 注解名 {定义体}

　　注解参数的可支持数据类型：

　　　　1.所有基本数据类型（int,float,boolean,byte,double,char,long,short)
　　　　2.String类型
　　　　3.Class类型
　　　　4.enum类型
　　　　5.Annotation类型
　　　　6.以上所有类型的数组

　　Annotation类型里面的参数该怎么设定: 
　　**第一:**只能用public或默认(default)这两个访问权修饰.例如,String value();这里把方法设为defaul默认类型；　 　
　　**第二:**参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和 String,Enum,Class,annotations等数据类型,以及这一些类型的数组.例如,String value();这里的参数成员就为String;　　
　　**第三:**如果只有一个参数成员,最好把参数名称设为"value",后加小括号.例:下面的例子FruitName注解就只有一个参数成员。

　　简单的自定义注解和使用注解实例：
```java

```

## 实战
### 使用md5实现一致性hash算法
```java
package com.carl.demo.guava;

import com.google.common.hash.HashFunction;
import com.google.common.hash.Hashing;
import org.apache.commons.lang3.tuple.Pair;

import java.nio.charset.Charset;
import java.util.*;

/**
 * Created by carl on 16-3-4.
 */
public class ConsistentHash<T> {
    private final HashFunction hashFunction;
    private final int numberOfReplicas;
    private final SortedMap<Long, T> circle = new TreeMap<Long, T>();

    public ConsistentHash(HashFunction hashFunction,
                          int numberOfReplicas,
                          Collection<T> nodes) {
        this.hashFunction = hashFunction;
        this.numberOfReplicas = numberOfReplicas;

        for (T node : nodes) {
            add(node);
        }
    }

    public void add(T node) {
        for (int i = 0; i < numberOfReplicas; i++) {
            circle.put(hashFunction.hashString(node.toString() + i, Charset.defaultCharset()).asLong(),
                    node);
        }
    }

    public void remove(T node) {
        for (int i = 0; i < numberOfReplicas; i++) {
            circle.remove(hashFunction.hashString(node.toString() + i, Charset.defaultCharset()).asLong());
        }
    }

    public T get(Object key) {
        if (circle.isEmpty()) {
            return null;
        }
        //先对key进行hash运算
        long hash = hashFunction.hashString(key.toString(), Charset.defaultCharset()).asLong();
        if (!circle.containsKey(hash)) {
            SortedMap<Long, T> tailMap = circle.tailMap(hash);
            hash = tailMap.isEmpty() ? circle.firstKey() : tailMap.firstKey();
        }
        return circle.get(hash);
    }


    public static void main(String[] args) {
        ArrayList<String> al = new ArrayList<String>();
        al.add("redis1");
        al.add("redis2");
        al.add("redis3");
        al.add("redis4");
        HashFunction md5 = Hashing.md5();
        ConsistentHash<String> consistentHash = new ConsistentHash<>(md5, 100, al);
        SortedMap<Long, String> tree = consistentHash.circle;
        Pair[] pairs = new Pair[tree.size()];
        int i = 0;
        for (Map.Entry<Long, String> entry : tree.entrySet()) {
            pairs[i++] = Pair.of(entry.getKey(), entry.getValue());
        }
        System.out.println(tree);
        int totalSize = 200000;
        String[] userIds = new String[totalSize];
        for (int j = 0; j < totalSize; j++) {
            userIds[j] = UUID.randomUUID().toString();
        }
        Map<String, List<String>> m = new HashMap<>();
        for (int j = 0; j < totalSize; j++) {
            String uuid = userIds[j];
            String key = consistentHash.get(uuid);
            if (!m.containsKey(key)) {
                List<String> list = new LinkedList<>();
                m.put(key, list);
                list.add(uuid);
            } else {
                m.get(key).add(uuid);
            }
        }
        for (Map.Entry<String, List<String>> entry : m.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue().size());
        }
    }
}

```




## 一、 基本工具

### 1.1 如何避免使用null

**为何拒绝使用null**

轻率地使用null可能会导致很多令人惊愕的问题。通过学习Google底层代码库，我们发现**95%**的集合类不接受null值作为元素。我们认为， 相比默默地接受null，使用快速失败操作拒绝null值对开发者更有帮助。

此外，**Null的含糊语义让人很不舒服**。Null很少可以明确地表示某种语义，例如，Map.get(key)返回Null时，**可能表示map中的值是null**，**亦或map中没有key对应的值**。Null可以表示失败、成功或几乎任何情况。使用Null以外的特定值，会让你的逻辑描述变得更清晰。

**Null确实也有合适和正确的使用场景，如在性能和速度方面Null是廉价的**，而且在对象数组中，出现Null也是无法避免的。但相对于底层库来说，在应用级别的代码中，Null往往是导致混乱，疑难问题和模糊语义的元凶，就如同我们举过的Map.get(key)的例子。最关键的是，Null本身没有定义它表达的意思。

鉴于这些原因，很多Guava工具类对Null值都采用快速失败操作，除非工具类本身提供了针对Null值的因变措施。此外，Guava还提供了很多工具类，让你更方便地用特定值替换Null值。


```
	/**
     * 主要测试方法
     *
     * @param args
     */
    public static void main(String[] args) {
        //1.创建一个新引用
        Optional<String> username = Optional.absent();
        System.out.println(username);
        //2.查看是否存在
        System.out.println(username.isPresent());
        //3.如果不存在提供默认值
        System.out.println(username.or("bb"));
        //4.存在的话，如何获取值
        username = Optional.of("aa");
        System.out.println(username.get());
    }
```



**实战案例：**

1. 不要在Set中使用null，或者把null作为map的键值。使用特殊值代表null会让查找操作的语义更清晰。

2. 如果你想把null作为map中某条目的值，更好的办法是 不把这一条目放到map中，而是单独维护一个”值为null的键集合” (null keys)。Map 中对应某个键的值是null，和map中没有对应某个键的值，是非常容易混淆的两种情况。因此，最好把值为null的键分离开，并且仔细想想，null值的键在你的项目中到底表达了什么语义。
3. 如果你需要在列表中使用null——并且这个列表的数据是稀疏的，使用`Map<Integer, E>`可能会更高效，并且更准确地符合你的潜在需求。
4. 此外，考虑一下使用自然的null对象——特殊值。举例来说，为某个enum类型增加特殊的枚举值表示null，比如java.math.RoundingMode就定义了一个枚举值UNNECESSARY，它表示一种不做任何舍入操作的模式，用这种模式做舍入操作会直接抛出异常。

>如果你真的需要使用null值，但是null值不能和Guava中的集合实现一起工作，你只能选择其他实现。比如，用JDK中的Collections.unmodifiableList替代Guava的ImmutableList




### 1.2 前置条件



Guava在Preconditions类中提供了若干前置条件判断的实用方法，我们强烈建议在Eclipse中静态导入这些方法。每个方法都有三个变种：
1. 没有额外参数：抛出的异常中没有错误消息；
2. 有一个Object对象作为额外参数：抛出的异常使用Object.toString() 作为错误消息；
3. 有一个**String**对象作为额外参数，并且有**一组任意数量的附加Object对象**：这个变种处理异常消息的方式有点类似printf，但考虑GWT的兼容性和效率，只支持%s指示符。例如：


**实战**

相比Apache Commons提供的类似方法，我们把Guava中的Preconditions作为首选。Piotr Jagielski在他的博客中简要地列举了一些理由：

- 在静态导入后，Guava方法非常清楚明晰。checkNotNull清楚地描述做了什么，会抛出什么异常；
checkNotNull直接返回检查的参数，让你可以在构造函数中保持字段的单行赋值风格：this.field = checkNotNull(field)
- 简单的、参数可变的**printf风格异常**信息。鉴于这个优点，在JDK7已经引入Objects.requireNonNull的情况下，我们仍然建议你使用checkNotNull。
- 在编码时，如果某个值有多重的前置条件，我们建议你把它们放到**不同的行**，这样有助于在调试时定位。此外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息。
 
```
public class Main {
    public static void main(String[] args) {
        String t = null;
        checkNotNull(t,"%s元素不能为%s","t","空值");
        checkNotNull(t);
        checkNotNull(t,"t元素不能为空");

    }
}
```


### 1.3 简单排序

先上hello_world:

```java
public class Main {
    public static void main(String[] args) {
        Optional<Integer> possible = Optional.of(1);
        //1. 创建按照长度的排序对象
        Ordering<String> byLengthOrdering = new Ordering<String>() {
            public int compare(String left, String right) {
                return Ints.compare(left.length(), right.length());
            }
        };
        //2. 初始化数据
        List<String> list = Arrays.asList(new String[]{"5", "3", "1", "2", "0",null});
        System.out.println(list);
        //3. 使用排序对象进行排序
        Collections.sort(list, Ordering.natural().nullsFirst());
        System.out.println("排序后:");
        System.out.println(list);
    }
}

```

>观察：本质上是帮助生了一个comparator对象

**实战：**

- **可以创建多种排序器**，例如自然排序，按照对象的toString()进行字符串的字典排序，或者从Comparator对象转化为排序器，或者直接继承Ordering

- **链式调用** `reverse()` `nullFirst()` `nullLast()` 等等你懂的，甚至可以进行合成

- **支持操纵集合**



### 1.4 异常

guava 的异常关注点：

**如何包装成一个运行期异常**
异常的**传递和传播**


## 二、集合

### 2.1 不可变集合

**为什么要使用不可变集合**

**优点:**
1. 当对象被不可信的库调用时，不可变形式是安全的；
2. 不可变对象被多个线程调用时，不存在竞态条件问题
3. 不可变集合不需要考虑变化，因此可以节省时间和空间。所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）；
4. 不可变对象因为有固定不变，可以作为常量来安全使用。

创建对象的不可变拷贝是一项很好的防御性编程技巧。Guava为所有JDK标准集合类型和Guava新集合类型都提供了简单易用的不可变版本。

 JDK也提供了Collections.unmodifiableXXX方法把集合包装为不可变形式，但我们认为不够好：
- **笨重而且累赘**：不能舒适地用在所有想做防御性拷贝的场景；
- 不安全：**要保证没人通过原集合的引用进行修改**，返回的集合才是事实上不可变的；
- 低效：**包装过的集合仍然保有可变集合的开销**，比如并发修改的检查、散列表的额外空间，等等。
如果你没有修改某个集合的需求，或者希望某个集合保持不变时，把它防御性地拷贝到不可变集合是个很好的实践。

>重要提示：所有Guava**不可变集合的实现都不接受null值**。我们对Google内部的代码库做过详细研究，发现只有5%的情况需要在集合中允许null元素，剩下的95%场景都是遇到null值就快速失败。如果你需要在不可变集合中使用null，请使用JDK中的Collections.unmodifiableXXX方法。更多细节建议请参考“使用和避免null”。

**<1> 如何使用不可变集合**

创建方式:
- `copyOf`方法，如 `ImmutableSet.copyOf(set);`
- `of`方法，如`ImmutableSet.of(“a”, “b”, “c”)或 ImmutableMap.of(“a”, 1, “b”, 2);`
- Builder工具，如：
```java
GOOGLE_COLORS =ImmutableSet.<Color>builder().addAll(WEBSAFE_COLORS).add(new Color(0, 191, 255)).build();
```

**<2>比想象中更智能的copyOf**
请注意，`ImmutableXXX.copyOf`方法会尝试在安全的时候避免做拷贝——实际的实现细节不详，但通常来说是很智能的，比如：
```java
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);
void thingamajig(Collection<String> collection) {
ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
}
```

>在这段代码中，
>a. ImmutableList.copyOf(foobar)会智能地直接返回foobar.asList(),它是一个ImmutableSet的常量时间复杂度的List视图。
b.作为一种探索，ImmutableXXX.copyOf(ImmutableCollection)会试图对如下情况避免线性时间拷贝：
在常量时间内使用底层数据结构是可能的——例如，ImmutableSet.copyOf(ImmutableList)就不能在常量时间内完成。
c.不会造成内存泄露——例如，你有个很大的不可变集合ImmutableList<String>
hugeList， ImmutableList.copyOf(hugeList.subList(0, 10))就会显式地拷贝，以免不必要地持有hugeList的引用。
d.不改变语义——所以ImmutableSet.copyOf(myImmutableSortedSet)会显式地拷贝，因为和基于比较器的ImmutableSortedSet相比，ImmutableSet对hashCode()和equals有不同语义。
e.在可能的情况下避免线性拷贝，可以最大限度地减少防御性编程风格所带来的性能开销。

**<3>asList视图**

所有不可变集合都有一个asList()方法提供ImmutableList视图，来帮助你用列表形式方便地读取集合元素。例如，你可以使用sortedSet.asList().get(k)从ImmutableSortedSet中读取第k个最小元素。

asList()返回的ImmutableList通常是——并不总是——开销稳定的视图实现，而不是简单地把元素拷贝进List。也就是说，asList返回的列表视图通常比一般的列表平均性能更好，比如，在底层集合支持的情况下，它总是使用高效的contains方法。

```java

    private static void _testCreate() {
        // 1. 使用 of 直接创建
        ImmutableSet<User> immutableSet = ImmutableSet.of(new User("a", 1)).of(new User("b", 2));
        System.out.println(immutableSet.asList().get(0).getUsername());

        // 2. 使用builder 创建
        
    }

    /**
     * 测试copyOf，依然如此
     */
    private static void _testCopyOf() {
        List<User> users = Lists.newArrayList(
                new User("a", 1),
                new User("b", 1),
                new User("c", 3)
        );

        ImmutableList<User> immutableList = ImmutableList.copyOf(users);
        User u1 = users.get(0);
        User u2 = immutableList.asList().get(0);
        immutableList.get(0).setUsername("special username");
        System.out.println(u1 == u2);
        System.out.println(u2.getUsername());
    }
```


### 2.2 新集合类型

Guava引入了很多JDK没有的、但我们发现明显有用的新集合类型。这些新类型是为了和JDK集合框架共存，而没有往JDK集合抽象中硬塞其他概念。作为一般规则，Guava集合非常精准地遵循了JDK接口契约。


----------


#### Multiset

可以用两种方式看待Multiset：
1. 没有元素顺序限制的`ArrayList<E>`
2.` Map<E, Integer>`，键为元素，值为计数

**注意事项**
请注意，`Multiset<E>`不是`Map<E, Integer>`，虽然Map可能是某些Multiset实现的一部分。准确来说Multiset是一种Collection类型，并履行了Collection接口相关的契约。关于Multiset和Map的显著区别还包括：

1. Multiset中的**元素计数只能是正数**。任何元素的计数都不能为负，也不能是0。elementSet()和entrySet()视图中也不会有这样的元素。
2. multiset.size()返回集合的大小，等同于所有元素计数的总和。对于**不重复元素的个数**，应使用elementSet().size()方法。（因此，add(E)把multiset.size()增加1）
3. multiset.iterator()会迭代重复元素，因此迭代长度等于multiset.size()。
4. Multiset**支持直接增加、减少或设置元素的计数**。setCount(elem, 0)等同于移除所有elem。
对multiset 中没有的元素，multiset.count(elem)始终返回0。


----------

**Multiset的各种实现**



| Map        | 对应的Multiset           | 是否支持null元素  |
| ------------- |:-------------:| -----:|
| HashMap      | TreeMultiset | 是 |
| TreeMap     | centered      |   是（如果comparator支持的话） |
| LinkedHashMap	| LinkedHashMultiset	| 是 |
| ConcurrentHashMap |	ConcurrentHashMultiset	| 否
| ImmutableMap	| ImmutableMultiset	| 否


----------





**SortedMultiset**

SortedMultiset是Multiset 接口的变种，它支持高效地获取指定范围的子集。比方说，你可以用 `latencies.subMultiset(0,BoundType.CLOSED, 100, BoundType.OPEN).size()`来统计你的站点中延迟在100毫秒以内的访问，然后把这个值和`latencies.size()`相比，以获取这个延迟水平在总体访问中的比例。

TreeMultiset实现SortedMultiset接口。在撰写本文档时，ImmutableSortedMultiset还在测试和GWT的兼容性。


----------


#### Multimap

**为何需要Multimap**
每个有经验的Java程序员都在某处实现过`Map<K, List<V>`>`或Map<K, Set<V>>`，并且要忍受这个结构的笨拙。例如，Map<K, Set<V>>通常用来表示非标定有向图。Guava的 Multimap可以很容易地把一个键映射到多个值。换句话说，Multimap是把键映射到任意多个值的一般方式。

很少会直接使用Multimap接口，更多时候你会用`ListMultimap`或`SetMultimap`接口，它们分别把键映射到List或Set。
```java
//使用list作为返回
 public static void main(String[] args) {
        ListMultimap<String, String> multimap = HashMultimap.create();
        multimap.put("1", "a");
        multimap.put("1", "b");
        System.out.println(multimap.get("2"));
        System.out.println(multimap.containsKey("2"));
    }
```


**Multimap还支持若干强大的视图：**

1. **asMap**为`Multimap<K, V>`提供`Map<K,Collection<V>>`形式的视图。返回的Map支持remove操作，并且会反映到底层的Multimap，但**它不支持put或putAll操作**。更重要的是，如果你想为Multimap中没有的键返回**null**，而不是一个新的、可写的空集合，你就可以使用asMap().get(key)。（你可以并且应当把asMap.get(key)返回的结果转化为适当的集合类型——如SetMultimap.asMap.get(key)的结果转为Set，**ListMultimap.asMap.get(key)的结果转为List**——Java类型系统不允许ListMultimap直接为asMap.get(key)返回List——译者注：也可以用Multimaps中的asMap静态方法帮你完成类型转换）
2. **entries**用`Collection<Map.Entry<K, V>>`返回Multimap中所有”键-单个值映射”——包括重复键。（对SetMultimap，返回的是Set）
3. **keySet**用Set表示Multimap中所有不同的键。
keys用Multiset表示Multimap中的所有键，每个键重复出现的次数等于它映射的值的个数。可以从这个Multiset中移除元素，但不能做添加操作；移除操作会反映到底层的Multimap。
4. **values()**用一个”扁平”的`Collection<V>`包含Multimap中的所有值。这有一点类似于Iterables.concat(multimap.asMap().values())，但它直接返回了单个Collection，而不像multimap.asMap().values()那样是按键区分开的Collection。



**本质上：**
	
	HashMultimap<Integer,User> 就是一个Map<Integer,List<User>>的HashMap 实现


	

```java
   private static void _testShow() {
        Multimap<Integer, User> multimap = _testCreate();
        for (User user : multimap.get(1)) {
            System.out.println(ToStringBuilder.reflectionToString(user));
        }
    }

    private static Multimap<Integer, User> _testCreate() {
        Multimap<Integer, User> multimap = HashMultimap.create(3, 3);
        multimap.put(1, new User("a0", 1));
        multimap.put(1, new User("a1", 1));
        multimap.put(1, new User("a2", 1));

        multimap.put(2, new User("b0", 1));
        multimap.put(2, new User("b1", 1));
        multimap.put(2, new User("b2", 1));

        multimap.put(3, new User("c0", 1));
        multimap.put(3, new User("c1", 1));
        multimap.put(3, new User("c2", 1));

        return multimap;
    }
```


----------

#### BiMap



一一映射

----------


### 2.3  强大的集合工具类

<1>**提供了各种静态工厂方法** 
```java
List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
```


<2>**Iterables**

在可能的情况下，Guava提供的工具方法更偏向于接受Iterable而不是Collection类型。在Google，对于不存放在主存的集合——比如从数据库或其他数据中心收集的结果集，因为实际上还没有攫取全部数据，这类结果集都不能支持类似size()的操作 ——通常都不会用Collection类型来表示。

..... 基本上你能用到的方法都有


### 2.4-集合扩展工具类

**<1> Forwarding装饰器**

```java
package com.carl.demo.guava;

import com.google.common.base.Function;
import com.google.common.base.Optional;
import com.google.common.collect.*;
import com.google.common.primitives.Ints;
import edu.emory.mathcs.backport.java.util.Arrays;
import edu.emory.mathcs.backport.java.util.Collections;

import static com.google.common.base.Preconditions.*;

import javax.annotation.Nullable;
import java.util.Collection;
import java.util.Comparator;
import java.util.Date;
import java.util.List;

/**
 * Created by carl on 16-3-16.
 */
public class Main {
    public static void main(String[] args) {

        MyForwardingList<String> myForwardingList =
                new MyForwardingList<>();
        myForwardingList.setSource(Lists.newArrayList("1", "3"));
        myForwardingList.add(0,"tttt");
    }

    static class MyForwardingList<String> extends ForwardingList<String> {

        private List<String> source;

        public List<String> getSource() {
            return source;
        }

        public void setSource(List<String> source) {
            this.source = source;
        }

        @Override
        protected List<String> delegate() {
            return source;
        }

        @Override
        public void add(int index, String element) {
            System.out.println("what happening");
            super.add(index, element);
        }
    }

}

```



**<2> PeekingIterator**
支持peek的迭代器



**<3> AbstractIterator**
```java

public static Iterator<String> skipNulls(final Iterator<String> in) {

    return new AbstractIterator<String>() {
        protected String computeNext() {
            while (in.hasNext()) {
                String s = in.next();
                if (s != null) {
                    return s;
                }
            }
            return endOfData();
        }
    };
}
```

**<4> AbstractSequentialIterator**


## 三、 字符串

Guava 提供了很多string的内置工具，包括Joiner Spliter
[guava-strings](http://ifeve.com/google-guava-strings/)



## 四、concurrent

### 4.1 striped 细粒度的锁

[深入Guava源码之Stripe](http://www.blogjava.net/DLevin/archive/2013/12/25/407990.html)

[Fine-Grained](http://codingjunkie.net/striped-concurrency/)

**功能:**

1. 支持类似与 concurrentHashMap的细粒度锁
2. 支持强引用，若引用的锁
3. 支持`lock`,`semaphore`等常用的线程工具



