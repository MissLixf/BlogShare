### 前端路由与后端路由
>鲁迅曾经说过，这个世界上本没有路，走的人多了便成了路，鲁迅还说过，路由本不分前端路由和后端路由，随着单页应用（SPA）的出现，便有了前端路由。
鲁迅：这话我没说过:sweat:
好吧，这的确不是鲁迅说的，但路确实是走出来的，前端路由也确实是随着单页应用的不断普及，前后端分离而演化出来的。

#### 论前端路由的进化史
最开始只有路由这个名词，路由就是一个单纯的URL，也就是一个链接地址，在浏览器里输入这个链接地址向服务器发起请求，服务器根据这个地址拿到对应的html文件返回给浏览器渲染（若html文件中引入了外部的js或css或其他文件资源，浏览器再一次发送请求，服务器再依次返回）。用户首次访问一个网站，输入地址后回车发起请求，服务器响应返回首页的html文件，浏览器渲染；用户点击详情页，浏览器发起请求，服务器响应返回详情页html文件；用户点击个人主页，浏览器发起请求，服务器响应返回个人主页的html文件......，然而有时候，这个html文件并不小，或者很大，甚至庞大！这就导致服务器负荷重，页面加载缓慢，用 户体验差。
###### 所以这整个过程中，存在着不平等。。。
前端：我只管往出丢路由,其他的与我无瓜！

服务器： ![](https://user-gold-cdn.xitu.io/2019/6/29/16ba247120f2dfc8?w=234&h=213&f=png&s=45593)

为了增强用户体验，让用户更快更流畅的访问网站，亦为了追求平等，单页应用出现了。单页应用就是整个项目只有一个html文件，用户首次访问网站，服务器返回一个html文件以及打包好的js文件和css文件，如果你不刷新，这将是唯一一次给你返回html文件。当你切换页面时，只会请求接口返回JSON数据，渲染页面。（如果使用了路由按需加载，每次切换页面时会加载对应的组件。）这时候点击详情页或者个人主页时浏览器并不会发送请求让服务器返回对应的详情页.html或个人主页.html。那浏览器是怎么知道点击详情页或者个人主页去渲染不同的界面呢？没错，依据就是前端路由。所以使用vue的时候需要引入vue-router，使用angular的时候需要引入ui-router。


### hash 模式和 history 模式
###### 浏览器对象（Browser）
Browser包含window对象，Navigator对象，Screen对象，History对象，Location对象等，其中window对象表示浏览器打开的窗口；Navigator对象包含浏览器的相关信息，其常用的属性有navigator.userAgent获取浏览器内核等信息；Screen对象包含客户端显示屏幕的信息，如screen.height或screen.width获取宽高；history对象包含访问过的URL，是window对象的一部分，有三个方法，back(),forward(),go(),调用这三个方法，浏览器会加载对应页面；location对象包含当前URL有关的信息，是window对象的一部分，常用的属性有location.hash返回URL的hash值，location.port返回当前使用的端口号。详细信息可查阅[文档](https://www.runoob.com/jsref/obj-window.html)

>hash和history只是浏览器的两个特性

###### hash
hash即‘#/xxx’，是浏览器用来做页面定位的，例如a标签的描点功能，使用 window.location.hash 可以读取当前页面的hash值,也可以写入，hashchange事件可以响应hash的的改变，有两个特点：
	1、改变hash值，浏览器不会重载页面，但会在历史访问里增加一条纪录
	2、刷新重载页面时，hash值不会传给服务器端，浏览器访问的URL是http://wenyan.51easymaster.com/#/WYHome， 实际上向服务器发起请求的URL是http://wenyan.51easymaster.com/

vue利用了浏览器的hash特性实现了前端路由（vue-router hash模式）。
当用户访问http://wenyan.51easymaster.com 时，服务器返回一个html文件（以及打包好的js文件和css文件），当访问 http://wenyan.51easymaster.com/#/WYHome 和http://wenyan.51easymaster.com/#/answerDetail 时浏览器就会根据hash值（#/WYHome 和 #/answerDetail）请求对应的JSON数据渲染对应页面。整个项目只有一个html文件，vue-router内部监听了hashchange事件响应hash值改变，根据hash值的不同渲染不同的视图。



###### history
HTML5 History Interface 中history对象新增了两个方法 pushState() 和 popState()。
这两个方法应用于浏览器的历史记录栈，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会重载页面。

vue-router的 history 模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。使用history模式，URL是http://wenyan.51easymaster.com/user/id 这个样子的，使用history模式要注意的是前端的 URL 必须和实际向后端发起请求的 URL 一致而且还需要后台配置支持，如访问http://wenyan.51easymaster.com/user/id 时如果后端缺少对 /user/id 的路由处理，将返回 404 错误。官方给出的方案是你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。具体请参阅[文档](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)

### over 