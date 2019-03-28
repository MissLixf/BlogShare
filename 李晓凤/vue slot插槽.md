## vue的内置组件slot
vue有很多内置的组件，有component，transition, transition-group, keep-alive, slot；其中slot(插槽)官方文档的解释为内容分发的出口，官方解释很抽象，我们不如直接来根据名字插槽来理解，即留出一个位置，你可以插入任何你想显示的东西例如文本或者hml标签

#### 为什么要用slot？
vue最大的特性就是组件化，在实际项目开发中都会封装公用组件，增强代码复用性。而公用组件就是要可以在多处引用，这就要求网站多处的渲染解构或效果要完全一样，才能把一样的这部分拿出来封装成一个公用组件多处引用，但实际项目开发中设计出来的网站很少有多处效果完全一样的情况，往往都是大体效果类似，并不完全一样，多少会有一些差别。这时候就会用到组件的slot了，封装公用组件的时候把不一样的地方作为一个插槽留出来，在引用组件的时候插入什么就显示什么。这样就实现了组件的高度复用。

#### 使用场景
假如我现在要画两幅画，第一幅是一个篮子里装着苹果，第二幅是一个篮子里装着橘子，如果两幅画的篮子里装着一样的，我只需要画一幅，然后复制粘贴就好了。slot的解决办法：画一个篮子把苹果或者橘子的空位留出来，复制粘贴出来两个篮子，当我要用第一幅画的时候，给篮子里画上苹果，用第二幅画的时候给篮子里画上橘子。

例如网站里，有很多像如下这样的tab标签

![](https://user-gold-cdn.xitu.io/2019/3/27/169be8b3b9558081?w=719&h=75&f=png&s=1989)


![](https://user-gold-cdn.xitu.io/2019/3/27/169be8fe9c9e98d0?w=702&h=69&f=png&s=4102)

这样定义公用组件myTab.vue：
```
<div>
	<div class="mytab">
	  <span v-for="item in showItems" :key="item.name">{{item.label}}</span>  
	  <slot></slot>   // 插槽，给苹果或橘子留出来的位置
	</div>
</div>
```

第一个地方调用：
```
import myTab from '@/components/myTab/myTab.vue'  //导入组件
... //此处省略其他语句
 components: {
      myTab  //注册组件
 },
...
//调用
<my-tab :showItems="showItems">
	<i class="iconfont">&#xe63a;</i>  // 插入筛选图标（画苹果）
</my-tab>

最终渲染结果：
<div>
	<div class="mytab">
	  <span v-for="item in showItems" :key="item.name">{{item.label}}</span>  
	  <i class="iconfont">&#xe63a;</i> 
	</div>
</div>

```

第二个地方调用：
```
import myTab from '@/components/myTab/myTab.vue'  //导入组件
...
 components: {
      myTab  //注册组件
 },
...
//调用
<my-tab :showItems="showItems">
	<span class="slotHandle"> // 插入退出查看按钮（画橘子）
	  退出查看
	  <i class="iconfont">&#xe61a;</i>
	</span>
</my-tab>

最终渲染结果：
<div>
	<div class="mytab">
	  <span v-for="item in showItems" :key="item.name">{{item.label}}</span>  
	  <span class="slotHandle">
	  退出查看
	  <i class="iconfont">&#xe61a;</i>
	</span> 
	</div>
</div>

```
当然了，用v-if通过prop传参也完全可以实现这个效果：
定义公用组件myTab.vue：
```
<div>
	<div class="mytab">
	  	<span v-for="item in showItems" :key="item.name">{{item.label}}</span>  
		<i class="iconfont"  v-if="type === '苹果'">&#xe63a;</i>  // 插入筛选图标（画苹果）

		<span class="slotHandle" v-if="type === '橘子'"> // 插入退出查看按钮（画橘子）
		  退出查看
		  <i class="iconfont">&#xe61a;</i>
		</span>
	</div>
</div>

...
export default{
	prop: ['type']
}

```
但是如果页面上有一百类似的地方要调用这个组件，定义组件的时候就要写一百个v-if来判断到底该显示什么，这样组件就显得臃肿不直观


### slot的name属性
有时候在一个组件里我们需要用到多个插槽，这时候就要用到slot的name属性

```
<div class="container">
  <header>
  ... //其他布局
    <slot name="header"></slot>
  ... //其他布局
  </header>
  <main>
  ... //其他布局
    <slot name='main'></slot>
  ... //其他布局
  </main>
  <footer>
  ... //其他布局
    <slot name="footer"></slot>
  ... //其他布局
  </footer>
</div>

```
调用时使用template标签以及v-slot来匹配slot的name（v-slot是vue2.6及以上版本支持的语法，vue2.6以下版本请查看[文档](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)）：
```
<container>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  // 不是必须写的，没有要插入的内容就不写

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</container>
```

### slot访问子组件的变量
slot是子组件留出来的插槽，插入的内容是由父组件定义的，也相当于插入的是父组件的标签，这些要被插入的标签当然可以随意的访问父组件的data属性了

父组件：father.vue
```
<my-tab :showItems="showItems">
	// 插入的内容开始
	<span class="slotHandle">
	  {{btnText}}   // 可以渲染出 退出查看 按钮
	  <i class="iconfont">&#xe61a;</i>
	</span>
	//插入的内容结束
</my-tab>
...
import myTab from '@/components/myTab/myTab.vue'  //导入组件
...
 components: {
      myTab  //注册组件
 },
 data () {
 	return {
 		btnText: '退出查看'
 	}
 }
...
```

如果要插入的标签里需要访问子组件的变量（以下是vue2.6及以上版本支持的语法，vue2.6以下版本请查看[文档](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)）

定义包含slot的子组件myTab.vue：
```
<div>
	<div class="mytab">
	  <span v-for="item in showItems" :key="item.name">{{item.label}}</span>  
	  <slot :sonCount='count'></slot>   // 在子组件中可以随意在slot上添加属性（sonCount）
	</div>
</div>

...
data () {
	return {
		count: 1
	}
}
```
父组件调用father.vue：

``` 
<my-tab :showItems="showItems" v-slot="myTabSlot"> ，// 注意时=,相当于： v-slot:default="myTabSlot"
	// 插入的内容开始
	{{myTabSlot.sonCount}}   // 访问到了tab.vue里的count:1
	//插入的内容结束
</my-tab>
...
import myTab from '@/components/myTab/myTab.vue'  //导入组件
...
 components: {
      myTab  //注册组件
 }
...
```
还记得v-slot是干嘛的吗？子组件有多个slot时用来匹配slot的name的，也就是v-slot的值是slot的name属性，这样就好理解了，我调用子组件的时候给子组件里的slot起了个名字（myTabSlot），我给子组件中的slot上添加了个属性(sonCount)，绑定了count，用myTabSlot.sonCount就可以访问到count变量了，感觉和ref很类似~

> 这里有要注意的点：
> 官方说法: 当只有默认插槽时，组件的标签才可以被当作插槽的模板来使用，这样我们就可以把 v-slot 直接用在组件上。
> 翻译解说： 当子组件定义slot时只有一个slot，且没有给这一个slot添加name属性，这个slot插槽就是默认插槽（其实默认插槽有一个默认的name为default），调用子组件时，子组件标签（my-tab）可以代替template标签，v-slot可以用在（my-tab）上，否则v-slot只能用在template标签上


### 语法糖：
跟 v-on 和 v-bind 一样，v-slot 也有缩写，即把参数之前的所有内容 (v-slot:) 替换为字符 #，注意是v-slot：（有冒号）

例如上边代码：v-slot:default="myTabSlot"  可以写成 #default="myTabSlot",#后跟的是slot的name，如果是默认插槽，name为default，不能省略



----------------------------------------------------------完结--------------------------------------------------------
