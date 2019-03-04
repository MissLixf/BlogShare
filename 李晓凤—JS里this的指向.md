## 普通函数和箭头函数里this指向总结
* this指向上下文
* 只有script脚本函数有上下文
* 普通函数的this在调用时确定，指向调用该函数时的对象，在非严格模式下如果没有直接调用或没有显式调用，则会指向window，在严格模式下如果没有直接调用或显示调用，会指向undefined 
* 箭头函数里的this不绑定值，也可以理解为箭头函数里也没有上下文，所以箭头函数里的this指向离他最近一层的上下文

***
### 普通函数：
1. 
```
var a=11
function test1(){
  this.a=22;
  let b=function(){
  console.log(this.a);
  };
  b();
}
var x=new test1();
```
> 输出11，因为没有显式调用函数b,所以函数b里的this指向window

2.
```
var obj = {
  name: 'latency',
  sayName: function () {
      console.log("name:", this.name);
  }
}
obj.sayName()
```
>  输出latency ，obj显式的调用了函数，所以函数里的this指向obj

3.
```
window.val = 1;
var obj = {
  val: 2,
  dbl: function () {
    this.val *= 2; 
    val *= 2;    // ------------------------[3]
    console.log(val);
    console.log(this.val);
  }
}
obj.dbl();  // 输出 2，4   ------------------[1]
var func = obj.dbl;
func(); // 输出8, 8       -------------------[2]
```
>代码[1]处显示调用函数db1,函数内的this指向obj, [3]处的val访问的是window.val
代码[2]处相当于window.func(),函数里的this指向window，[3]处的val也访问的是window.val,经过两次`*2`运算window.val = 8



### 箭头函数：

1.
```
var obj = {
  name: 'latency',
  sayName: () => {
    console.log("name:", this.name);
  }
}
obj.sayName()
```
> 输出undefined，箭头函数里的this没有绑定上下文，找离他最近的一层上下文（非函数没有上下文），即直到找到window，window没有name属性，输出undefined

2.
```
var obj = {
  name: 'latency',
  sayName: function () {
    return () => {
      console.log("name:", this.name);
    }
  }
}
obj.sayName()
```
> 输出latency，寻找离他最近的上下文，function里是有上下文的，function是由obj显式调用，所以function里的this指向obj，所以箭头函数里的this指向obj，输出latency
