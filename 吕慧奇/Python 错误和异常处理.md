# Python 错误和异常处理

## 一、错误和异常的概念：

* **错误**是无法通过其他代码进行处理问题，比如`语法错误`或`逻辑错误`。

  * 语法错误：是单词或格式等写错，只能根据系统提示去修改相应的代码。

  * 逻辑错误：是代码实现功能的逻辑有问题，系统不会报错，同样根据系统提示去修改相应代码。

    ```json
    # 输入代码
    while True
        print 'Hello world'
    # 输出结果
    File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 1380 
        while True
                 ^
    SyntaxError: invalid syntax
    ```

    语法分析器指出错误行(line 1380)，并且检测错误的位置前面显示一个'^'箭头符号，`错误是由箭头前面的标记引起的`。

* **异常**是程序执行过程中，出现未知的问题，`这里语法和逻辑都是正确的`。

  * 语句或表达式在语法上是正确的，当执行时可能会引发错误。`运行期检测到的错误称为异常，并且程序不会无条件的崩溃`。

    ```json
    # 输入代码
    print 1 / 0
    # 输出结果
    Traceback (most recent call last):
      File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 1385, in <module>
        print 1 / 0
    ZeroDivisionError: integer division or modulo by zero
    
    # 输入代码
    print 6 + '6'
    # 输出结果
    Traceback (most recent call last):
      File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 1387, in <module>
        print 6 + '6'
    TypeError: unsupported operand type(s) for +: 'int' and 'str'
    
    # 输入代码
    print x == 'abc'
    # 输出结果
    Traceback (most recent call last):
      File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 1389, in <module>
        print x == 'abc'
    NameError: name 'x' is not defined
    ```

    `错误信息的最后一行指出发生了什么错误`。

    异常也有不同的类型，异常类型做为信息的一部分显示出来，ZeroDivisionError：零除错误、TypeError：类型错误、NameError：命名错误，打印错误信息时，异常的类型作为异常的内置名显示。

## 二、所有标准异常类：

|         异常名称          |                        描述                        |
| :-----------------------: | :------------------------------------------------: |
|       BaseException       |                   所有异常的基类                   |
|        SystemExit         |                   解释器请求退出                   |
|     KeyboardInterrupt     |             用户中断执行(通常是输入^C)             |
|         Exception         |                   常规错误的基类                   |
|       StopIteration       |                 迭代器没有更多的值                 |
|       GeneratorExit       |        生成器(generator)发生异常来通知退出         |
|        SystemExit         |               Python 解释器请求退出                |
|       StandardError       |              所有的内建标准异常的基类              |
|      ArithmeticError      |               所有数值计算错误的基类               |
|    FloatingPointError     |                    浮点计算错误                    |
|       OverflowError       |                数值运算超出最大限制                |
|     ZeroDivisionError     |            除(或取模)零 (所有数据类型)             |
|      AssertionError       |                    断言语句失败                    |
|      AttributeError       |                  对象没有这个属性                  |
|         EOFError          |             没有内建输入,到达EOF 标记              |
|     EnvironmentError      |                 操作系统错误的基类                 |
|          IOError          |                 输入/输出操作失败                  |
|          OSError          |                    操作系统错误                    |
|       WindowsError        |                    系统调用失败                    |
|        ImportError        |                 导入模块/对象失败                  |
|        LookupError        |                 无效数据查询的基类                 |
|        IndexError         |            序列中没有没有此索引(index)             |
|         KeyError          |                  映射中没有这个键                  |
|        MemoryError        |     内存溢出错误(对于Python 解释器不是致命的)      |
|         NameError         |            未声明/初始化对象 (没有属性)            |
|     UnboundLocalError     |               访问未初始化的本地变量               |
|      ReferenceError       | 弱引用(Weak reference)试图访问已经垃圾回收了的对象 |
|       RuntimeError        |                  一般的运行时错误                  |
|    NotImplementedError    |                   尚未实现的方法                   |
|        SyntaxError        |                  Python 语法错误                   |
|     IndentationError      |                      缩进错误                      |
|         TabError          |                   Tab 和空格混用                   |
|        SystemError        |                一般的解释器系统错误                |
|         TypeError         |                  对类型无效的操作                  |
|        ValueError         |                   传入无效的参数                   |
|       UnicodeError        |                 Unicode 相关的错误                 |
|    UnicodeDecodeError     |                Unicode 解码时的错误                |
|    UnicodeEncodeError     |                 Unicode 编码时错误                 |
|   UnicodeTranslateError   |                 Unicode 转换时错误                 |
|          Warning          |                     警告的基类                     |
|    DeprecationWarning     |               关于被弃用的特征的警告               |
|       FutureWarning       |           关于构造将来语义会有改变的警告           |
|      OverflowWarning      |        旧的关于自动提升为长整型(long)的警告        |
| PendingDeprecationWarning |              关于特性将会被废弃的警告              |
|      RuntimeWarning       |      可疑的运行时行为(runtime behavior)的警告      |
|       SyntaxWarning       |                  可疑的语法的警告                  |
|        UserWarning        |                 用户代码生成的警告                 |

## 三、异常的关系数图：

![1554969115258](C:\Users\lh9\Desktop\各种DOM\Python错误和异常\001.png)

## 四、异常的解决思路：

* `系统内置了很多应用场景，在程序运行过程中，一旦触发相关场景，系统就会向外抛出相应的问题，这就是系统抛出的异常`。

* 预防：添加容错处理，代码虽然会触发异常，但使用容错处理可以不让异常触发。

* 解决：如果容错代码过多时，会使得整个程序非常乱，这时可以使用捕捉异常进行处理。

  ![1554972124652](C:\Users\lh9\Desktop\各种DOM\Python错误和异常\002.png)

## 五、异常处理：

#### 1、检测到异常后, 不会往下执行：

```json
try:
    6 / 0
    print error

except ZeroDivisionError as e:
    print '除零错误'

except NameError as e:
    print 'error名字错误'

else:
    print '没有出现异常时, 会走这里'

finally:
    print '无论是否抛出异常都会走这里'
```

上面的栗子：`出现异常，直接跳过了print error`， 直接去执行 except ZeroDivisionError as e: 进行捕捉，从而不会去执行print error。

#### 2、异常合并处理：

```json
try:
    print error

except (ZeroDivisionError, NameError) as e:
    print '合并处理异常'

else:
    print '执行else中的代码块'

finally:
    print '无论是否抛出异常都会走这里'

// 执行结果：
合并处理异常
无论是否抛出异常都会走这里
```

上面的栗子：`如果多个异常的处理是相同的，可以将这些异常进行合并处理`

#### 3.使用Exception进行捕捉：

```json
try:
    print error

except Exception as e:
    print e
    print '若不知道具体的异常是什么, 可用Exception进行捕捉'
else:
    print '执行else中的代码块'

finally:
    print '无论是否抛出异常都会走这里'

// 执行结果：
name 'error' is not defined 
若不知道具体的异常是什么, 可用Exception进行捕捉
无论是否抛出异常都会走这里
```

若不知道具体是什么异常，可以直接使用Exception进行捕捉，因为这些`常见的异常都是继承自Exception`。

## 六、抛出异常：

* `使用raise语句自己触发异常`

* 语法格式如下：

  ```json
  raise[Exception [,args, [, traceback]]]
  ```

  `语句中的Exception是异常的类型(如: NameError)，args参数是一个异常的参数值。该参数是可选的，如果不提供，异常的参数是'None'`。

  ```json
  # 输入：
  raise ValueError('value error')
  
  # 输出结果：
  Traceback (most recent call last):
    File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 1433, in <module>
      raise ValueError('value error')
  ValueError: value error
  ```

