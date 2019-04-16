## vue的另一个内置组件keep-alive
keep-alive是vue提供的用来缓存组件的，被keep-alive包裹的组件在离开时不会销毁要离开的组件，而是将其缓存在内存中，避免重新渲染，，需要注意的是keep-alive是vue2.0及以上版本具有的，2.0以下版本不具有该组件。

#### 用这个虚拟组件对我得网站有什么好处?
回想一下你实际开发网站的时候是否又这样的场景，页面上有几个tab页来回切换展示不同的数据，如果没有，那网页导航肯定会有吧？没有tab页切换总有导航切换吧~~，在切换的过程中（特别是tab切换），每切换到一个页面就会完全重新渲染一次，组件里的data,comouted等等要重新计算，包括页面上的数据都要重新请求，如果数据多的话每次切换都能明显的感觉到‘闪白屏’，这样用户体验很不好，而且造成很多不必要的请求，如果平时有考虑过网站性能问题，那你肯定听过‘要想大幅度优化页面，必须从http请求上入手’，这个时候就要用keep-alive了，用了后你会发现只有第一次切换的时候需要请求数据，渲染组件，第二次再切换的时候就直接拿缓存了。

查看效果：
>recommend.vue页面添加钩子函数打印

![](https://user-gold-cdn.xitu.io/2019/4/13/16a149034f068f18?w=615&h=178&f=png&s=38913)

>第一次切换到动态tab页，请求数据

![](https://user-gold-cdn.xitu.io/2019/4/13/16a148ec90ea1b35)

>第二次切换到动态tab页，没有任何请求

![](https://user-gold-cdn.xitu.io/2019/4/13/16a1492bb81b2da5?w=1470&h=764&f=png&s=91603)

>两次切换控制台打印，没有走destroyed函数销毁组件，两次切换只有第一次打印创建组件和挂载,第二次拿的缓存

![](https://user-gold-cdn.xitu.io/2019/4/13/16a149435fb0ac7a?w=1223&h=625&f=png&s=64863)

#### 怎么用？
具体用法请百度吧，很全啊！！:laughing:

#### 缓存究竟在哪里？
在组件里打印this,层层递进的寻找，最终找到，具体路径是：VueComponent>$vnode(虚拟DOM)>parent(这一级找到的是keep-alive)>componentInstance（组件实例）>cache
![](https://user-gold-cdn.xitu.io/2019/4/13/16a14b9691f6924f?w=1265&h=640&f=png&s=72449)

cache是一个Object,每个key对应一个组件的缓存，打开cache下的componentInstance(组件实例)，你会发现和VueComponent > $vnode > componentInstance 一样，他的缓存，当然得和他一样了~

#### 怎么实现的缓存？
简单看一下源码吧
rander函数：
```
keep-alive也是一个组件，也符合组件的基本构造，有prop，mounted，created等，但是主要的就是他的rander函数了，即渲染规则
render () {
	<!-- 在keep-alive组件文件里，this指向当前组件实例，即上文提到的层层递进找到的parent.componentInstance -->

	<!-- 按照this.$slot.default找，发现default是一个数组，存的是一个vnode，这个vnode就是用keep-alive包裹的当前要渲染的组件，如下图所示-->
    const slot = this.$slots.default

    <!--将default[0]赋值给vnode常量  -->
    const vnode: VNode = getFirstComponentChild(slot) 

    <!-- 将vnode.componentOptions赋值给componentOptions -->
    const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions

    <!-- 不必细究---开始 -->
    if (componentOptions) {
      const name: ?string = getComponentName(componentOptions)
      const { include, exclude } = this
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode
      }
      <!-- 不必细究--结束 -->

	<!-- 重点 -->
	<!-- 将this.cache和this.keys(this指向上文已说明)赋值给常量cahce和key -->
      const { cache, keys } = this  

    <!-- 判断vnode.key存在吗，不存在就赋值常量key为componentOptions.Ctor.cid，存在就赋值常量key为vnode.key，拿到的key为要渲染的组件对应的key值，我拿到的是94-->
      const key: ?string = vnode.key == null
        ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
        : vnode.key

    <!--再看cache是个对象，如下图，key值为94对应一个vnode，找到了，说明有缓存  -->
      if (cache[key]) {
      	<!-- 有缓存当然是要拿缓存了，将缓存里的组件实例赋值给即将渲染的组件的实例 -->
        vnode.componentInstance = cache[key].componentInstance
        remove(keys, key)
        keys.push(key)
      } else {
        cache[key] = vnode
        keys.push(key)
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode)
        }
      }

      vnode.data.keepAlive = true
    }
    return vnode || (slot && slot[0])
}
```
>按照路径找到的default：

![](https://user-gold-cdn.xitu.io/2019/4/13/16a155bd8dd80a13?w=1131&h=676&f=png&s=58287)

>按照路径找到的cache

![](https://user-gold-cdn.xitu.io/2019/4/13/16a1566646ddbf46?w=1069&h=496&f=png&s=47550)

#### 几个注意点
大体思路应该理解了，说一下我用keep-alive的时候踩过的几个坑：

- 版本问题，文字开头就提到了，keep-alive是vue2.0以上的版本才具有的，而且vue2.1+的版本才支持有条件的缓存，也就是include和exculed两个属性，用之前请先打开项目的package.json查看vue的版本
- name值的问题，include和exclude属性里写的是组件的name值，注意是组件.vue文件里的name值，export default{}里的组件的name值，并不是路由文件里的name值
- 多级子路由的问题，keep-alive包裹着谁，就会缓存谁，并不会缓存子路由对应的组件，如果你想缓存子路由的组件，那就需要给子路由对应的router-view也包裹上keep-alive。
- keep-alive缓存的组件如果渲染时拿的是缓存，那么，组件data、computed里的属性都不会重新计算，完全是拿的第一次渲染时的值，所以，如果有改变的话可以再activated回调函数里处理