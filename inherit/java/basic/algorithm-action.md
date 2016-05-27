# Algorithm-基础篇

[TOC]


## 一、字符串

### 1.1 字符串空格替换

[描述](http://blog.csdn.net/believejava/article/details/38682361)

**思路：**
		
		1 找出所有空格，确定最终字符串的长度
		2 按照新长度，创建新的字符串并将原字符串的内容拷贝
		3 使用2个指针p1,p2 p2去追赶p1
		

```java
package com.carl.demo.algorithm;


/**
 * Created by carl on 16-4-7.
 */
public class StringDemo {

    /**
     * 从字符串的后面开始复制和替换，
     * 首先准备两个指针，p1和p2，
     * p1指向原始字符串的末尾，
     * p2指向替换后字符串的末尾，
     * 接下来，向前移动指针p1，
     * 逐个把它指向的字符复制到p2，
     * 碰到一个空格之后，把p1向前移动1格，
     * 在p2处插入字符串“20%”，由于“20%”长度为3，
     * 同时也要把p2向前移动3格。直到p1=p2，
     * 表明所有空格都已经替换完毕。
     */
    private static String replaceBlank(String src) {
        //1.不考虑null情况
        //2.获取空格总数并且创建新数组并且拷贝
        int numbers = _getBlankNum(src);
        int newLength = numbers * 2 + src.length();
        char[] newString = new char[newLength];
        System.arraycopy(src.toCharArray(), 0, newString, 0, src.length());
        //3.创建2个指针p1,p2
        int p1 = src.length() - 1;
        int p2 = newLength - 1;
        //4.开始数组移位
        while (true) {
            //4.1 如果越界，终止循环
            if (p1 < 0) {
                break;
            }
            //4.2 如果p2已经追到p1，表示没有空格了
            if (p1 == p2) {
                break;
            }
            char nowChar = newString[p1];
            if (Character.isWhitespace(nowChar)) {
                //4.3 如果是空格
                newString[p2--] = '%';
                newString[p2--] = '0';
                newString[p2--] = '2';
                p1--;
            } else {
                //4.4 如果不是空格
                newString[p2--] = newString[p1--];
            }
        }
        return new String(newString);
    }


    private static int _getBlankNum(String src) {
        int total = 0;
        for (int i = 0; i < src.length(); i++) {
            char c = src.charAt(i);
            if (Character.isWhitespace(c)) {
                total++;
            }
        }
        return total;
    }

    public static void main(String[] args) {
        String s = " 1a b  c  ";
        System.out.println(replaceBlank(s));
    }
}

```

### 1.2 Manacher-回文字符串算法，入门篇中有具体实现


## 二、线性结构

### 2.1  使用2个栈实现队列，使用2个对列实现栈

[来源](http://www.cnblogs.com/kaituorensheng/archive/2013/03/02/2939690.html)

（1） 2个栈实现队列：
    
	s1是入栈的，s2是出栈的。
	入队列：直接压入s1即可
	出队列：如果s2不为空，把s2中的栈顶元素直接弹出；否则，把s1的所有元素全部弹出压入s2中，再弹出s2的栈顶元素



```java
package com.carl.demo.algorithm;

import com.alibaba.dubbo.common.utils.Stack;

/**
 * Created by carl on 16-4-7.
 */
public class DoubleStackQueueImpl {

    private Stack<String> firstStack = new Stack<>();

    private Stack<String> nextStack = new Stack<>();

    public void add(String key) {
        firstStack.push(key);
    }

    public String remove() {
        if (nextStack.isEmpty()) {
            _move();
        }
        return nextStack.pop();
    }


    private void _move() {
        while (firstStack.size() > 0) {
            nextStack.push(firstStack.pop());
        }
    }

}
```

（2） 2个队列实现栈

	   q1是专职进出栈的，q2只是个中转站。元素集中存放在一个栈中，但不是指定(q1 或 q2)。
	    需要指定目前哪个队列是入栈队列

入栈：直接入pushtmp所指队列即可
出栈：把pushtmp的除最后一个元素外全部转移到队列tmp中,然后把刚才剩下q1中的那个元素出队列

```java
package com.carl.demo.algorithm;

import java.util.LinkedList;
import java.util.Queue;

/**
 * Created by carl on 16-4-7.
 */
public class DoubleQueueStackImpl {
    private Queue<String> first = new LinkedList<>();
    private Queue<String> next = new LinkedList<>();

    /**
     * 当isFirst为true时，first是当前队列
     */
    private boolean isFirst = true;


    public void push(String key) {
        if (isFirst)
            //1. 如果当前队列是first，入first
            first.add(key);
        else
            //2. 如果当前队列是next，入next
            next.add(key);
    }


    public String pop() {
        //1. 先移动所有元素，只剩最后一个
        _move();
        //2. 最后一个就是所求
        String ret = isFirst ? first.remove() : next.remove();
        //3. 修改当前入栈队列
        isFirst = !isFirst;
        return ret;
    }

    private void _move() {
        if (isFirst) {
            //1. 从first->next
            while (first.size() != 1) {
                next.add(first.remove());
            }
        } else {
            //2. 从next->first
            while (next.size() != 1) {
                first.add(next.remove());
            }
        }
    }

}
```


### 2.2 二维有序数组查找问题

[问题介绍](http://www.2cto.com/kf/201506/407300.html)


```java
package com.carl.demo.algorithm;

/**
 * Created by carl on 16-4-7.
 */
public class ArraySearchDemo {

    public static boolean find(int[][] array, int target) {
        boolean ret = false;
        int col = 0;
        int row = array[0].length - 1;
        while (true) {
            //1. 如果超出限制，跳出
            if (col == array.length || row < 0) {
                break;
            }
            //2. 如果只剩下一行了
            if (col == array.length - 1) {
                System.out.println("当前行是:" + (col + 1));
                return binarySearch(array[col], row, target);
            }
            //3. 找出右上角的数据
            int nowVal = array[col][row];
            if (nowVal == target) {
                //3.1 如果找到了当前数据
                System.out.println("当前行是:" + (col + 1) + ",当前列是:" + (row + 1));
                ret = true;
                break;
            } else if (nowVal > target) {
                //3.2 如果当前数据小一点，当前列是不可能的
                row--;
            } else {
                //3.3 当前数据大一点，当前行是不可能的
                col++;
            }
        }
        return ret;
    }

    /**
     * <pre>
     *     从0到limit位置查找target,若有返回true,否则false
     * </pre>
     */
    private static boolean binarySearch(int[] src, int limit, int target) {
        boolean ret = false;
        //可以二分搜索
        int start = 0;
        int end = limit;
        while (true) {
            if (start > end) {
                //1. 没有查到，break了
                break;
            } else {
                //2. 取中值
                int mid = (start + end) >> 1;
                int midVal = src[mid];
                if (midVal == target) {
                    //2.1 找到了
                    System.out.println("当前列是:" + (mid + 1));
                    ret = true;
                    break;
                } else if (midVal > target) {
                    //2.2 target小一点，应该在mid左边
                    end = mid - 1;
                } else {
                    //2.3 target大一点，应该在mid右边
                    start = mid + 1;
                }
            }
        }
        return ret;
    }


    public static void main(String[] args) {
        int[][] arr = {
                {1, 2, 8, 9},
                {2, 4, 9, 12},
                {4, 7, 10, 13},
                {6, 8, 11, 15},
                {8, 10, 14, 17},
                {8, 25, 21, 20}
        };
        find(arr,25);
    }
}

```


## 三、位运算

### 3.1 找出数组中两个只出现一次的数字

**思路：**

**xor运算规律:**

		1. a ^ 0 = a , a ^ 1 =~a , a ^ a = 0
		2. 如果x ^ y = 0 那么x = y （反证）
		3. 0 ^ 1 = 1 ,其他 1 ^ 1, 0 ^0 都是0

**&运算规律**
	
	1. 1 & 1 = 1 其他都是0

所有的数全部^运算，就是那2个不同数x和y的 x^y = s
寻找到s二进制第一个为1的位置和相应的值count
根据这个位置分为两组（用反正法，x和y必定不可能在同一组）
so ？ 搞定

	


```java
package com.carl.demo.test;

import com.google.common.primitives.Ints;

/**
 * Created by carl on 16-4-21.
 */
public class XorDemo {

    private static void _testXOr() {
        System.out.println(1 ^ 0); //1
        System.out.println(0 ^ 0); //0
        System.out.println(1 ^ 1); //0
    }


    private static void _testAnd() {
        System.out.println(1 & 0);
        System.out.println(0 & 0);
        System.out.println(1 & 1);
    }

    public static void main(String[] args) {
        int[] src = {1,1,3,2,4,6,2,6,4,7};
        findTheSingleTwo(src);
    }

    public static void findTheSingleTwo(int[] ary) {
        int x = 0;
        for (int i = 0; i < ary.length; i++) {
            x = x ^ ary[i];
        }

        int y = _findFirstSameBitIndex(x);
        int num1 = 0;
        int num2 = 0;

        for (int i = 0; i < ary.length; i++) {
            if ((y & ary[i]) == 0) {
                num1 = num1 ^ ary[i];
            } else {
                num2 = num2 ^ ary[i];
            }
        }
        System.out.println("num1:" + num1);
        System.out.println("num2:" + num2);

    }

    private static int _findFirstSameBitIndex(int s) {
        System.out.println(Integer.toBinaryString(s));
        int result = s;
        int count = 1;

        while (true) {
            if ((result & 1) == 1) {
                break;
            }
            result = result >> 1;
            count = count << 1;
        }
        return count;
    }

}

```