> 场景1: 
A组件引用了B组件，B组件引用了C组件，此时A组件和C组件就是祖孙级的关系，A和B或者B和C都是父子级的关系。父子级的组件的传参用prop和$emit,甚至用更直接的$parnet，都可以搞定。但是A和C之间的传参怎么办呢？ 也是有很多方法滴！可以依据层级关系，；可以用vuex;可以用EventBus。

- 依据层级关系传参： A和B是父子组件，A先传给B，B再传给C。但是导致代码太过繁琐，如果B组件中要定义一堆props和methods
- 用vuex： 不得不说vuex简直太方便了，但是如果你的项目很小，整个项目就一个地方用到了祖孙级的传参，那引入vuex就显得很不划算了
- EventBus： 本人不怎么用，具体没研究过~

接下来介绍一种利用vue2.4.0新增的特性实现祖孙传参的方法：

### inheritAttrs  [文档](https://cn.vuejs.org/v2/api/#inheritAttrs)

![](https://user-gold-cdn.xitu.io/2019/9/5/16cff65a20661495?w=1020&h=565&f=png&s=71787)

如果是第一次看到这个属性，看文档的介绍就有些不知所云，直接用代码尝试一下：

子组件代码child.vue:子组件中props里定义了name，name是父组件要传的
```
<template>
  <div class="childContent">
    my name is {{name}}, 测试inheritAttrs属性
  </div>
</template>
<script>
  export default{
    data () {
      return {}
    },
    props: ['name']
  }
</script>

```


父组件代码index.vue:父组件在调用child的时候传了name，还传了一些其他的内容，age，university，quantity。
```
<template>
  <div class="fatherContent">
    <child
      :name="myname"
      age="20"
      university="山西大学"
      :quantity="num"
    ></child>
  </div>
</template>
<script>
  import child from './child.vue'
  export default{
    components: { child },
    data () {
      return {
        myname: '南风NY',
        num: 0
      }
    }
  }
</script>
```


浏览器渲染结果：
![](https://user-gold-cdn.xitu.io/2019/9/5/16cff73b37fbc134?w=1296&h=94&f=png&s=19990)

##### 可以看到子组件不要的属性age，university，quantity被作用在了子组件的根元素上，也就是class为childContent的div上，这就是文档中所说的 “默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。” 



* * *


##### inheritAttrs默认为true,在子组件中设置inheritAttrs为false

子组件child.vue:
```
<template>
  <div class="childContent">
    my name is {{name}}, 测试inheritAttrs属性
  </div>
</template>
<script>
  export default{
    inheritAttrs: false, // 默认为true， 现在设为false
    data () {
      return {}
    },
    props: ['name']
  }
</script>
```

浏览器渲染结果：
![](https://user-gold-cdn.xitu.io/2019/9/5/16cff7f41e1710a2?w=905&h=100&f=png&s=17112)
一切正常，父组件多传的那些属性age，university，quantity，在子组件里不体现

* * *
接下来再分析一下文档中的最后一句

> 通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。

结合上下文解释：虽然inheritAttrs设置为false，不会将父组件传的多余的属性作用到子组件上，但是在子组件中通过$attrs可以访问这些属性

确实是，在子组件中我打印了$attrs, 包含除了name（props包含）外的其他属性
![](https://user-gold-cdn.xitu.io/2019/9/5/16cff96834c7658f)

### $attrs  [文档](https://cn.vuejs.org/v2/api/#vm-attrs)
>  包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

##### 再一次不知所云~~ 吗？

直白的解释： $attrs是用在子组件中的一个属性，是一个对象，包含子组件中未定义为prop但是父组件传了的属性，如果子组件未定义任何prop，父组件传的所有除了class和style的属性都包含在$attrs里；并且可以通过 v-bind="$attrs" 传入内部组件。

代码如下：

父组件index.vue：引用子组件child，传入name，age，university，quantity
```
<template>
  <div class="fatherContent">
    <child
      :name="myname"
      age="20"
      university="山西大学"
      :quantity="num"
    ></child>
  </div>
</template>
<script>
  import child from './child.vue'
  export default{
    components: { child },
    data () {
      return {
        myname: '南风NY',
        num: 0
      }
    }
  }
</script>
```

子组件child.vue: 无需定义大量props，用v-bind绑定$attrs到grandson组件上
```
<template>
  <div class="childContent">
    测试inheritAttrs和绑定$attrs

    <grandson v-bind="$attrs"></grandson>
  </div>
</template>
<script>
  import grandson from './grandson.vue'
  export default{
    inheritAttrs: false, // 默认为true， 现在设为false
    components: { grandson },
    created () {
      console.log('$attrs:', this.$attrs)
    }
  }
</script>
```

孙组件grandson.vue:定义props，直接用这些属性
```
<template>
  <div>
    my name is {{name}}，
    I am {{age}},
    university is {{university}}
    <el-button type="primary">点赞 + {{quantity}}</el-button>
  </div>
</template>
<script>
  export default{
    props: ['name', 'age', 'university','quantity'],
    methods: {}
  }
</script>
```

浏览器渲染结果：

![](https://user-gold-cdn.xitu.io/2019/9/5/16d0018bed253f2b?w=436&h=230&f=png&s=9342)

这样就实现了父组件向孙组件传参，是不是很简单~


##### 那么问题来了，grandson组件如何向父组件传参呢？ grandson组件如何改变父组件传过来的值呢？
莫慌！官方文档很人性化的提供了vm.$listeners，也是2.4.0新增的

### $listeners  [文档](https://cn.vuejs.org/v2/api/#vm-listeners)

> 包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。

直白的解释：也是用在子组件中的一个属性，是一个对象，包含子组件中未emit但是父组件监听了的不含 .native 修饰器的事件监听器。
好吧！可能不是很直白！直接用代码解释吧：

父组件代码index.vue： 监听了addQuantity
```
<template>
  <div class="fatherContent">
    <child
      :name="myname"
      age="20"
      university="山西大学"
      :quantity="num"
      @addQuantity="addQuantity"
    ></child>
  </div>
</template>
<script>
  import child from './child.vue'
  export default{
    components: { child },
    data () {
      return {
        myname: '南风NY',
        num: 0
      }
    },
    methods: {
      addQuantity () {
        this.num++
      }
    }
  }
</script>
```

子组件代码child.vue: 打印$listeners

```
<template>
  <div class="childContent">
    测试inheritAttrs和绑定$attrs

    <grandson v-bind="$attrs"></grandson>
  </div>
</template>
<script>
  import grandson from './grandson.vue'
  export default{
    inheritAttrs: false, // 默认为true， 现在设为false
    components: { grandson },
    created () {
      console.log('$attrs:', this.$attrs)
      console.log('$listeners', this.$listeners)
    }
  }
</script>
```
打印结果： 
![](https://user-gold-cdn.xitu.io/2019/9/5/16d00293edef2363?w=652&h=202&f=png&s=21607)

> 通过 v-on="$listeners" 传入内部组件

父组件代码index.vue: 监听addQuantity

```
<template>
  <div class="fatherContent">
    <child
      :name="myname"
      age="20"
      university="山西大学"
      :quantity="quantity"
      @addQuantity="addQuantity"
    ></child>
  </div>
</template>
<script>
  import child from './child.vue'
  export default{
    components: { child },
    data () {
      return {
        myname: '南风NY',
        quantity: 0
      }
    },
    methods: {
      addQuantity (haslike) {
        this.quantity += haslike
      }
    }
  }
</script>
```
子组件child.vue: 通过 v-on="$listeners" 传入到孙组件
```
<template>
  <div class="childContent">
    测试inheritAttrs和绑定$attrs

    <grandson v-bind="$attrs" v-on="$listeners"></grandson>
  </div>
</template>
<script>
  import grandson from './grandson.vue'
  export default{
    inheritAttrs: false, // 默认为true， 现在设为false
    components: { grandson },
    created () {
      console.log('$attrs:', this.$attrs)
      console.log('$listeners', this.$listeners)
    }
  }
</script>
```
孙组件grandson.vue: 触发父组件监听的addQuantity，并传入自己的值haslike

```
<template>
  <div>
    my name is {{name}}，
    I am {{age}},
    university is {{university}} <br/>
    我的掘力值 {{quantity}} <br/>
    <el-button type="primary" @click="add">点赞</el-button>
  </div>
</template>
<script>
  export default{
    props: ['name', 'age', 'university','quantity'],
    data () {
      return {
        haslike: 10,
      }
    },
    methods: {
      add () {
        this.$emit('addQuantity', this.haslike)
      }
    }
  }
</script>
```

渲染效果： 点赞一次掘力值加10
![](https://user-gold-cdn.xitu.io/2019/9/5/16d0035d07009b37?w=450&h=305&f=png&s=11714)



—————————————————分割线——————————————————————


祖孙组件传参完美实现啦！