# Python数据结构：

## Ptyhon 字符串表达形式

* `Python 有一个内置的函数叫 repr`，它能把一个`对象用字符串`的形式表达出来以便辨认，这就是`字符串表示形式`。
* repr 就是通过 `__ repr__这个特殊方法`来得到一个对象的字符串表示形式的。
  * 在使用` % 符号的字符串格式中`，这个函数返回的结果用来代替` %r 所代表的对象`；
  * 使用`str.format 函数所用到的新式字符串格式化语法`

## __ repr __ 和 __ str __ 区别：

* __ str __ 是再`str()函数被使用`，或是再用`print函数打印一个对象`的时候才被调用，并且它返回的字符串对终端用户更友好。
* 实现两个特殊方法中的一个,`__ repr __ 是更好的选择`，因为如果一个对象`没有  __ str__ 函数`，而Python又需要调用它的时候，`解释器会用__ repr __作为替代`。

## 自定义的布尔值

* 默认的情况下，我们自己定义的类的实例总是被认为是`真的`，除非这个类对__ bool __ 或者__ len __函数有自己的实现。
* `bool(x)`的背后是条用了`x.__ bool __ ()`的结果；如果`不存在` __ bool __ 方法，那么bool(x)会尝试调用 `x. __ len__()`。
  * 若返回`0`， 则`bool会返回False`；否则返回`True`。
  * 因为__ bool __函数的返回类型应该是`布尔型`，所以我们通过`bool(abs(self))`把模值变成了布尔值。

## 特殊方法

* 跟`运算符无关`的特殊方法：

  | 类别                  | 方法名                                                       |
  | --------------------- | ------------------------------------------------------------ |
  | 字符串/字节列表示形式 | __ repr __ 、 __ str __ 、__ format __ 、 __ bytes __        |
  | 数值转换              | __ abs __ 、 __ bool __ 、__ complex __ 、 __ int __ 、 __ float __ 、 __ hash __ 、 __ index __ |
  | 集合模拟              | __ len __ 、 __ getitem __ 、 __ setitem __ 、 __ delitem __ 、 __ contains __ |
  | 迭代枚举              | __ iter __ 、 __ reversed __ 、 __ next __                   |
  | 可调用模拟            | __ call __                                                   |
  | 上下文管理            | __ enter__ 、 __ exit __                                     |
  | 实例创建和销毁        | __ new __ 、 __ init__ 、 __ del __                          |
  | 属性管理              | __ getattr __ 、 __ getattribute __ 、 __ setattr __ 、 __ delattr __ 、 __ dir __ |
  | 属性描述符            | __ get __ 、 __ set __ 、 __ delete __                       |
  | 类相关的服务          | __ prepare __  、__ instancecheck __ 、 __ subclasscheck __  |

  

* 跟`运算符相关`的特殊方法：

  | 类别               | 方法名和对应的运算符                                         |
  | ------------------ | ------------------------------------------------------------ |
  | 一元运算符         | __ neg __ (-:减号) 、 __ pos __  (+加号) 、__ abs __ 、 （abs():绝对值） |
  | 比较运算符         | __ lt __ (<) 、 __ le __ (<=) 、 __ eq __ (==)、 __ ne __ (!=) 、 __ gt __ (>) 、__ ge __ (>=) |
  | 算术运算符         | __ add __ (+)、__ sub __ (-)、 __ mul __ (*) 、 __ truediv __ (/) 、__ floordiv __ (//) 、__ mod __ (%) 、__ divmod __ ( 包含商和余数的元组 )、__ pow __ (**或 pow())、 __ round __ round() |
  | 反向算数运算符     | __ radd __ 、__ rsub __ 、__ rmul __ 、__ rtruediv __ 、__ rfloordiv __ 、__ rmod __ 、__ rdivmod __ |
  | 增量赋值算术运算符 | __ iadd __ 、 __ isub __ 、 __ imul __ 、__ itruediv __ 、 __ ifloordiv __ 、 __ imod __ 、__ ipow __ |
  | 位运算符           | __ invert __  (~)、__ lshift __ (<<)、 __ rshift __ (>>) 、__ and __ (&)、 __ or __ (\|)、__ xor __ (^) |
  | 反向位运算符       | __ rlshift __ 、 __ rrshift __ 、 __ rand __ 、 __ rxor __ 、 __ ror __ |
  | 增量赋值位运算符   | __ ilshift __ 、 __ irshift __ 、__ iand __ 、__ ixor __ 、__ ior __ |

* 当交换两个操作数的位置时，就会调用反向运算符(b * a 而不是 a * b)
* 增量赋值运算符则是一种把中缀运算符变成赋值运算符(a = a * b 变成了 a *= b)

## 内置序列类型

* 容器序列：
  * `list、tuple和collections.deque、这些序列能存放不同的类型数据。`
* 扁平序列：
  * `str、bytes、bytearray、memoryview和array.array，这类序列只能容纳一种类型。`
* `容器序列存放的是它们所包含的任意类型的对象的引用，而扁平序列里存放的是值而不是引用。`
* 可变序列：
  * `list、bytearray、array.array、collections.deque和memoryview。`
* 不可变序列：
  * `tuple、str和bytes`

## 列表推导同filter和map的比较：

* filter和map合起来能做的事情，列表推导也可以做。

  ```json
  # 下面是列表推导
  symbols = '$¢£¥€¤'
  beyond_ascii = [ord(item) for item in symbols if ord(item) > 127]
  print(beyond_ascii)
  
  # 用map 和 filter 组合来创建
  beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))
  print(beyond_ascii)
  ```

  * 列表推导的作用只有一个：`生成列表。`
  * 如果想生成`其他`类型的序列，`生成器表达式就派上了用场。`

## 生成器表达式：

* 生成器表达式背后遵守了`迭代器协议`，可以逐个地产出元素，而不是先建立一个完整的列表，然后再把这个列表页传递到某个构造函数里`

* `生成器表达式的语法跟列表推导差不多，只不过把方括号换成圆括号而已。`

  ```json
  colors = ['black', 'white']
  sizes = ['S', 'M', 'L']
  
  t_shirt = ((c, s) for c in colors for s in sizes)
  print(t_shirt)
  # 输出的是：<generator object <genexpr> at 0x0000000003709900>
  ```

  * 避免额外的内存占用。

## 元组

* 元组其实是对数据的记录。

  * 元组中的每个元素都存放了记录中`一个字段的数据`，外加这个`字段的位置`。

  * 正式这个位置信息给数据赋予了意义。

  * 但是如果把元组当作一些字段的集合，那么`数量`和`位置信息`就变得非常重要了。

    ```json
    #  把元组用作记录
    city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014) 
    ```

    ```json
    traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ('ESP', 'XDA205856')]
    for country, _ in traveler_ids:
        print(country)
    	# 输出结果如下：
    	# USA
    	# BRA
    	# ESP
    ```

    * for 循环可以分别提取元组里的元素，也叫作`拆包`。因为元组中第二个元组对我们没有什么用，所以它赋值给`_`占位符。

### 一、元组拆包:

* 元组拆包可以应用到任何可迭代对象上，唯一硬性要求是，被可迭代对象中的`元素数量必须跟接收这些元素的元组空挡数一致`，除非我们用`*`来表示忽略多余的元素，

* 最好辨认的元组拆包形式就是`平行赋值`，把一个可迭代对象里的元素，一并赋值到由对应变量组成的元组中。

  ```json
  lax = (33.9425, -188.4008056)
  latitude, longitude = lax
  print(latitude)
  # 输出结果：33.9425
  print(longitude)
  # 输出结果：-188.4008056
  ```

  * 更优化的写法：

    ```json
    b, a = a, b
    ```

  * 还可以用`*运算符`把一个可迭代对象拆开作为函数的参数：

    ```json
    t = (20, 8)
    a, b = divmod(*t)
    print(a)
    # 输出结果：2
    print(b)
    # 输出结果：4
    ```

* `用*来处理剩下的元素`：

  * `函数用*args来获取不确定数量的参数`

    * 再python3 中，这个概念被扩展到了平行赋值中:

      ```json
      >>> a, b, *rest = range(5)
      >>> a, b, rest
      (0, 1, [2, 3, 4])
      >>> a, b, *rest = range(3)
      >>> a, b, rest
      (0, 1, [2])
      >>> a, b, *rest = range(2)
      >>> a, b, rest
      (0, 1, [])
      ```

  * 再平行赋值中，`*前缀`只能用再一个变量名前面，但是这个变量可以出现再赋值表达式的任意位置：

    ```json
    >>> a, *body, c, d = range(5)
    >>> a, body, c, d
    (0, [1, 2], 3, 4)
    >>> *head, b, c, d = range(5)
    >>> head, b, c, d
    ([0, 1], 2, 3, 4)
    ```

### 二、嵌套元组拆包：

* 元组可以是嵌套式的，例如：(a, b, (c, d))

  ```json
  metro_areas = [
      ('Tokyo', 'JP', 36.23, (35.32132, 32.1231)),
      ('Sao Paulo', 'BR', 19.231, (-23.12, -47.32131))
  ]
  for name, cc, pop, (latitude, longitude) in metro_areas:
      print(name, cc, pop, latitude, longitude)
  # 输出结果如下：
  Tokyo JP 36.23 35.32132 32.1231
  Sao Paulo BR 19.231 -23.12 -47.32131
  ```



### 三、具名元组：

* `collections.nametuple 是一个工厂函数，它可以用来构建一个带字段名的元组和一个有名字的类(对调视程序有很大的帮助)`

* `namedtuple构建的类的实例所消耗的内存跟元组是一样的，因为字段名都被存在对应的类里面。`

* 定义和使用具名元组：

  ```json
  from collections import namedtuple
  City = namedtuple('City', 'name country population coordinates')
  print(City)
  # 输出结果：
  <class '__main__.City'>
  ```

  * 创建一个具名元组需要`两个参数`， 一个是`类名`，另一个是类的各个`字段名字`。字段名可以是由数个`字符串`组成的可迭代对象，或者是由`空格分开`的字段名组成的字符串。

  ```json
  tokyo = City('Tokyo', 'JP', '36.933', (36.933, 312.321))
  print(tokyo)
  ```

  * 存放再对应字段里的数据要以一串`参数`的形式传入到构造函数中。

  ```json
  print(tokyo.coordinates)
  # 输出结果(36.933, 312.321)
  print(tokyo[1])
  # 输出结果JP
  ```

  * 通过`字段名`或者`位置`来获取一个字段的信息。

* 具名元组的`属性和方法`:

  * `_fields属性是一个包含这个类所有字段名称的元组`:

    ```json
    City._fields
    # 输出结果：
    ('name', 'country', 'population', 'coordinates')
    ```

  * 用`_make()通过接收一个可迭代对象来生成这个类的一个实例，它的作用跟City(*delhi_data) 是一样的。`

    ```json
    dehli_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.6123, 77.1239))
    delhi = City._make(dehli_data)
    print(delhi)
    # 输出结果：
    City(name=u'Delhi NCR', country=u'IN', population=21.935, coordinates=LatLong(lat=28.6123, long=77.1239))
    ```

  * 用`_asdict() 把具名元组以collections.OrderedDict`的形式返回：

    ```json
    from collections import namedtuple
    Role = namedtuple('Role', 'ADMIN OPERATOR TUTOR SUPERVISOR SELLER CLASS_HEADER FINANCIAL')
    a = Role(ADMIN=(8388608, 1), OPERATOR=(256, 2), TUTOR=(128, 3),
             SUPERVISOR=(16, 4), SELLER=(8, 5), CLASS_HEADER=(2, 6),
             FINANCIAL=(32, 7))
    c = a._asdict()
    for key, value in c.items():
        print(key, value)
    # 输出结果如下：
    ADMIN (8388608, 1)
    OPERATOR (256, 2)
    TUTOR (128, 3)
    SUPERVISOR (16, 4)
    SELLER (8, 5)
    CLASS_HEADER (2, 6)
    FINANCIAL (32, 7)
    ```

## 切片：

* 再Python中，像`列表(list)、元组(tuple)和字符串(str)这类序列类型都支持切片的操作`。
* a: b: c 这种用法只能`作为索引或者下标`用在 [] 中来返回一个切片对象：slice(a, b, c)。对`seq[start:stop:step] `进行求值的时候，Python 会调用`seq.__ getitem__(slice(start, stop, step))。`

#### 一、对象进行切片：

* s[a: b: c]的形式对s再a和b之间以c为`间隔取值`。

  ```json
  s = 'bicycle'
  print(s[::3])
  # 输出结果：bye
  ```

* c的值也可以为`负`，负值意味着`反向取`。

  ```json
  s = 'bicycle'
  print(s[::-1])
  #输出结果：elcycib
  s = 'bicycle'
  print(s[::-2])
  #输出结果：eccb
  ```

#### 二、多维切片：

* `[] 运算符里还可以使用以逗号分开的多个索引或者是切片`

* [] 运算符的话，对象的特殊方法 __ getitem__ 和 __ setitem__ 需要以元组的形式来接收a[i, j] 中的索引。

* 二维切片：

  ```json
  a[m:n, k:i]
  ```

#### 三、给切片赋值：

* 把切片放在赋值语句的`左边`，或把它作为`del操作的对象`，我们就可以对序列`进行嫁接、切除或者马上修改`。

  ```json
  >>> l = list(range(10))
  >>> l
  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  >>> l[2:5] = [20, 30]
  >>> l
  [0, 1, 20, 30, 5, 6, 7, 8, 9]
  >>> del l[5:7]
  >>> l
  [0, 1, 20, 30, 5, 8, 9]
  >>> l[3::2] = [11, 22]
  >>> l
  [0, 1, 20, 11, 5, 22, 9]
  >>> l[2:5] = 100  # 如果赋值的对象是一个切片，那么赋值的右侧必须是可迭代，一个值也是如此
  Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  TypeError: can only assign an iterable
  >>> l[2:5] = [100]
  >>> l
  [0, 1, 100, 22, 9]
  ```

## 对序列使用 + 和 *：

* `+`号两侧的序列由`相同类型的数据所构成`，再`拼接`过程中，两个被操作的序列不会被修改，会`新建一个包含同样类型数据的序列来作为拼接结果`。

  ```json
  >>> l = [1, 2, 3]
  >>> l * 5
  [1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
  >>> 5 * 'abcd'
  'abcdabcdabcdabcdabcd'
  ```

  * `+ 和 * 号`都遵循这个规律，不修改原有的序列，而是新建一个新的序列。

#### 建立由列表组成的列表：

* 一个包含 3 个列表的列表，嵌套的 3 个列表各自有 3 个元素：

  ```json
  my_list = [['_'] * 3 for item in range(3)]
  print(my_list)
  # 输出结果：[[u'_', u'_', u'_'], [u'_', u'_', u'_'], [u'_', u'_', u'_']]
  my_list[1][2] = 'x'
  print(my_list)
  # 输出结果：[[u'_', u'_', u'_'], [u'_', u'_', u'x'], [u'_', u'_', u'_']]
  ```

  * 实际本质：

    ```json
    my_list = []
    for item in range(3):
        row = ['_'] * 3
        my_list.append(row)
    my_list[1][2] = 'x'
    print(my_list)
    # 输出结果：[[u'_', u'_', u'_'], [u'_', u'_', u'x'], [u'_', u'_', u'_']]
    ```

    

  * 下面要展示一个**错的**演示：

  ```json
  my_list = [['_'] * 3] * 3
  print(my_list)
  # 输出结果：[[u'_', u'_', u'_'], [u'_', u'_', u'_'], [u'_', u'_', u'_']]
  my_list[1][2] = '0'
  print(my_list)
  # 输出结果：[[u'_', u'_', u'0'], [u'_', u'_', u'0'], [u'_', u'_', u'0']]
  ```

  * 外面的列表其实包含3个指向同一个列表的`引用`。

  * 错的本质是：

    ```json
    row = ['_'] * 3
    my_list = []
    for item in range(3):
        my_list.append(row)
    
    my_list[1][2] = 'o'
    print(my_list)
    # 输出结果：[[u'_', u'_', u'o'], [u'_', u'_', u'o'], [u'_', u'_', u'o']]
    ```

  * **每次迭代都新建一个列表，作为row追加到my_list**

    

## 序列的增量和赋值：

* 增量赋值运算符 `+= 和 *=`的表现取决于它们的第一个操作对象。

* `+= 背后的特殊方法是__ iadd__()。如果一个类没有实现这个方法的话，python退一步调用____ add__。`

  ```json
  a += b
  ```

  * 1.a实现了__ iadd__ 方法，就会调用这个方法。就像调用了a.extend(b)一样。
  * 2.如果a没有实现__ iadd__的话， 就变成了`a = a+b`一样了，a + b 的新值赋值给a。

* `不要把可变对象放在元组里`。

* `增量赋值不是一个原子操作`。

## list.sort方法和内置函数sorted：

* `list.sort 方法`会就地`排序列表`，也就是说不会把原列表复制一份。也是这个`方法返回值是None的原因，提示你本方法不会新建一个列表`。

  * **如果一个函数或者方法对对象进行的是就地改动，那它应该返回None，好让调用者知道传入的参数发生了变动，而且并未产生新的对象。**

* `内置函数sorte，它会新建一个列表作为返回值`。这个方法可以`接收任何形式的可迭代对象作为参数`，甚至包括`不可变序列`或`生成器`。而不管sorted接收的是怎样的参数，`它最后都会返回一个列表`。

* **list.sort方法还是sorted函数**，都由两个可选的关键字参数。

  * reverse：
    * 如果被设定为`True`，被排序的序列里的元素会以`降序输出`。这个参数`默认值是False`。
  * key：
    * 一个只有一个参数的函数，这个函数会被用在序列里的每一个元素上。这个参数的默认值是`恒等函数`，也就是默认用元素自己的值来排序。

  ```json
  >>> fruits = ['grape', 'raspberry', 'apple', 'banana']
  >>> sorted(fruits)
  ['apple', 'banana', 'grape', 'raspberry']  # 新建一个按照字母排序的字符串列表
  >>> fruits
  ['grape', 'raspberry', 'apple', 'banana']  # 原列表并没有变化
  >>> sorted(fruits, reverse=True)
  ['raspberry', 'grape', 'banana', 'apple']  # 按照字母降序排序
  >>> sorted(fruits, key=len)
  ['grape', 'apple', 'banana', 'raspberry']  # 按照字符串的长度进行排序
  >>> sorted(fruits, key=len, reverse=True)
  ['raspberry', 'banana', 'grape', 'apple'] # 按照字符串的长度 反转进行排序
  >>> fruits
  ['grape', 'raspberry', 'apple', 'banana']  # 原列表fruits都没有任何变化  
  >>> fruits.sort() # 对原列表排序，返回None会被控制台葫芦
  >>> fruits
  ['apple', 'banana', 'grape', 'raspberry'] 
  ```

  

## 数组：

* `如果我们需要一个只包含数字的列表，那么array.array比list更高效`。

* `数组支持所有跟可变序列有关的操作，包括.pop、.insert、和.extend。`

* 数组还提佣`从文件读取和存入文件`的更快的方法，如`.frombytes和.tofile`

  ```json
  from array import array
  ```

  