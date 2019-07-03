# Python 高阶函数的使用(map、reduce、filter、zip)

## 一、什么是高阶函数？

`一个函数可以作为参数传给另外一个函数，或者一个函数的返回值为另外一个函数（若返回值为该函数本身，则为递归），满足其一则为高阶函数。 `

## 二、map的使用：

根据提供的函数对指定序列做`映射，并返回映射后的序列`

```json
map(function, iterable, ...)
```

参数：

* `function`：函数，序列中的每个元素需要指定的操作，可以是`匿名函数`
* `iterable`：一个或多个序列

返回值：

* `python 2中返回列表，python 3中返回map类`

正常使用：

```json
def add(x):
    """加1函数"""
    return x + 1

result = map(add, [1, 2, 3, 4, 5])
print result

# 如果是Python3
print list(result)

# 输出结果： [2, 3, 4, 5, 6]
```

使用`匿名函数`：

```json
result = map(lambda x: x + 1, [1, 2, 3, 4, 5])
print result

# 如果使用Python3
print list(result)

# 输出结果：[2, 3, 4, 5, 6] 
```

使用`内置函数`：

```json
result = map(str, [1, 2, 3, 4, 5])
print result

# 如果使用Python3
print list(result)

# 输出结果：['1', '2', '3', '4', '5']
```

使用`多个序列`：

```json
result = map(lambda x, y: x + y, [1, 2, 3], [1, 2, 3])
print result

# 如果使用Python3
print list(result)

# 输出结果：[2, 4, 6]

# 注意：如果俩个序列中值不等，会报错：
result = map(lambda x, y: x + y, [1, 2, 3], [1, 2, 3, 4, 5])
print result
# 报错信息如下：
Traceback (most recent call last):
  File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 2431, in <module>
    result = map(lambda x, y: x + y, [1, 2, 3], [1, 2, 3, 4, 5])
  File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 2431, in <lambda>
    result = map(lambda x, y: x + y, [1, 2, 3], [1, 2, 3, 4, 5])
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

类似`zip`函数操作：

```json
result = map(None, [1, 2, 3, 4, 5], [1, 2, 3, 4])
print result
# 输出结果：[(1, 1), (2, 2), (3, 3), (4, 4), (5, None)]
```

## 三、reduce的使用：

函数会对参数序列中元素`进行累积`

```json
reduce(function, iterable[, initializer])
```

**注意：在Python3中，reduce函数已经被从全局名字空间里移除，使用前需要导入functools模块来调用reduce函数**

```json
from functools import reduce
```

参数：

* `function`：函数，序列中的每个元素需要执行的操作，可以是`匿名函数`
* `iterable`：需要执行操作的序列
* `initializer`：可选，初始参数

返回值：

* 返回函数的计算结果

正常使用：

```json
def add(x, y):
    """求2个参数相加"""
    return x + y


resutl = reduce(add, [1, 2, 3, 4, 5])  # 相当于 1 + 2 + 3 + 4 + 5
print resutl

# 输出结果：15
```

添加初始参数的使用：

```json
def add(x, y):
    """求2个参数相加"""
    return x + y


result = reduce(add, [1, 2, 3, 4, 5], 2)  # 相当于  2 + 1 + 2 + 3 + 4 + 5   
print result

# 输出结果：17
```

使用`匿名函数`：

```json
result = reduce(lambda x, y: x + y, [1, 2, 3, 4, 5]) # 相当于  1 + 2 + 3 + 4 + 5
print result
# 输出结果：15

result = reduce(lambda x, y: x + y, [1, 2, 3, 4, 5], 1)  # 相当于  1 + 1 + 2 + 3 + 4 + 5
print result
# 输出结果：16
```

使用`字符串`，进行拼接：

```json
result = reduce(lambda x, y: x + y, ['1', '2', '3', '4'], 'str类型：')
print result

# 输出结果：str类型：1234
# 可以看出, 设置初始参数, 会先处理初始参数
```

## 三、filter的使用：

用于过滤序列，过滤掉不符合条件的元素，返回符合条件元组组成的`新列表`

```json
filter(function, iterable)
```

参数：

* `function`：过滤操作执行的函数
* `iterable`：需要过滤的序列
* 序列中每个元素作为参数进行传递给函数进行判断，返回`True或False`，最后将返回`True`的元素放到新列表中

返回值：

* `python 2 返回的是过滤后的列表，python 3 返回的是一个filter类`

正常使用：

```json
def check_func(x):
    """校验参数是否是奇数或偶数：
        1.奇数返回True
        2.偶数返回False
    """
    if x % 2 == 1:
        return True
    else:
        return False

result = filter(check_func, [1, 2, 3, 4, 5])  # 相当于 [True, False, True, False, True]
print result

# 如果使用Python3
print list(result)

# 输出结果：[1, 3, 5]
```

使用`匿名函数`：

```json
result = filter(lambda x: True if x % 2 == 1 else False, [1, 2, 3, 4, 5])
print result

# 如果使用Python3
print list(result)

# 可以简写为：
result = filter(lambda x: x % 2 == 1, [1, 2, 3, 4, 5])
print result

# 如果使用Python3
print list(result)

# 输出结果：[1, 3, 5]
```

## 四、zip的使用：

用于将可迭代的对象作为参数，将这些参数元素打包成一个个`元组`，然后返回`这些元组组成的对象，优点就是可以节约内存空间`

```json
zip([iterable, ...])
```

注意：

1.如果各个迭代器的`元素个数不一致`，返回列表长度与最短的对象相同

2.函数有一个参数，接受一个或多个序列

3.函数可以使用`*`号操作符，将`元组解压为列表`，返回二维矩阵式 

返回值：

* `python 2 中返回一个列表，python 3 中返回一个对象`

简单使用：

```json
int_list = [1, 2, 3, 4, 5]
str_list = ['a', 'b', 'c', 'd', 'e']

my_zip = zip(int_list, str_list)
print my_zip

# 如果使用Python3
print list(result)

# 输出结果: [(1, 'a'), (2, 'b'), (3, 'c'), (4, 'd'), (5, 'e')]
```

两个列表`元素个数不同`时的使用：

```json
int_list = [1, 2, 3, 4, 5, 6]
str_list = ['a', 'b', 'c', 'd', 'e']

my_zip = zip(int_list, str_list)
print my_zip

# 如果使用Python3
print list(result)

# 输出结果：[(1, 'a'), (2, 'b'), (3, 'c'), (4, 'd'), (5, 'e')]
# 可以看出，元素的个数与最短的列表一致, int_list列表中的 6 没有对应str_list 列表中的值
```

使用`*`操作符，将元组解压为列表：

```json
int_list = [1, 2, 3, 4, 5]
str_list = ['a', 'b', 'c', 'd', 'e']

result = zip(*zip(int_list, str_list))
print result

# 输出结果：[(1, 2, 3, 4, 5), ('a', 'b', 'c', 'd', 'e')]
```

