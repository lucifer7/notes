# Algorithm-入门篇

[TOC]

## 帮助文档

## 一、字符串

### 1.1 Manacher算法：求解最长回文字符串
[demo](http://blog.csdn.net/chenxuegui1234/article/details/19418907)

```java
package com.carl.demo.algorithm;

/**
 * Created by carl on 16-4-5.
 */
public class ManacherDemo {

    private static String _convert(String src, char c) {
        if (src == null)
            throw new NullPointerException("空不能进行转换");
        StringBuilder ret = new StringBuilder(src.length() * 2 + 1);
        for (int i = 0; i < src.length(); i++) {
            ret.append(c).append(src.charAt(i));
        }
        ret.append(c);
        return ret.toString();
    }


    private static String _convert(String src) {
        return _convert(src, '#');
    }


    public static String manacher(String src) {
        String x = _convert(src);
        int[] rad = new int[x.length()];
        int maxCenter = -1, maxRight = -1, r;
        for (int i = 0; i < x.length(); i++) {
            //1. 重置r=1
            r = 1;
            //2. 判断r的大小
            if (i < maxRight) {
                int v1 = rad[2 * maxCenter - i] - 1;
                int v2 = maxRight - i + 1;
                r = Math.min(v1, v2);
            }
            while (true) {
                if (i < r || (i + r >= x.length())) {
                    break;
                }
                if (x.charAt(i - r) != x.charAt(i + r)) {
                    break;
                }
                r++;
            }
            rad[i] = r;
            //3.
            if (i + r - 1 > maxRight) {
                maxRight = i + r - 1;
                maxCenter = i;
            }
        }
        //4.找出rad[最大值]
        int max = -1;
        int i = -1;
        for (int j = 0; j < rad.length; j++) {
            if (rad[j] > max) {
                max = rad[j];
                i = j;
            }
        }
        StringBuilder sb = new StringBuilder();
        for (int j = i - (max - 1); j <= (i + (max - 1)); j++) {
            char c = x.charAt(j);
            if (c == '#')
                continue;
            sb.append(c);
        }

        return sb.toString();
    }

    public static void main(String[] args) {
        System.out.println(manacher("fffffababbbbbbbba"));
    }
}

```


## 二、 线性结构

### 2.1 栈

**（1）特点：**

	先进后出，应用如递归中方法栈，后缀前缀中缀表达式。。。

**（2）使用数组实现栈**

```java
package com.carl.demo.algorithm.linar;

import java.util.Arrays;

/**
 * Created by carl on 2016/4/3.
 */
public class MyStack {
    private int[] table;
    private int topIndex = -1;

    public MyStack(int capacity) {
        this.table = new int[capacity];
    }

    //基本操作，压栈
    public void push(int val) {
        if (isFull())
            throw new IndexOutOfBoundsException("stack已满");
        topIndex++;
        table[topIndex] = val;
    }

    //基本操作，出栈
    public int pop() {
        if (isEmpty()) {
            throw new IndexOutOfBoundsException("stack为空");
        }
        //1.取出topIndex
        //2.topIndex -1
        return table[topIndex--];

    }

    public int peek() {
        if (isEmpty()) {
            throw new IndexOutOfBoundsException("stack为空");
        }
        return table[topIndex];
    }

    public boolean isEmpty() {
        return topIndex < 0;
    }


    public boolean isFull() {
        return topIndex >= table.length - 1;
    }

    public void display() {
        System.out.println("table:" + Arrays.toString(table));
    }

}


```


### 2.2 队列

#### 循环队列
**（1） 特点：**

		先进先出

**（2）使用数组实现循环队列**

>1.使用一个额外变量表示数目
>2.不使用一个额外变量表示数目，但需要多出一个位置

前者太简单，不考虑了，考虑后者，也简单。。
**思路：**
	额外拿出一个位置来，只有`start = end`时，队列为空

```java
package com.carl.demo.algorithm.linar;

import java.util.Arrays;

/**
 * Created by carl on 2016/4/3.
 */
public class MyQueue {

    private int[] table;
    private int front;
    private int end;
    private int size;

    public MyQueue(int capacity) {
        this.table = new int[capacity+1];
        front = 0;
        end = 0;
        size = 0;
    }


    public void insert(int val) {
        if (isFull())
            throw new IndexOutOfBoundsException("队列已满");
        table[end] = val;
        end = (end + 1) % this.table.length;
    }

    public int remove() {
        if (isEmpty())
            throw new IndexOutOfBoundsException("队列已空");
        int ret = table[front];
        front = (front + 1) % table.length;
        return ret;
    }

    public int peekFront() {
        return table[front];
    }

    public boolean isEmpty() {
        return front == end;
    }

    public boolean isFull() {
        return (this.end + 1) % (table.length) == this.front;
    }

    public void display() {
        System.out.println("table:" + Arrays.toString(table));
    }

}

```

#### 双端队列

**（1） 概念：**

		双端都可以进行数据插入和移除操作的队列。它综合提供了栈和队列的功能，并不是很常用。



#### 优先级队列

暂不考虑，貌似jdk使用的是堆排序



### 2.3 链表

**（1） 概念：**

		链表是一种特殊的线性表，由一系列的节点组成，节点的顺序是通过元素的指针链接次序来确定的。

**（2）双端链表**

		不仅仅记录起始节点，也记录结束节点

**（3）有序链表**
**（4）双向链表：**

			每一个节点不仅仅保存next的节点，还保存prev节点，请自己实现。




## 三、递归

**分治法：** 伟大的思想。。。将不能解决的问题分为一个个能解决的小问题。

### 3.1 简单实例：

#### 二分搜索：太过简单，没有实现。。

#### 汉诺塔：简单实现。。。

```java
package com.carl.demo.algorithm.recursive;

/**
 * Created by carl on 2016/4/4.
 */
public class HanoTa {
    private static final String[] TOWERS = new String[]{"A", "B", "C"};

    /**
     * <pre>
     *     从from移动到to,移动n个圆盘
     *     从上往下对圆盘进行编号 1，2，3.....n
     * </pre>
     */
    private static void _move(int from, int to, int n) {
        if (n == 1) {
            //1.如果只要移动一个
            System.out.println("从" + TOWERS[from] + "移动到" + TOWERS[to] + "，移动第" + n + "块圆盘");

        } else {
            //2.否则
            int other = _findTheOther(from, to);
            _move(from, other, n - 1);
            System.out.println("从" + TOWERS[from] + "移动到" + TOWERS[to] + "，移动第" + n + "块圆盘");
            _move(other, to, n - 1);
        }
    }


    /**
     * <pre>
     *     从from->to，剩下那个用来过渡的tower
     * </pre>
     *
     * @return
     */
    private static int _findTheOther(int from, int to) {
        for (int i = 0; i < TOWERS.length; i++) {
            if (i != from && i != to) {
                return i;
            }
        }
        return -1;
    }


    public static void main(String[] args) {
      _move(0,2,3);
    }
}


```

#### 其他章节有应用，例如排序章节


## 四、排序

### 4.1 希尔，快速，归并，堆，桶排序

## 五、树

### 3.1 把二元查找树转变成排序的双向链表-太简单，没有实现

中序遍历 改变指向


### 3.2 红黑树 - 参考TreeMap源码实现

[理解旋转](http://blog.csdn.net/gabriel1026/article/details/6311339)
[维基百科-红黑树](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)

**以下是一些简单的切入点（insert 操作）：**
	
	
	1.假设父亲节点p，叔叔节点是u，祖父节点是pp，插入
	2.如果p黑，不用处理
	3.插入的首先默认是红色，root的话是黑色
	4.如果p红，u红，p，u变黑，pp变红
	5.如果p红，u黑，看顺序
	6.如果p红，u黑，p是左子
		如果x也是左子，p变黑，pp便红，pp右旋
		如果x是右子，p左旋，考虑p变成上种情况
	7.如果p红，u黑，p是右子
		如果x也是右子，p变黑，pp便红，pp左旋
		如果x是左子，p右旋，考虑p变成上种情况
		




```java
package com.carl.demo.algorithm;


import com.carl.demo.util.StringUtils;

import java.util.LinkedList;
import java.util.Queue;

/**
 * Created by carl on 16-4-5.
 */
public class RedBlackTree {

//    红黑树规则：
//    节点是红色或黑色。
//    根是黑色。
//    所有叶子都是黑色（叶子是NIL节点）。
//    每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
//    从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。


    private Node root;
    private int size;

    public void put(int toBeAdd) {
        //1. 如果是根节点，设置根，然后返回
        Node t = root;
        if (t == null) {
            root = new Node(toBeAdd);
            size = 1;
            return;
        }

        Node parent; // 该变量保存父节点

        //2.找到要插入的节点
        int cmp; //该变量保存的是当前节点的值
        while (true) {
            cmp = toBeAdd - t.val;
            parent = t;
            if (cmp == 0) {
                //2.1 如果该值已经存在
                System.out.println(StringUtils.format("值%s已经存在", toBeAdd));
                return;
            } else if (cmp > 0) {
                t = t.right;
            } else {
                t = t.left;
            }
            if (t == null) {
                break;
            }
        }
        //3. 创建要增加的新节点
        Node e = new Node(parent, toBeAdd);
        //4. 增加节点
        if (cmp > 0) {
            parent.right = e;
        } else {
            parent.left = e;
        }
        //5. 增加后修复红黑树平衡性问题
        _fixAfterInsertion(e);
        size++;
    }

    private void _fixAfterInsertion(Node x) {
        //新增的都默认是红色
        x.color = RED;

        while (x != null) {
            //1. 判断是否要终止循环
            if (x == root || x.parent.color == BLACK) {
                //上升到根节点，break
                //父节点是黑色，不用处理
                break;
            }
            // 此时父亲一定是红色,祖父一定是黑色

            //2. 如果x的父亲 是 祖父的左儿子
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                //2.1 找到叔叔节点
                Node u = rightOf(parentOf(parentOf(x)));
                if (colorOf(u) == RED) {
                    //2.1.1 叔叔节点是红色，此时叔叔节点一定要存在
                    //2.1.2 此时叔叔节点和父亲节点都是红色
                    setColor(parentOf(x), BLACK);
                    setColor(u, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    //2.1.3 父亲和叔叔变黑，祖父变红，上升到祖父节点是红色,递归一直根节点然后变黑
                    x = parentOf(parentOf(x));
                } else {
                    //2.1.2 叔叔节点不存在，或者是黑色节点
                    //先看复杂代码
                    if (x == leftOf(parentOf(x))) {
                        //2.1.3 如果当前是左子
                        //a 将父亲变黑
                        //b 将祖父变红
                        //c 对祖父LL旋转
                        setColor(parentOf(x), BLACK);
                        setColor(parentOf(parentOf(x)), RED);
                        _rotateRight(parentOf(parentOf(x)));
                    } else {
                        //2.1.4 如果当前是右子
                        //a 对父RR旋转
                        x = parentOf(x);
                        //b 此时重置x为当前父元素
                        _rotateLeft(x);
                        //c 旋转完 ，变为上面一样x是红，x的父亲为红，x的叔叔为黑
                        //（x此时是插入节点的父节点）
                        setColor(parentOf(x), BLACK);
                        setColor(parentOf(parentOf(x)), RED);
                        _rotateRight(parentOf(parentOf(x)));
                    }
                }

            } else {
                //3. 另外中情况类似，就不测试了
                Node y = leftOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        _rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    _rotateLeft(parentOf(parentOf(x)));
                }
            }


        }

        root.color = BLACK;
    }

    // LL旋转即是左旋转，当左子的左子造成的不平衡时 调整
    private void _rotateRight(Node p) {
        if (p == null)
            return;
        Node l = p.left;
        //1. 如果l不存在，不能实现左旋
        if (l == null) {
            throw new UnsupportedOperationException("当前节点没有右子节点，不能左旋:" + p.val);
        }
        //2. 找出所有相关节点
        Node pp = p.parent;
//        Node ll = l.left;
        Node lr = l.right;
//        Node r = p.right;

        //3. 左旋
        //3.1 调整p和lr的的关系
        p.parent = l;
        p.left = lr;
        if (lr != null)
            lr.parent = p;
        //3.2 调整pp和l的关系
        if (pp == null) {
            root = l;
        } else if (pp.left == p) {
            pp.left = l;
        } else {
            pp.right = l;
        }
        l.parent = pp;
        //3.3 调整l和p关系
        l.right = p;
        p.parent = l;

    }


    // RR旋转即是左旋转，当右子的右子造成的不平衡时 调整
    private void _rotateLeft(Node p) {
        if (p == null)
            return;
        Node r = p.right;
        //1. 如果r不存在，不能实现左旋
        if (r == null) {
            throw new UnsupportedOperationException("当前节点没有右子节点，不能左旋:" + p.val);
        }
        //2. 找出所有相关节点
        Node pp = p.parent;
//        Node rr = r.right;
        Node rl = r.left;
//        Node l = p.left;
        //3. 右旋
        //3.1 调整p和rl关系
        p.right = rl;
        if (rl != null)
            rl.parent = p;
        //3.2 调整pp和r关系
        if (pp == null) {
            //3.2.1 表明p是root节点啊
            root = r;
        } else if (pp.left == p) {
            pp.left = r;
        } else {
            pp.right = r;
        }
        r.parent = pp;
        //3.3 调整r和p关系
        r.left = p;
        p.parent = r;
        //3.4 rr和l 没有影响
    }


    private void _levelPrint() {
        if (root == null) {
            return;
        }
        Queue<Node> queue = new LinkedList<>();
        queue.add(root);
        _levelPrint(queue);
    }

    private void _levelPrint(Queue<Node> queue) {
        while (!queue.isEmpty()) {
            Node x = queue.remove();
            System.out.println(StringUtils.format(
                    "当前:%s,左子:%s,右子:%s,层数：%s,颜色:%s",
                    x.val,
                    x.left == null ? "空" : x.left.val,
                    x.right == null ? "空" : x.right.val,
                    _getHeight(x),
                    x.color == BLACK ? "黑" : "红"
            ));
            Node left = x.left;
            Node right = x.right;
            if (left != null) {
                queue.add(left);
            }
            if (right != null) {
                queue.add(right);
            }
        }
    }


    private static int _getHeight(Node x) {
        if (x == null) {
            throw new NullPointerException("空节点不能获取高度");
        }
        int ret = 1;
        while (true) {
            x = x.parent;
            if (x == null) {
                break;
            } else {
                ret++;
            }
        }
        return ret;
    }

    private static final boolean RED = false;
    private static final boolean BLACK = true;

    private static <K, V> boolean colorOf(Node p) {
        return (p == null ? BLACK : p.color);
    }

    private static <K, V> Node parentOf(Node p) {
        return (p == null ? null : p.parent);
    }

    private static <K, V> void setColor(Node p, boolean c) {
        if (p != null)
            p.color = c;
    }

    private static Node leftOf(Node p) {
        return (p == null) ? null : p.left;
    }

    private static <K, V> Node rightOf(Node p) {
        return (p == null) ? null : p.right;
    }


    /**
     * 内部节点
     */
    private static class Node {
        private int val;
        private Node parent;
        private Node left;
        private Node right;
        private boolean color = BLACK;

        public Node(Node parent, int val) {
            this.parent = parent;
            this.val = val;
        }

        public Node(int val) {
            this.val = val;
        }

        public Node(int val, Node left, Node right) {
            this.val = val;
            this.left = left;
            this.right = right;
        }

        public Node(int val, Node parent, Node left, Node right) {
            this.val = val;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }


    }


    private static void _display(Node x) {
        System.out.println(StringUtils.format(
                "当前:%s,左子:%s,右子:%s",
                x.val,
                x.left == null ? "空" : x.left.val,
                x.right == null ? "空" : x.right.val
        ));
    }

    public static void main(String[] args) {
        RedBlackTree tree = new RedBlackTree();
        for (int i = 0; i < 100; i++) {
            tree.put(i);
        }

        tree._levelPrint();
        System.out.println(tree.size);
    }

    private static void _testRotateRight() {
        // 测试LL旋转
        Node p = new Node(0);
        Node l = new Node(1);
        Node r = new Node(2);
        Node ll = new Node(3);
        Node lr = new Node(4);

        p.left = l;
        l.parent = p;
        p.right = r;
        r.parent = p;
        l.left = ll;
        ll.parent = l;
        l.right = lr;
        lr.parent = l;

//        _rotateRight(p);
        _display(l);
        _display(ll);
        _display(p);
        _display(lr);
        _display(r);
    }
}

```

### 3.3 AVL树
### 3.4 B树
### 3.5 B+树
### 3.6 堆（以最大堆为例）

**（1） 插入算法**

	  由于需要维持完全二叉树的形态，需要先将要插入的结点x放在最底层的最右边，插入后满 足完全二叉树的特点； 
	  然后把x依次向上调整到合适位置满足堆的性质,例如下图中插入80,先将80放在最后,然后两次上浮到合适位置. 

**（2）删除算法**

	操作原理是：当删除节点的数值时，原来的位置就会出现一个孔,填充这个孔的方法就是，把最后的叶子的值赋给该孔并下调到合适位置，最后把该叶子删除。 
	
**（3）初始化**

	方法1：插入法： 
	  从空堆开始，依次插入每一个结点，直到所有的结点全部插入到堆为止。 
	  时间：O(n*log(n)) 
	  
	方法2：调整法： 
	    序列对应一个完全二叉树；从最后一个分支结点（n div 2）开始，到根（1）为止，依次对每个分支结点进行调整（下沉），以便形成以每个分支结点为根的堆，当最后对树根结点进行调整后，整个树就变成了一个堆。 
	  时间：O(n) 
	对如图的序列,要使其成为堆,我们从最后一个分支结点(10/2),其值为72开始,依次对每个分支节点53,18,36 45进行调整(下沉). 

**（4）堆排序：**

/*把根节点跟最后一个元素交换位置，调整剩下的n-1个节点，即可排好序*/    


**完整代码如下：**

```java
package com.carl.demo.algorithm;

import com.google.common.base.Joiner;
import com.google.common.collect.Lists;

import java.util.ArrayList;
import java.util.Random;

/**
 * Created by carl on 16-4-6.
 */
public class MaxHeap {
    private ArrayList<Integer> table;

    public MaxHeap() {
        this.table = Lists.newArrayListWithExpectedSize(16 + 1);
        this.table.add(null); //下标为0的位置不做任何处理
    }


    public MaxHeap(int[] array) {
        this.table = Lists.newArrayListWithExpectedSize(array.length + 1);
        this.table.add(null);
        for (int i = 0; i < array.length; i++) {
            this.table.add(array[i]);
        }
        //1. 找到最后一个位置的父节点
        int parent = _getParentIndex(table.size() - 1);
        //2. 从这里开始往上调整
        if (parent != 0) {
            for (int i = parent; i >= 1; i--) {
                _heapDown(i);
            }
        }
    }


    public static void sort(int[] array) {
        MaxHeap heap = new MaxHeap(array);
        System.out.println("开始：");
        heap._display();
        for (int i = 0; i < array.length; i++) {
            //交换第一个和最后一个位置
            int first = 1;
            int last = array.length - i;
            if (first >= last) {
                break;
            }
            ArrayList<Integer> table = heap.table;
            int temp = table.get(first);
            table.set(first, table.get(last));
            table.set(last, temp);
            //_heapDown()一次,last之前的位置
            heap._heapDown(first, last - 1);
            heap._display();
        }
        System.out.println("结束：");
        heap._display();
    }

    public void remove(int index) {
        //1.由于有下标0的存在，下标+1
        index = index + 1;
        if (index < 1 || index > table.size() - 1) {
            throw new IndexOutOfBoundsException("删除的下标超出范围");
        }
        //2.将最后一个元素放到当前位置并删除最后一个额元素
        table.set(index, table.get(table.size() - 1));
        table.remove(table.size() - 1);
        //3.算法下移
        _heapDown(index);
    }

    private void _heapDown(int index) {
        _heapDown(index, table.size() - 1);
    }

    private void _heapDown(int index, int bound) {
        int now = index;
        if (now > bound) {
            //删除的刚好是最后一个元素
            return;
        }
        int leftChild;
        int nowVal, target, targetVal;
        nowVal = table.get(now);
        while (true) {
            //1.查看左子，右子
            leftChild = _getLeftChildIndex(now);
            if (leftChild > bound) {
                //2.左子都不存在
                table.set(now, nowVal);
                break;
            } else {
                target = _findTarget(leftChild, bound);
                targetVal = table.get(target);
                if (targetVal > nowVal) {
                    table.set(now, targetVal);
                    now = target;
                } else {
                    table.set(now, nowVal);
                    break;
                }
            }
        }


    }

    private int _findTarget(int leftChild, int bound) {
        int rightChild = leftChild + 1;
        //如果没有右子
        if (rightChild > bound) {
            return leftChild;
        }
        //否则,找最大的儿子
        int rightVal = table.get(rightChild);
        int leftVal = table.get(leftChild);
        if (leftVal > rightVal) {
            return leftChild;
        }
        return rightChild;
    }


    /**
     * 插入算法
     *
     * @param key
     */
    public void add(int key) {
        table.add(key);
        _heapUp(table.size() - 1);
    }

    private void _heapUp(int x) {
        int now = x;
        int parent;
        int parentVal;
        int nowVal = table.get(now);
        while (true) {
            parent = _getParentIndex(now);
            //1. 如果父元素不存在，表示已经是根了
            if (parent == 0) {
                table.set(now, nowVal);
                break;
            }
            //2. 和父元素比较大小
            parentVal = table.get(parent);
            if (nowVal > parentVal) {
                //3.如果比父元素大，父元素要下移
                table.set(now, parentVal);
                now = parent;
            } else {
                //4.如果比父元素要小，不动
                table.set(now, nowVal);
                break;
            }
        }
    }


    private void _display() {
        Joiner joiner = Joiner.on(",").skipNulls();
        System.out.println(joiner.join(table));
    }

    private static int _getLeftChildIndex(int x) {
        return 2 * x;
    }

    private static int _getRightChildIndex(int x) {

        return 2 * x + 1;
    }

    private static int _getParentIndex(int x) {
        return x / 2;
    }

    private static void _testAdd() {
        MaxHeap heap = new MaxHeap();
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            int now = random.nextInt(50);
            heap.add(now);
            System.out.println("now:" + now);
            heap._display();
            System.out.println("------------------");
        }
    }

    private static void _testRemove() {
        MaxHeap heap = new MaxHeap();
        heap.add(40);
        heap.add(30);
        heap.add(20);
        heap.add(25);
        heap.add(35);
        heap.add(11);
        heap.add(16);
        heap._display();
        heap.remove(0);
        heap._display();
        heap.remove(5);
        heap._display();
        heap.remove(3);
        heap._display();
    }

    private static void _testAdjust() {
        int size = 5;
        int[] array = new int[size];
        Random random = new Random();
        for (int i = 0; i < array.length; i++) {
            array[i] = random.nextInt(50);
        }
        System.out.println(java.util.Arrays.toString(array));
        array = new int[]{28, 49, 47, 42, 22};
        MaxHeap heap = new MaxHeap(array);
        heap._display();
    }

    private static void _testSort() {
        int size = 10;
        int[] array = new int[size];
        Random random = new Random();
        for (int i = 0; i < array.length; i++) {
            array[i] = random.nextInt(50);
        }
        sort(array);
    }


    public static void main(String[] args) {
        _testRemove();
    }
}
```


### 3.7 使用哈夫曼树进行压缩