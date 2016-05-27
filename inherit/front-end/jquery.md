# JQUERY
<style>
	.red{
    	color:#f00;
    }
</style>

[TOC]

# 参考文档

[IE和FireFox中JS兼容之event .](http://blog.csdn.net/zgmzyr/article/details/7448973)

## 1. Basic

### form - selected

使用表单操作select

```javascript
 $(function () {
        $("#btn").click(function () {
            selectValue("interest", "篮球");
        });
    });


    function selectValue(name) {
        //先清除以前所有的选择项
        $("input[name='" + name + "']").each(function () {
            $(this).removeAttr("checked");
        });
        //最好是以数组的方式传值
        var values = [];
        for (var i = 1; i < arguments.length; i++) {
            values.push(arguments[i]);
        }
        $("input[name='" + name + "']").val(values);
        for (var i = 0; i < values.length; i++) {
            $("input[name='" + name + "'][value='" + values[i] + "']").attr("checked", "checked");
        }
    }

    function getSelectedValues(name) {
        var ret = [];
        $("input[name='" + name + "']:checked").each(function () {
            ret.push($(this).val());
        });
        return ret;
    }
```

### jquery-event on live delegate...

-  **传统的获取事件源**

```javascript
  $(function () {
        var all = $("*");
        all.each(function () {
        	//DOM0事件模型
            this.onclick = function (event) {
                //获取事件对象
                event = event ? event : window.event;
                //获取事件源
                var target = event.target ? event.target : event.srcElement;
                //可以通过以下方法来取消事件的冒泡
//                event.stopPropagation();//非ie浏览器
                event.cancelBubble = true; //ie浏览器
                print("target.id:" + target.id + ",target.tagName:" + target.tagName + ",this.id:" + this.id);
            }
        });
    });

    function print(txt) {
        $("#content").append(txt + "<br/>");
    }

     /**
     * 单体模式 dom2
     * 实现一个跨浏览器的事件处理程序
     */
    EventUtil = {
        addHandler: function (element, type, handler) {
            if (element.addEventListener) {		//FF
                element.addEventListener(type, handler, false);
            } else if (element.attachEvent) {		//IE
                element.attachEvent('on' + type, handler);
            }
        },
        removeHandler: function (element, type, handler) {
            if (element.removeEventListener) {		//FF
                element.removeEventListener(type, handler, false);
            } else if (element.detachEvent) {		//IE
                element.detachEvent('on' + type, handler);
            }
        }
    };
```

- `juqery`  有效的解决了这个问题(bind)

```javascript
 $(function () {
 		//使用bind方法成功解决了以上的兼容性问题
        var all = $("*");
        all.each(function () {
            $(this).bind("click", {index: 1}, function (event) {
                print("事件源:" + event.target.nodeName + ",this:" + this.tagName);
                //event.stopPropagation(); //通过使用 stopPropagation() 方法只阻止一个事件起泡。
                // event.preventDefault(); //通过使用 preventDefault() 方法只取消默认的行为。
                //return false; //通过返回false来取消默认的行为并阻止事件起泡。
            });
        });
    });

    function print(txt) {
        $("#content").append(txt + "<br/>");
    }
```

- **live** 方法的原理

```javascript
	 $(document).bind("click", function (event) {
            var target = $(event.target);
            var obj = target.closest(".aaa");
            if (event.target == obj[0]) {
                //事件源地址是符合条件的，就执行回调函数
                alert(target.html());
            }
        });
```

	>1. 开销太大，可以指定上下文，否则是默认的document
	>2. 某些默认的方法不会执行


```javascript
  $(function () {

        //其中 .container表示的是上下文
        $(".aaa", ".container").live("click", function (event) {
            var target = $(event.target)
            alert(target.html());
            return false;
        });


        $(".container").append('<a href="111" class="aaa">222</a>');
    });
```

- **delegate**:之后的版本推荐使用这种写法，原理跟live相同，使用delegate委派必须指定上下文

```javascript
 		//其中 .container表示的是上下文
        $(".container").delegate(".aaa","click",function(event){
            var target = $(event.target)
            alert(target.html());
            return false;
        });
```

- **on** 同时拥有bind和live的功能

```javascript
	$(function () {

        //注意,on中没有 selector参数，此时的on相当于bind
        $(".aaa").on("click", function (event) {
            var target = $(event.target)
            alert(target.html());
            return false;
        });

        $(".container").append('<a href="111" class="aaa">222</a>');
    });
```

下面用on模拟live方法

```javascript
	$(function () {
        //注意,on中没有 selector参数，此时的on相当于bind
        $(".container").on("click", ".aaa", {index: "what ever"}, function (event) {
            alert(event.data.index);
            return false;
        });

        $(".container").append('<a href="111" class="aaa">222</a>');
    });
```


### 鼠标事件详解

1. mouseover 和 mouseout 由于会事件有冒泡的影响，jquery中使用mouseenter 和 mouseleave来代替

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent1, #parent2 {
            height: 200px;
            width: 300px;
            background: #8e7;
            margin: 40px;
        }

        #child1, #child2 {
            height: 60px;
            width: 150px;
            background: #843;
            position: relative;
            left: 30px;
            top: 30px;
        }
    </style>
</head>
<body>
<div id="parent1">
    parent1
    <div id="child1">chidl1</div>
</div>

<div id="parent2">
    parent2
    <div id="child2">chidl1</div>
</div>

<div id="content"></div>
<script type="text/javascript" src="./common/jquery-1.8.0.min.js"></script>
<script>


    //
    $(function () {
        $("#parent1").on("mouseenter mouseleave", function (event) {
            print(event.type + "," + this.id);
        });
    });

    function print(txt) {
        $("#content").append(txt + "<br/>");
    }

</script>
</body>
</html>
```

### trigger和triggerHandler

1. 触发其它的事件，trigger不会阻止事件的冒泡mtriggerHandler只会触发用户自己绑定的事件
2. 都可以传递参数

```javascript
<script>
    $(function () {
        $("#a").on("click", function (event, a, b) {
            alert($(this).html() + "," + a + "," + b);
        });

        $("#b").on("click", function (event) {
            $("#a").trigger("click", [1,2]);
        });
    });
</script>
```

### Jquery工具箱


## 2.插件开发


### 2.1插件开发基础
jQuery 插件都是以外链的方式引入，默认规则，按照功能定义，例如:`jquery.util.movie.js`

*代码如下*:

```javascript
/**
 * Created by carl on 16-2-17.
 */
// 使用闭包来防止冲突，将jQuery作为参数传递

(function ($) {
    $.say = function () {
        alert("hello world");
    }
	
    //使用extend满足可变参数
    $.complex = function (p1, options, p2) {
        var settings = $.extend({a2: "ok", a3: "hello"}, options || {});
        alert("p1:" + p1 + ",p2:" + p2);
        alert("a2:" + settings.a2+",a3:"+settings.a3);
    }

})(jQuery);

```

下面来研究extend这个方法
```javascript
//合并 settings 和 options，修改并返回 settings。
var settings = { validate: false, limit: 5, name: "foo" };
var options = { validate: true, name: "bar" };
var settings = jQuery.extend(settings, options);
// 结果是settings == { validate: true, limit: 5, name: "bar" }
var empty = {};
var defaults = { validate: false, limit: 5, name: "foo" };
var options = { validate: true, name: "bar" };
var settings = jQuery.extend(empty, defaults, options);
//settings == { validate: true, limit: 5, name: "bar" }
//empty == { validate: true, limit: 5, name: "bar" }

```

### 2.2开发插件支持confirm
```javascript
/**
 * Created by carl on 16-2-29.
 * jQuery 竞价插件
 */
(function ($) {

   
    $.liveConfirm = function (context, child, options) {
        var source = "#confirm-panel";
        if (options['source']) {
            source = options['source'];
        }
        if ($(source).length > 0) {
        } else {
            console.error("can't find element '" + source + "' in page");
            return;
        }
        var settings = $.extend({
            ok: function (event, data) {
                //ok回调函数
            },
            cancel: function (event, data) {
                //cancel回调函数
            },
            msg: "confirm the operation?", //显示信息
            show: function (e, data) { //展示confirm框自定义方法
                e.stopPropagation();
                var x = this.offset().top;
                var y = this.offset().left;
                if (e.clientY > ($(window).height() - $('.delete-popup').height())) {
                    $(source).css('top', x - $('.delete-popup').height());
                }
                else {
                    $(source).css('top', x + $(this).height());
                }
                $(source).css('display', 'none').css('left', y - $(source).width() + $(this).width() + 80).fadeIn();
            },
            source: source,
            data: {}
        }, options || {});

        $(context).on("click", child, function (event) {
            var panel = $(settings['source']);
            //1.为当前的对象赋值引用
            var _t = $(this);
            //2.点击click展示页面
            settings['show'].call(_t, event, settings['data']);
            //3.绑定事件
            var confirmLink = panel.find("a").eq(0);
            var cancelLink = panel.find("a").eq(1);
            confirmLink.off();
            cancelLink.off();
            confirmLink.one("click", function () {
                settings['ok'].call(_t, event, settings['data']);
                panel.hide();
            });
            cancelLink.one("click", function () {
                settings['cancel'].call(_t, event, settings['data']);
                panel.hide();
            });
        });
    }


    $.fn.confirm = function (options) {
        var source = "#confirm-panel";
        if (options['source']) {
            source = options['source'];
        }
        if ($(source).length > 0) {
        } else {
            console.error("can't find element '" + source + "' in page");
            return;
        }

        var settings = $.extend({
            ok: function (e, data) {
            },
            cancel: function (e, data) {
            },
            msg: "confirm the operation?",
            source: source,
            data: {},
            show: function (e, data) {
                e.stopPropagation();
                var x = this.offset().top;
                var y = this.offset().left;
                if (e.clientY > ($(window).height() - $('.delete-popup').height())) {
                    $(source).css('top', x - $('.delete-popup').height());
                }
                else {
                    $(source).css('top', x + $(this).height());
                }
                $(source).css('display', 'none').css('left', y - $(source).width() + $(this).width() + 80).fadeIn();
            }
        }, options || {});
        // this 就是当前jQuery对象
        var _t = this;
        _t.off();
        _t.on("click", function (event) {
            $("#confirm-msg-span").html(settings['msg']);
            //打开页面
            var panel = $(settings['source']);
            settings['show'].call(_t, event, settings['data']);
            var confirmLink = panel.find("a").eq(0);
            var cancelLink = panel.find("a").eq(1);
            confirmLink.off();
            cancelLink.off();
            confirmLink.one("click", function (event) {
                settings['ok'].call(_t, event, settings['data']);
                panel.hide();
            });
            cancelLink.one("click", function (event) {
                settings['cancel'].call(_t, event, settings['data']);
                panel.hide();
            });
        });
    }
})(jQuery);
```

	以上是工作中自己写的confirm.js插件，支持2种情况
	1. 第一个方法liveConfirm是支持动态生成html内容，类似`live`方法
	2. 第二个方法则是支持 `bind`方法





## 3.Jquery常用插件

### AutoComplete


**Hello World:**
```javascript

<script type="application/javascript" src="./js/jquery-1.10.2.min.js"></script>
<script type="application/javascript" src="./js/jquery-ui-1.10.0.custom.min.js"></script>
<script>
    $(function () {
        var sources = ["java", "c++", "c", "ruby", "python", "javascript", "php", "jsp", "asp"];
        $("#tags").autocomplete({
            source: sources,
            minLength: 0,
            change: function (event, ui) {
                alert("发生change事件");
            },
            //选择时候的事件
            focus: function (event, ui) {
                alert("发生focus事件");
            },
            //搜索完成之前的事件，会得到搜索出来的一组元素
            response: function (event, ui) {
                alert("发生response事件");
            },
            //选择完成之后触发的事件
            select: function (event, ui) {
                alert("发生select事件");
            }
        });
    });
</script>
```




**Events:**

- change( event, ui ):
	Triggered when the field is **blurred**, if the value has changed.
(这个field发生blur事件的时候触发，如果当前值改变的话)
<br/>
- focus( event, ui ):
	Triggered when focus is moved to an item (not selecting). The default action is to replace the text field's value with the value of the focused item, though only if the event was triggered by a **keyboard interaction**.
Canceling this event prevents the value from being updated, but does not prevent the menu item from being focused.
<br/>
-  response:
	Triggered after a **search** completes, before the menu is shown. Useful for local manipulation of suggestion data, where a custom source option callback is not required. This event is always triggered when a search completes, even if the menu will not be shown because there are no results or the Autocomplete is disabled.
<br/>
- search( event, ui )Type: autocompletesearch
	Triggered before a search is performed, **after minLength and delay are met**. If canceled, then no request will be started and no items suggested.



