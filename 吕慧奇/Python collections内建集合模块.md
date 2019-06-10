# Python collections内建集合模块

* `collections 是 Python内建的一个集合模块，提供了许多有用的集合类。`

## 1、namedtuple:

* `tuple`可以表示不变集合，例如，一个二维坐标就可以表示成：

  ```json
  p = (1, 2)
  ```

* 看到`(1, 2)`很不容易看出来这是一个二维坐标，定义一个`class又小题大做`，这时`namedtuple`就派上用场了：

  ```json
  import collections
  Point = collections.namedtuple('Point', ['x', 'y'])
  p = Point(1, 2)
  print(p.x)
  print(p.y)
  # 输入结果如下：
  1
  2
  ```

* `namedtuple`是一个函数，它用来创建一个自定义的`tuple`对象，并且规定了`tuple`元素的个数，并可以用属性而不是索引来引用`tuple`的某个元素

* 我们可以用`namedtuple`很方便的定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用。

* 验证创建的`Point`对象是`tuple`的一种子类：

  ```json
  >>> isinstance(p, Point)
  True
  >>> isinstance(p, tuple)
  True
  ```

* 如果要用`坐标和半径表示一个圆`，也可以用`namedtuple`定义： 

  ```json
  import collections
  Circle = collections.namedtuple('Circle', ['x', 'y', 'r'])
  c = Circle(1, 2, 3)
  print(c.x)
  print(c.y)
  print(c.r)
  
  print(isinstance(c, Circle))
  print(isinstance(c, tuple))
  # 输出结果如下：
  1
  2
  3
  True
  True
  ```

## 2、deque:

* 使用`list`存储数据时，`按索引访问元素很快`，但是`插入和删除元素就很慢了`，因为**`list是线性存储，数据量大的时候，插入和删除效率很低`**

* `deque是为了高效实现插入和删除操作的双向列表`：

  ```json
  import collections
  q = collections.deque(['a', 'b', 'c'])
  q.append('x')  # 默认是添加尾部
  q.appendleft('y')  # 从左边进行添加
  print(q)
  # 输出结果如下：
  deque([u'y', u'a', u'b', u'c', u'x'])
  ```

* `deque`除了实现list的`append()`和`pop()`外，还支持`appendleft()和popleft()`，这样就可以非常高效地往头部添加或者删除元素。

## 3、defaultdict：

* 使用`dict`时，如果引用的key不存在，就会抛出`KeyError`。**`如果希望key不存在时，返回一个默认值，就可以用defaultdict`**

  ```json
  import collections
  dd = collections.defaultdict(lambda: None)
  dd['key1'] = 'abc'
  print(dd['key1'])  # key1存在
  # abc
  print(dd['key2'])  # key2 不存在返回默认值
  # None
  ```

* **注意：默认值是调用函数时返回的，而函数在创建`defaultdict`对象时传入。**

* 除了在key不存在时返回默认值，`defaultdict`的其他行为跟`dict`是完全一样的

## 4、OrderedDict:

* 使用`dict`时，key是无序的。在对`dict`做迭代时，我们无法确定Key顺序。

* 如果要保持Key的顺序，可以用`OrderedDict`:

  ```json
  import collections
  
  my_dict = dict([('a', 1), ('b', 2), ('c', 3)])
  print(my_dict)  # dict的key是无序的
  
  od = collections.OrderedDict([('a', 1), ('b', 2), ('c', 3)])
  print(od)  # OrderedDict的key 是有序的
  
  # 输出结果如下：
  {u'a': 1, u'c': 3, u'b': 2}
  OrderedDict([(u'a', 1), (u'b', 2), (u'c', 3)])
  ```

* 注意：`orderedDict`的key会按照插入的顺序排列，不是key本身排序。

  ```json
  import collections
  op = collections.OrderedDict()
  op['x'] = 1
  op['y'] = 2
  op['z'] = 3
  print(op)
  # 输出结果如下：
  OrderedDict([(u'x', 1), (u'y', 2), (u'z', 3)])  # 按照插入的key顺序返回
  
  my_dict = {}
  my_dict['x'] = 1
  my_dict['y'] = 2
  my_dict['z'] = 3
  print(my_dict)
  # 输出结果如下：
  {u'y': 2, u'x': 1, u'z': 3}
  ```

## 5、Counter:

* `counter`是一个简单的计数器，例如，统计字符出现的个数：

  ```json
  import collections
  
  c = collections.Counter()
  for ch in 'character':
      c[ch] = c[ch] + 1
      
  print(c)
  # 输出结果如下：
  Counter({u'a': 2, u'c': 2, u'r': 2, u'e': 1, u'h': 1, u't': 1})
  ```

* `Counter`实际上也是`dict`的一个子类，上面的结果可以看出，字符`a`,`c`,`r`出现了2次，其他的各出现1次。