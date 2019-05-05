
### import指令

[MDN]官方解释（https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import）
> 静态的import 语句用于导入由另一个模块导出的绑定。无论是否声明了 strict mode ，导入的模块都运行在严格模式下。在浏览器中，import 语句只能在声明了 type="module" 的 script 的标签中使用。

解释：
<script>元素的默认行为是将JS文件作为脚本加载，而非作为模块加载，当type属性缺失或包含一个JS内容类型(如"text/javascript")时就会发生这种情况。<script>元素可以执行内联代码或加载src中指定的文件，当type属性的值为"module"时支持加载模块。将type设置为"module"可以让浏览器将所有内联代码或包含在src指定的文件中的代码按照模块而非脚本的方式加载

​
import命令具有提升效果，会提升到整个模块的头部，首先执行引擎处理import 指令是在编译时，即代码运行之前，所以import指令必须放在模块的顶层，不能放在代码中，所以如下代码会报错：

```
if(a===2) {
	import '../a/a.js'
}
```

### import() 
为了像node的require一样实现动态加载，于是出现了import（）函数,与require的功能类似，可以实现动态加载，但是import()函数是异步加载的，require是同步加载的，会造成性能问题。import（）函数返回一个promise对象，import()模块加载成功后，这个模块会作为一个对象，当作then方法的参数，主要适应场景有：条件加载，vue定义异步组件，路由按需加载等。由于返回一个promise，所以可以用.then的来获取到export的值

```
import('./myModule.js')
	.then(myModule => {
	console.log(myModule.default);
});

import()还允许动态生成路径
import(fun())  // 路径有fun()动态返回
.then(...);

还可以同时加载多个模块
Promise.all([
import('./module1.js'),
import('./module2.js'),
import('./module3.js'),
])
.then(([module1, module2, module3]) => {
···
});
```

### 异步组件
vue定义异步组件时，允许将组件定义成一个工程函数，返回一个promise,在需要渲染该组件时再加载，import（）函数刚好返回一个promise

异步组件：
```
Vue.component('head-com', function (resolve, reject) {
	$.get("./head.html").then(function (res) {
	    resolve({
	        template: res
	    })
    }
});
```
使用import（）后的异步组件定义：
```
Vue.component('async-webpack-example',
  // 该 `import` 函数返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```

### 总结：

1、在JavaScript ES6中，export与export default均可用于导出常量、函数、文件、模块等，你可以在其它文件或模块中通过import+(常量 | 函数 | 文件 | 模块)名的方式，将其导入，以便能够对其进行使用，但在一个文件或模块中，export、import可以有多个，export default仅有一个
2、通过export方式导出，在导入时要加{ }，export default则不需要
3、module.exports / exports  :  只有node支持的导出
```
module.exports = {
a  : ' ',
fun:''
}


exports.a = ' '
exports.fun = ' '
```
4、export / export default  :  只有es6支持的导出

```
export:
// 写法一
export var m = 1;
export function fun() {}

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};

export default: 
//写法一：
var name = 1
export default name

//写法二
export default {
a: ' ' ,
str: ' ',
fun: ' '
}
```

5、import : 只有es6支持的导入
```
import   myname (as secondname)   from  ' export '
import  { myname (as secondname) ,  myname2  (as thirdname)  }   from ' export default '
```
6、require:  node和es6都支持的导入
```
var  a = require(' ./filepath ')
```