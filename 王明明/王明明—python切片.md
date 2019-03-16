## 切片的基础用法



```
li = [1, 4, 5, 6, 7, 9, 11, 14, 16]

# 以下写法都可以表示整个列表，其中 X >= len(li)
li[0:X] == li[0:] == li[:X] == li[:] 
== li[::] == li[-X:X] == li[-X:]

li[1:5] == [4,5,6,7] # 从1起，取5-1位元素
li[1:5:2] == [4,6] # 从1起，取5-1位元素，按2间隔过滤
li[-1:] == [16] # 取倒数第一个元素
li[-4:-2] == [9, 11] # 从倒数第四起，取-2-(-4)=2位元素
li[:-2] == li[-len(li):-2] 
== [1,4,5,6,7,9,11] # 从头开始，取-2-(-len(li))=7位元素

# 步长为负数时，列表先翻转，再截取
li[::-1] == [16,14,11,9,7,6,5,4,1] # 翻转整个列表
li[::-2] == [16,11,7,5,1] # 翻转整个列表，再按2间隔过滤
li[:-5:-1] == [16,14,11,9] # 翻转整个列表，取-5-(-len(li))=4位元素
li[:-5:-3] == [16,9] # 翻转整个列表，取-5-(-len(li))=4位元素，再按3间隔过滤

# 切片的步长不可以为0
li[::0]  # 报错（ValueError: slice step cannot be zero）  

```

## 切片的高级用法

当取出切片的结果时，它是一个独立对象，因此，可以将其用于赋值操作，也可以用于其它传递值的场景。但是，**切片只是浅拷贝** ，它拷贝的是原列表中元素的引用，所以，当存在变长对象的元素时，新列表将受制于原列表。

若将切片作为独立对象取出，那你会发现它们都是空列表，即 `li[:0]==li[len(li):]==li[6:6]==[]` ，对空列表赋值，并不会破坏原有的元素，只会在特定的索引位置中拼接进新的元素。删除纯占位符时，也不会影响列表中的元素。

```
li = [1, 2, 3, 4]

# 在头部拼接
li[:0] = [0] # [0, 1, 2, 3, 4]
# 在末尾拼接
li[len(li):] = [5,7] # [0, 1, 2, 3, 4, 5, 7]
# 在中部拼接
li[6:6] = [6] # [0, 1, 2, 3, 4, 5, 6, 7]

# 给切片赋值的必须是可迭代对象
li[-1:-1] = 6 # （报错，TypeError: can only assign an iterable）
li[:0] = (9,) #  [9, 0, 1, 2, 3, 4, 5, 6, 7]
li[:0] = range(3) #  [0, 1, 2, 9, 0, 1, 2, 3, 4, 5, 6, 7]
```

## 自定义对象实现切片功能

### 魔术方法：`**getitem**(self, key)`

`__getitem__(self, key)` 方法用于返回参数 key 所对应的值，这个 key 可以是整型数值和切片对象，并且支持负数索引；如果 key 不是以上两种类型，就会抛 TypeError；如果索引越界，会抛 IndexError ；如果定义的是映射类型，当 key 参数不是其对象的键值时，则会抛 KeyError 。

```
class MyDict():
    def __init__(self):
        self.data = {}
    def __len__(self):
        return len(self.data)
    def append(self, item):
        self.data[len(self)] = item
    def __getitem__(self, key):
        if isinstance(key, int):
            return self.data[key]
        if isinstance(key, slice):
            slicedkeys = list(self.data.keys())[key]
            return {k: self.data[k] for k in slicedkeys}
        else:
            raise TypeError

d = MyDict()
d.append("My")
d.append("name")
d.append("is")
d.append("Python")
print(d[2])
print(d[:2])
print(d[-4:-2])
print(d['hi'])
```

## 迭代器切片

Python 的 itertools 模块，用它提供的方法可轻松实现迭代器切片。

```
import itertools

# 例1：简易迭代器
s = iter("123456789")
for x in itertools.islice(s, 2, 6):
    print(x, end = " ")   # 输出：3 4 5 6
for x in itertools.islice(s, 2, 6):
    print(x, end = " ")   # 输出：9

# 例2：斐波那契数列迭代器
class Fib():
    def __init__(self):
        self.a, self.b = 1, 1

    def __iter__(self):
        while True:
            yield self.a
            self.a, self.b = self.b, self.a + self.b
f = iter(Fib())
for x in itertools.islice(f, 2, 6):
    print(x, end = " ")  # 输出：2 3 5 8
for x in itertools.islice(f, 2, 6):
    print(x, end = " ")  # 输出：34 55 89 144
```

其主要用途在于截取大型迭代器（如无限数列、超大文件等等）的片段，实现精准的处理