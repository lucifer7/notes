### HTML5 本地存储
#### IndexedDB
http://www.cnblogs.com/dolphinX/p/3415761.html
#### Difference between cookie, localStorage, sessionStorage
1. cookie由服务端生成，用于标识用户身份；而两个storage用于浏览器端缓存数据
2. 三者都是键值对的集合
3. 一般情况下浏览器端不会修改cookie，但会频繁操作两个storage
4. 如果保存了cookie的话，http请求中一定会带上；而两个storage可以由脚本选择性的提交
5. 会话的storage会在会话结束后销毁；而local的那个会永久保存直到覆盖。cookie会在过期时间之后销毁。
6. 安全性方面，cookie中最好不要放置任何明文的东西。两个storage的数据提交后在服务端一定要校验（其实任何payload和qs里的参数都要校验）。
[See Ref](http://jerryzou.com/posts/cookie-and-web-storage/)

### 无刷新跳转 | 局部刷新 | 缓存页面
1. PJAX
pjax是对ajax + pushState的封装，让你可以很方便的使用pushState技术。
同时支持了缓存和本地存储，下次访问的时候直接读取本地数据，无需在次访问。
并且展现方式支持动画技术，可以使用系统自带的动画方式，也可以自定义动画展现方式。

开源主页 ： https://github.com/welefen/pjax
雅虎的实现： http://yuilibrary.com/yui/docs/pjax/

1. for fallback browser
use URL hash instead(onhashchange)
OR:
use pludgin: history.js
Home page:  ..

### Quickling/Page cache 解决方案
https://github.com/fex-team/fis-plus/wiki/quickling

http://www.slideshare.net/ajaxexperience2009/chanhao-jiang-and-david-wei-presentation-quickling-pagecache

HTML 属性全称

作者：贺师俊
链接：https://www.zhihu.com/question/47896214/answer/108198960
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

那些比较明显的（如 p、h1~h6、ol、ul、li、abbr、col、img、ins、del、q 等）我就不写了。

a: anchor
link的端点叫做anchor。link是从一端指向到另一端，通常a通过href属性指向外部资源，此a即为起始端点，但早期a也可以用name属性表示文档中可以被链接到的目标端点（后来任意元素上有id属性均可成为目标端点）。

b: bold
i: italic
u: underlined
s: strike-through
注意以上四个元素现在的语义都和原来偏重样式的定义不同了。

bdi: bidi isolated
bdo: bidi override

br: (line) break
wbr: word break
hr: horizontal rule

dl: definition list
dt: definition term
dd: definition description

dfn: (instance) definition

iframe: inline frame

var: variable
kbd: keyboard
samp: sample

sub: subscript
sup: superscript

pre: preformatted

rt: ruby text
rp: ruby parenthesis 

以上。

注意，这里写的是当初起名字时候的单词，但是精确的语义还是要看现在的spec，因为有些元素的语义已经和当初不完全一致了。