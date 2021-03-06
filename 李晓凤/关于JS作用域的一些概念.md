### 执行环境、变量对象、作用域、作用域链

对于刚刚接触JS的人来说，可能很难搞清楚这些概念是什么，写代码的时候不会用到，但是平时看书看博客的时候这些概念总是会触不及防的出现在你面前，让人很混乱。那么到底这些概念都是什么？

### 执行环境
> 执行环境定义了变量或函数有权访问的其他数据 (JS高级程序设计)

执行环境，就是一个环境，JS的执行环境有全局环境和函数环境之分，这个环境中限定了这个环境中的变量或函数能访问哪些变量或函数。执行环境就是作用域。全局执行环境是最大的一个执行环境，在Web浏览器中全局执行环境被认为是window对象。而每个函数都有自己的执行环境，也就是函数环境。

### 变量对象
每一个执行环境都有与之关联的变量对象，这个变量对象就和普通的对象一样是Object类型，变量对象中存储的是执行环境中定义的所有的变量和函数。

### 作用域链
代码在执行环境中执行时会创建变量对象的一个作用域链，是为了保证对执行环境有权访问的所有变量和函数的有序访问。也就是说在当前执行环境下要访问一个变量，会按照作用域链来寻找这个变量，首先要寻找的就是当前作用域，如果当前作用域没有这个变量，就到上一层执行环境关联的变量对象中寻找，这样一直延续到全局环境。

```
eg:
var color = 'blue'
function changeColor () {
	if(color === 'blue') {
		color = 'red'
	} else {
		color = 'blue'
	}
}

函数changeColor的作用域链是：changeColor函数关联的变量对象 -> 全局环境的变量对象(也就是函数作用域到全局作用域)，当函数内要访问color变量时，会先从函数内寻找，函数内没有定义color变量，再往外层的变量对象中寻找，最后在全局变量对象中找到color。
```

### 块级作用域
JS没有块级作用域，什么是块级作用域，在其他类C语言中，用'{}'花括号包起来的就是块级作用域，可以在块级作用域内定义只有自己能访问的变量或函数。但是，JS没有块级作用域，如下代码，在'{}'花括号内定义的变量color在外部依然可以访问到。
```
if(true) {
	var color = 'blue'
}
console.log(color)  // blue
```
### let
但是 。。。
自从es6出现以后，引入了let，从此js有了块级作用域

```
if(true){
	let a = 1
	console.log(a)  // 1
}
console.log(a) // undefined

花括号'{}'包起来的就是一个块级作用域，在块级作用域内用let定义的变量a只能在块级作用域内访问，在外边访问会返回undefined
```