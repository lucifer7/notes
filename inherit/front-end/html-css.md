# front end


[TOC]



## html

先看几个不熟悉的标签

1. `<dl>` 这种跟列表有关
2. `<marquee>` 可以自动滚动
3. `<pre>` 代码


## css

1. RGB 直接看以下demo:
- `#ff ff ff` : 白色，整体都比较亮
- `#00 00 00` : 黑色，整体比较暗
- `#ff 00 00` : 红色
- `#21 22 67` : 数字都比较小，所以比较暗，偏向67黑色

2. 盒子模型

- margin padding 12点钟方向顺时针

- 推荐的居中方式, 兼容大部分浏览器
    ```css
            .container {
                border: 1px solid blue;
                width: 1100px;
                height: 500px;
                position: absolute;
                left: 50%;
                margin-left: -550px;
            }
    ```
3. 关于position

- absolute ：寻找父类最近的absolute, 以他为基准，有时候不用float，用absolute来定位更简单
```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>position</title>
        <style>
            body{
                padding: 50px;
            }
            .d1{
                width: 400px;
                height: 400px;
                border: 1px solid red;
                position: absolute;
                left:50px;
            }

            .d2{
                width: 200px;
                height: 200px;
                border: 1px solid blue;
                position: absolute;
                right: 20px;
            }
        </style>
    </head>
    <body>
        <div class="d1">d1
            <div class="d2"></div>
        </div>
    </body>
    </html>
```

- fixed
- relative 经常用来定位，非常有用



## javascript

### 基础概念-函数



-  1 **首先关注函数的2种定义方式**：`function f1(){...}`和`var f1 = function()`，前者是声明，在一个script文件中可以在任何位置声明,后者需要按顺序执行
```javascript
    	f1();//ok
        function f1(){
        }

        f2();//wrong
        var f2 = function(){}
```
	
函数的赋值 直接复制对象，函数是一个特殊的对象
```javascript
     function f1() {
        alert("1111");
    }

    var f2 = f1;
    f2 = function () {
        alert("2222");
    }
    f1(); //1111
    f2(); //2222
    
    //一般对象的赋值是一种引用传递
    var o1 = {}
    o1['key'] = '111';
    var o2 = o1;
    o2['key'] = '222';
    alert(o1['key']);
```
	
-  2 **函数的传值**
	函数是一个对象：可以当作参数，也可以当做**返回值**
```javascript
    //js是支持函数式编程的语言
    //函数可以当作参数
    function callFunc(fn, args) {
        fn(args);
    }

    var f1 = function (num) {
        alert(num);
    }

    /*callFunc(f1, "1");
     callFunc(f1, "2");*/

    //函数也可以当作返回值，仿佛扩充了参数的作用域
    var returnFunc = function (args) {
        var ret = function (num) {
            return args + num;
        }
        return ret;
    }

    var ret = returnFunc(1);
    alert(ret(2));
    alert(ret(3));
    ```
	
	排序的例子(使用了函数返回值和闭包)
```javascript
     //最简单的例子，按照对象属性排序
    var Person = function (age, name, country) {
        this.age = age;
        this.name = name;
        this.country = country;
    }

    Person.prototype = {
        constructor: Person,
        toString: function () {
            return "[name:" + this.name + ",age:" + this.age + ",country:" + this.country + "]";
        }
    }

    var p1 = new Person("1", "c", "TW");
    var p2 = new Person("5", "a", "HK");
    var p3 = new Person("3", "d", "SH");
    var p4 = new Person("4", "b", "BJ");

    ary = [p1, p2, p3, p4];

    ary.sort((function sortByProperty(p) {
        var ret = function (a, b) {
            if (a[p] == b[p])
                return 0
            return a[p] > b[p] ? 1 : -1;
        }
        return ret;
    })("age"));

    for (var i = 0; i < ary.length; i++) {
        console.info(ary[i].toString());
    }
```

### 基础概念-arguments and this

```javascript
 var color = "red";

    function showColor() {
        alert(this.color);
    }


    //this主要看函数调用者是谁
    function Fn(color) {
        this.color = color;
        this.fn = showColor;
    }

    var c = new Fn("yellow");
    c.fn(); // this 是c对象
    showColor(); // this 是当前window对象

    showColor.call(window); //调用者是当前window对象
    showColor.call(c); //调用者是c对象

    // 使用call和apply可以在对象中定义方法了
```



### 基础概念-prototype:

关于原型对象：

```javascript
    function Person() {
    }

	var p1 = new Person();
    Person.prototype.sayHi = function () {
        alert(this.name);
    }
	
    p1.sayHi(); //不会报错
    
    /**
     * 检测原型中是否有属性
     */
    function hasPrototypeProperty(obj, prop) {
        return (!obj.hasOwnProperty(prop)) && (prop in obj);
    }
    
      // 原型重写需要重新制定contructor
    Person.prototype = {
        constructor: Person,
        name: "carl"
    }
    
    var p2 = new Person();
    p2.sayHi(); //原型已经重写，所以报错
```

**实现继承**

```javascript
 	function extend(sub, sup) {
        var F = new Function();
        F.prototype = sup.prototype;
        sub.prototype = new F();
        sub.prototype.constructor = sub;
        sub['superClass'] = sup.prototype;
        if (sup.prototype.constructor == Object.prototype.constructor) {
            sup.prototype.constructor = sup;
        }
    }

    function Super(age, name) {
        this.age = age;
        this.name = name;
    }

    Super.prototype.say = function () {
        alert(this.name + "," + this.age);
    }

    function Sub(sex) {
        this.sex = sex;
    }

    Sub.prototype.hello = function () {
        alert(this.age + "," + this.name + "," + this.sex);
    }

    extend(Sub, Super);
```

### 基础概念-作用域链，闭包等


- 1. 理解了函数的作用域链就理解了闭包

```javascript
        function fn1() {
            var fns = new Array();
            for (var i = 0; i < 10; i++) {
                var tf = function (num) {
                    fns[num] = function () {
                        return num;
                    }
                }
                tf(i);
            }
            return fns;
        }

        var fs = fn1();
        for (var i = 0; i < fs.length; i++) {
            document.write(fs[i]());
        }
```

- 2.理解闭包中的this

```javascript
	var name = "window";
        var person = {
            name: "123",
            say: function () {
                return function () {
                    alert(this.name);
                }
            }
        };
        person.say()(); // window
        person.say().call(person,person.say()); // 123
```

>   a.视频+自己debug比较容易观察
>   b.js中没有块级作用域的概念, 防止冲突可以用匿名函数

-  3.匿名函数
```javascript
 (function () {
	for (var i = 0; i < 10; i++) {
                    //do something
	}
})();
alert(i); //undefined or error  
```


