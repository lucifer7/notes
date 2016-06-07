
### Difference between cookie, localStorage, sessionStorage
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
