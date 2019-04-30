# Python 面向对象

## 一、面向对象跟面向过程的区别：

* **面向过程编程**：

  ```json
  1.导入各种外部库
  2.设计各种全局变量
  3.写一个函数完成某个功能
  4.写一个函数完成某个功能
  5.写一个函数完成某个功能
  6. ................
  7.写一个main函数作为程序入口
  ```

  * 面向过程编程，很多重要的数据被放置再全局数据区，这样所有的函数都可以进行访问。而且每个函数都有自己的局部数据，将某些功能代码封装到函数中，就不必重复编写，仅需调用函数即可。`从代码的组织形式来看就是根据业务逻辑从上到下垒代码 。`

  * 举个栗子：

    * 假设我们有一辆汽车，我们知道它的速度(60km/h)，以及A、B两地的距离(100km)。要算出开着这辆车从A地到B地花费的时间。   

      ```json
      // 面向过程的方法
      speed = 60.0
      distance = 100.0
      time = distance / speed
      print time  // 1.66666666667
      
      speed1 = 60.0
      distance1 = 100.0
      time1 = distance1 / speed1
      print time1  // 1.66666666667
      
      distance2 = 200.0
      time2 = distance2 / speed1
      print time2  // 3.33333333333
      
      speed2 = 150.0
      time3 = distance1 / speed2
      print time3  // 0.666666666667
      
      time4 = distance2 / speed2
      print time4  // 1.33333333333
      ```

* **面向对象编程**：

  ```json
  1.导入各种外部库
  2.设计各种全局变量
  3.决定你要的类
  4.给每个类提供完整一组操作
  5.明确地使用继承来表现不同类之间的共同点
  6.根据需要,决定是否写一个main函数作为程序入口
  ```

  * 面向对象编程中，将函数和变量进一步封装成类，类才是程序的`基本元素`，它将数据和操作紧密地连接再一起，并保护数据不会被外界的函数意外地改变。`类和类的实例(也称为对象)是面向对象的核心概念，是和面向过程编程、函数式编程的根本区别。`

  * 举个栗子：

    * 跟上述的面向过程的栗子相同，我们看看面向对象是怎样进行操作。

      ```json
      // 面向对象
      class Car:
          speed = 0
          def drive(self, distance):
              time = distance / self.speed
              print time
      car1 = Car()
      car1.speed = 60.0
      car1.drive(100.0)
      car1.drive(200.0)
      car2 = Car()
      car2.speed = 150.0
      car2.drive(100.0)
      car2.drive(200.0)
      ```

## 二、类的基本用法：

* 面向对象是通过定义`class类来定义`，简单的说，面向对象编程是只使用class类，`在class类中封装、继承的功能，并且还可以构造要传入的参数。`

  ```json
  class Dog(object):
      """创建一个类的名字为Dog, 类名规则是驼峰式命名"""
  
      def __init__(self, name):   // __init__(self) 初始化方法
          self.name = name  // 初始化变量
  
      def eat(self):
          """定义一个吃的方法"""
          print self.name  
          
  // 创建Dog类的时候, 会先执行__init__初始化方法, 首先传入name参数, 返回的dog是对象的实例, 实例可以调用类中的方法
  dog = Dog('金毛')
  dog.eat()
  // 输出结果： 金毛
  ```

* 下面详细介绍一些**`专业术语概念`**:

  **1.类(class)**：用来描述具有相同属性和方法的对象的集合。`它定义了该集合中每个对象所共有的属性和方法`。

  **2.实例**：`也称对象。通过类定义的初始化方法`，赋予具体的值，成为一个"实体"。

  **3.实例化**：`创建类的实例的过程或操作`。

  **4.实例变量**：定义在实例中的变量，只作用于`当前实例`。

  **5.类变量**：类变量是所有公有的变量。`类变量定义在类中，但在方法之外`。

  **6.数据成员**：类变量、实例变量、方法、类方法、静态方法、属性等的统称。

  **7.方法**：`类中定义的函数`。

  **8.静态方法**：`不需要实例化就可以由类执行的方法`。

  **9.类方法**：`类方法是将本身作为对象进行操作的方法`。

  **10.方法重写**：如果`从父类继承的方法不能满足子类的需求，可以对父类的方法进行改写`。

  **11.封装**：将内部实现包裹起来，对外透明，提供api接口进行调用的机制。

  **12.继承**：将一个派生类继承父类的变量和方法。

  **13.多态**：根据对象类型的不同以不同的方式进行处理。

## 三、类与实例：

```json
class Dog(object):
    """创建一个类名Dog的类: 继承object表示的是新式类"""
    description = '目前我们创建一个类的变量'

    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex

    # 定义构造的过程就是"实例化"
    def eat(self):
        """创建一个吃的方法"""
        print "{}正在吃吃锅包肉".format(self.name)

# 创建类的时候, 里面传入的参数, 取决于__init__方法中接收的参数, dog称为实例或称为对象
dog = Dog('哈士奇', 6, "男")

# 实例或对象.(点)方法
dog.eat()  # 调用类里面的eat()方法, 输出结果：哈士奇正在吃吃锅包肉

print dog.name  # 实例变量指的是实例本身拥有的变量.每个变量在内存中的位置是不一样的 输出结果：哈士奇
print dog.description  # 类变量, 在类里面找到定义的变量 输出结果：目前我们创建一个类的变量
```

## 四、调用类的三种方法：

#### 1.实例方法：

```json
class Dog(object):

    def __init__(self, name):
        self.name = name

    def eat(self):
        print "{}正在吃吃锅包肉".format(self.name)


dog = Dog('哈士奇')
dog.eat()  # 这种调用方法就是实例方法
```

#### 2.静态方法：

* `静态方法由类调用，无默认参数。并且将实例方法参数中的self去掉，再方法定义上方加上@staticmethod，就成为静态方法。`
* `静态方法属于类，但是和实例无关`。

```json
class Dog(object):
    
    # 静态方法的使用
    @staticmethod
    def eat():
        print "建议仅使用，'类名.静态方法', 调用方式"


Dog().eat()  # 调用了类的方法, 仅在类中运行而不在实例中运行的方法 
// 输出结果：建议仅使用，'类名.静态方法', 调用方式
```

#### 3.类方法：

* `类方法由类调用，采用@classmethod装饰，至少传入一个cls（代指类本身，类似self）参数`。
* `执行类方法时，自动将调用该方法的类赋值给cls`。 

```json
class Dog(object):

    # 创建一个类方法
    @classmethod
    def eat(cls, name):
        """定义一个类方法"""
        print "{}正在吃锅包肉".format(name)


dog = Dog()
dog.eat('哈士奇')  # 可以使用实例名.类方法() 输出结果:哈士奇正在吃锅包肉
Dog.eat('哈士奇')  # 类名.类方法()   输出结果：哈士奇正在吃锅包肉
```

## 五、类的特性：

#### 1.封装：

* `封装是指将数据与具体操作的实现代码放在某个对象内部，外部无法访问。必须要先调用类的方法才能启动。`

  ```json
  class Dog(object):
  
      description = '类的封装'
  
      def __int__(self, name, age, sex):
          self.name = name
          self.age = age
          self.sex = sex
  
  
  print Dog.description  //类变量, 再类里面找到定义的变量。 输出结果：类的封装
  print description
  
  // 报错信息如下
  Traceback (most recent call last):
    File "C:/Users/lh9/PycharmProjects/lvtest/apps/tests.py", line 891, in <module>
      print description
  NameError: name 'description' is not defined
  ```

#### 2.继承：

* 当我们定义一个class的时候，可以从某个现有的class继承，`新的class称为子类（Subclass），而被继承的class称为基类、父类或超类（Base class、Super class）`。

  ```json
  class Animal(object):
      """定义一个基类(父类或超类)"""
  
      def run(self):
          """基类有一个run方法"""
          print 'Animal is running'
  
  
  class Dog(Animal):
      """定义一个子类, 名为Dog, 该类继承了Aniaml父类"""
      pass
  
  
  class Cat(Animal):
      """定义一个子类, 名为Cat, 该类继承了Aniaml父类"""
      pass
  
  
  dog = Dog()
  dog.run()  // 输出结果 Animal is running
  
  cat = Cat()
  cat.run() // 输出结果 Animal is running
  ```

* 继承最大的好处是`子类获得了父类的全部功能`，由于Aniaml实现了run()方法，因此Dog和Cat作为它的子类，就自动拥有了run()方法。

#### 3.多态：

* 当子类和父类都存在相同的run()方法时，`子类的run()覆盖了父类的run()，再代码运行的时候，总是会调用子类的run()`。这样，我们就获得了继承的另一个好处，多态。

  ```json
  class Animal(object):
      """定义一个基类(父类或超类)"""
  
      def run(self):
          """基类有一个run方法"""
          print 'Animal is running'
  
  
  class Dog(Animal):
      """定义一个子类, 名为Dog"""
  
      def run(self):
          """Dog类拥有run方法"""
          print 'Dog is running'
  
  
  class Cat(Animal):
      """定义一个子类, 名为Cat"""
      def run(self):
          """Cat类拥有run方法"""
          print 'Cat is running'
  
  
  dog = Dog()
  dog.run()  // 输出结果：Dog is running
  
  cat = Cat()
  cat.run() // 输出结果：Cat is running
  ```

* 继承可以把父类的所有功能都直接拿过来，子类只需要新增自己特有的方法，也可以把父类不适合的方法覆盖重写。

* `有了继承，才能有多态`

* 旧的方式定义Python类允许不从object类继承，但这种编程方式已经严重不推荐使用。任何时候，如果没有合适的类可以继承，就继承自object类。 

## 六、重写与调用父类方法：

#### 1.方法重写：

* 当父类中的方法不符合我们需求时，可以通过重写，实现我们的需求。

  ```json
  class Animal(object):
      """创建一个基类"""
      def __init__(self):
          self.name = '所有动物的名字'
  
      def eat(self):
          print '所有的动物都具有eat的方法'
  
  
  class Dog(Animal):
      """创建一个子类"""
  
      def eat(self):
          print '子类也具有eat的方法'
  
  
  dog = Dog()
  dog.eat()  # 执行结果：子类也具有eat的方法; 默认调用子类的方法, 将父类的eat方法进行了覆盖
  ```

#### 2.调用父类的普通方法：

* 调用父类的方法，详见代码下面代码注释：

  ```json
  class Animal(object):
      """创建一个基类"""
      def __init__(self):
          self.name = '所有动物的名字'
  
      def eat(self):
          print '所有的动物都具有eat的方法'
  
  
  class Dog(Animal):
      """创建一个子类"""
  
      def eat(self):
  	    # 方法1：再重写的方法内强制调用 父类名.父类方法名() 若有参数, 则需传参
          # Animal.eat(self)  
  
  	    # 方法2: super(子类名, 子类对象).父类方法()  若有参数, 则需传参
          # super(Dog, self).eat()  
  
   	    # 方法3：仅支持Python3
          # super().eat()
          pass
  
  dog = Dog()
  dog.eat()  # 执行结果：所有的动物都具有eat的方法
  ```

#### 3.调用父类的属性：

```json
class Animal(object):
    """创建一个基类"""
    def __init__(self):
        self.name = '所有动物的名字'

    def eat(self):
        print '所有的动物都具有eat的方法'


class Dog(Animal):
    """创建一个子类"""

    def __init__(self):
        # 方法1: 在重写的方法内强制调用 父类名.父类方法名() 如果需要传参, 则传参
        # Animal.__init__(self)

        # 方法2: super(子类名, 子类对象).父类方法() 如果需要传参, 则传参
        super(Dog, self).__init__()
        
        # 方法3: 仅支持Python3
        # super().__init__()

    def eat(self):
        print '子类的动物都具有eat的方法'


dog = Dog()
print dog.name  # 执行结果: 所有动物的名字 调用父类的属性
```

## 七、魔法方法：

**1.__ doc __**:

* 说明性文档和信息。Python自建，无需自定义。

  ```
  class Dog(object):
      """描述类的定义信息"""
      def eat(self):
          pass
  
  # 打印类的说明文档
  print(Dog.__doc__)  // 输出结果：描述类的定义信息
  ```

**2.__ init __ 和 __ new __ **：

*  __ init __ 实例化方法，通过类创建实例时，自动触发执行。 

  ```json
  class Dog(object):
      def __init__(self, name):
          self.name = name
          self.age = 18
          print '类创建实例时, 自动触发执行'
  
  # 在类创建实例的时候, 触发__init__方法
  dog_obj = Dog('哈士奇')   // 执行结果： 类创建实例时, 自动触发执行
  ```

  * 以上是__ init __ 最普通的用法了, 但 __ init __ 其实不是实例化一个类的时候第一个被调用的方法。`当使用Dog('哈士奇') 这样的表达式来实例化一个类时, 最先被调用的方法其实是 __ new __ 方法。

* __ new __ 方法接收的参数虽然跟 __ init __ 一样， 但 __ init __ 是在类实例创建之后调用，而 __ new __ 方法正式创建这个类实例的方法。

  ```json
  class Dog(object):
  
      def __new__(cls, name, age):
          print '__new__ 方法'
          return super(Dog, cls).__new__(cls, name, age)
  
      def __init__(self, name, age):
          print '__init__ 方法'
          self.name = name
          self.age = age
  
      def __str__(self):
          return '%s今年已经%s' % (self.name, self.age)
  
  
  dog = Dog('哈士奇', 18)
  print dog
  # 执行结果： __new__ 方法   __init__ 方法  哈士奇今年已经18
  ```

  * 可以看到， __ new __ 方法的调用是发生在 __ init __ 之前的，其实实例化一个类的时候，具体执行逻辑：

    1.dog = Dog('哈士奇', 18)。

    2.首先执行name  和 age 参数来执行Dog类的 __ new __ 方法，这个 __ new __ 方法会返回Dog类的一个实例(一般使用super(Dog, cls). __ new __（cls, name, age）这样的方式)。

    3.用这个实例来调用类的 __ init __ 方法，上一步里面 __ new __ 产生的实例也就是 __ init __ 里面的 self。

* __ init __ 和 __ new __ 方法主要区别：

  * __ init __ 通常用于初始化一个新实例，控制这个初始化的过程，如：添加一些属性，做一些额外的操作，发生在类实例被创建完以后。`简单理解，它是实例级别的方法`
  * __ new __ 通常用于控制生成一个新实例的过程。`简单理解，它是类级别的方法`

* **用 __ new __ 来实现单例**：

  * 因为类每一次实例后产生的过程都是通过 __ new __ 来控制的，所以通过重载 __ new __ 方法，我们可以实现单例模式。

  * 单例模式：该模式的主要目的是确保`某一个类只有一个实例存在`。就是`无论再代码中创建多少个实例对象，但是创建出的对象共同指向内存中的同一个地址`。

  * 下面用 __ new __ 方法实现单例模式：

    ```json
    class Dog(object):
        instance = None  # 这个值就是将要创建的单例对象
    
        # 这三参数是解释器自动添加的, 这些参数实际上是给 __init__ 方法用
        def __new__(cls, *args, **kwargs):
            if Dog.instance is None:
                Dog.instance = object.__new__(cls)  # 调用父类的__new__ 方法来开辟内存
            return Dog.instance
    
        def __init__(self, name):
            self.name = name
    
    
    dog1 = Dog('哈士奇')
    dog2 = Dog('金毛')
    print dog1
    print dog2
    # 执行结果：
    <__main__.Dog object at 0x00000000032306D8>
    <__main__.Dog object at 0x00000000032306D8>
    ```

    * **因为类对象再本质上就符合单例模式所说的指向同一块内存，所以使用类对象来保存即将要创建的单例对象。**

**3.__ module __**:

* 表示当前操作的对象在属于哪个模块。

  ```json
  class Dog(object):
  
      def eat(self):
          pass
  
  dog = Dog()
  print(dog.__module__) // 输出结果：__main__ 表示的是当前执行模块中
  ```

**4.__ class __**：

*  表示当前操作的对象属于哪个类。

  ```json
  class Dog(object):
  
      def eat(self):
          pass
  
  dog = Dog()
  print(dog.__class__)  // 输出结果：<class '__main__.Dog'>  表示当前执行模块中的Dog类
  ```

**5.__ del __**：

* 析构方法，当对象在内存中被释放时，自动触发此方法。

* **注意**：

  * 此方法一般无须自定义，因为Python自带内存分配和释放机制，除非你需要在释放的时候指定做一些动作。
  * 析构函数的调用是由解释器在进行垃圾回收时自动触发执行的。 

  ```json
  class Dog(object):
  
      def __del__(self):
          print('马上要被回收了!')
  
  
  dog = Dog()
  del dog   // 输出结果：马上要被回收了!
  ```

**6.__ call __**：

* 如果为一个类编写了该方法，那么在该`类的实例后面加括号`，可会调用这个方法。

* 对于__ call __方法，是由`对象后加括号触发的，对象()或类()()`。

  ```json
  class Dog(object):
  
      def __init__(self):
          print('要执行init方法了！')
  
      def __call__(self, *args, **kwargs):
          print('要执行call方法了!')
  
  # 对象 = 类名（）
  dog = Dog()  # 执行__init__ 方法； 输出结果：要执行init方法了！
  # 对象()
  dog()  # 执行__call__ 方法； 输出结果：要执行call方法了!
  # 类()()
  Dog()()# 先执行__init__方法，在执行__call__ 方法；输出结果：要执行init方法了！要执行call方法了!
  ```

* 可以`用Python内建的callable()函数进行测试，判断一个对象是否可以被执行`。

  ```json
  class Dog(object):
  
      def __call__(self, *args, **kwargs):
          print('要执行call方法了!')
  
  
  print callable(Dog())  // 输出结果： True
  ```

**7.__ dict __ **：

* 列出类或对象中的所有成员。非常重要和有用的一个属性，Python自建，无需用户自己定义。 

  ```json
  class Dog(object):
      desc = '创建了一个名为Dog的类'
  
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
      def eat(self):
          print '它可以吃的方法'
  
  
  # 获取类的成员
  print(Dog.__dict__)
  // 输出结果：{'__module__': '__main__', '__doc__': None, '__dict__': <attribute '__dict__' of 'Dog' objects>, '__weakref__': <attribute '__weakref__' of 'Dog' objects>, 'eat': <function eat at 0x0000000003F15518>, '__init__': <function __init__ at 0x0000000003F154A8>, 'desc':创建了一个名为Dog的类'}
  
  # 获取 对象dog的成员
  dog = Dog('哈士奇', 18)
  print(dog.__dict__)
  // 输出结果：{'age': 18, 'name': '哈士奇'}
  ```

**8.__ str __**：

* 如果一个类中定义了str()方法，那么在打印对象时，默认输出该方法的返回值。 

  ```json
  class Dog(object):
  
      def __str__(self):
          return '在打印对象时, 默认输出该方法的返回值'
  
  
  dog = Dog()
  print(dog)
  // 输出结果：在打印对象时, 默认输出该方法的返回值
  ```

**9.__ getitem __ 、__ setitem __ 、__ delitem __ **：

* 在标识符后面加中括号[]，通常代表取值的意思。它们分别表示，取值、赋值、删除数据。

  ```json
  obj = 标识符[]  // 执行__getitem__ 方法
  标识符[] = obj  // 执行__setitem__ 方法
  del 标识符[]  // 执行__delitem__ 方法
  ```

  ```json
  class Dog(object):
  
      def __getitem__(self, item):
          """取值"""
          print('__getitem__', item)
  
      def __setitem__(self, key, value):
          """赋值"""
          print('__setitem__', key, value)
  
      def __delitem__(self, key):
          print('__delitem__', key)
  
  
  dog = Dog()
  result = dog['哈士奇']  # 自动触发执行__getitem__ 输出结果：('__getitem__', '哈士奇')
  dog['哈士奇'] = '16'  # 自动触发执行__setitem__ 输出结果: ('__setitem__', '哈士奇', '16')
  del dog['哈士奇']  # 自动触发执行 __delitem__  输出结果：('__delitem__', '哈士奇')
  ```

**10.__ iter __ **：

* 迭代器方法，列表、字段、元组之所以可以进行for循环，是因为内部定义了__ iter __ 这个方法。

* 若想让自定义的类的对象可以被迭代，那么就需要在类中定义这个方法，并让该方法的返回值是一个可迭代的对象。`当在代码中使用for循环遍历对象时，就会调用类的这个__ iter __方法。

  ```json
  // 下面举一个常见的错误栗子
  class Dog(object):
      pass
  
  dog = Dog()
  for obj in dog:
      print obj  
  # 报错信息：TypeError: 'Dog' object is not iterable
  # 原因: dog对象 不可以被迭代
  
  
  # 下面添加一个__iter__(), 但是什么都不返回
  class Dog(object):
      def __iter__(self):
          pass
  
  dog = Dog()
  for obj in dog:
      print obj
  # 报错信息:TypeError: iter() returned non-iterator of type 'NoneType'
  # 原因是__iter__ 方法没有返回一个可迭代的对象
  ```

  ```json
  // 下面举一个正确的栗子：
  class Dog(object):
  
      def __init__(self, my_list):
          self.my_list = my_list
  
      def __iter__(self):
          return iter(self.my_list)  // 返回一个iter()可迭代对象
  
  
  dog = Dog([10, 20, 30, 40, 50])
  for item in dog:
      print item
  # 输出结果：
  10
  20
  30
  40
  50
  ```

**11.__ len __ **：

* 返回元素的个数。

  ```json
  class Dog(object):
  
      def __init__(self, *args):
          self.name = args
  
      def __len__(self):
          return len(self.name)
  
  
  dog = Dog('哈士奇', '金毛')
  print len(dog) // 输出结果：2
  ```

**12.__ repr __**：

* 这个方法的作用和__ str __ 很像，`两者的区别是：__ str __ 返回用户看到的字符串，而 __repr __ 返回程序开发者看到的字符串，简单说 __ repr __ 是为了调试服务的`。

* 下面说说二者的区别吧：

  ```json
  
  In [2]: class Dog(object):
     ...:     def __init__(self, name):
     ...:         self.name = name
     ...:
     ...:     def __str__(self):
     ...:         return 'this is %s' %  self.name
     ...:
  
  In [3]: dog = Dog("哈士奇")
  
  In [4]: dog
  Out[4]: <__main__.Dog at 0x5062518>
  
  In [5]: class Dog(object):
     ...:     def __init__(self, name):
     ...:         self.name = name
     ...:     def __repr__(self):
     ...:         return 'this is %s' % self.name
     ...:
  
  In [6]: dog = Dog('哈士奇')
  
  In [7]: dog
  Out[7]: this is 哈士奇
  ```

**13.__ add __ ：加运算、 __ sub __ ：减运算、__ mul __ ：乘运算、 __ div __ ：除运算、__ mod __ ：求余运算、 __ pow __ ：幂运算**：

* 这些都是算术运算方法。

  ```json
  class Number(object):
  
      def __init__(self, numner):
          self.numner = numner
  
      def __add__(self, other):
          print '使用加号的时候, 执行__add__方法'
          return self.numner + other
  
      def __sub__(self, other):
          print '使用减号的时候, 执行__sub__方法'
          return self.numner - other
  
      def __mul__(self, other):
          print '使用乘号的时候, 执行__mul__方法'
          return self.numner * other
  
  
  number = Number(6)
  print number + 2  # 输出结果： 调用加号的时候, 执行__add__方法  7
  print number - 2  # 输出结果： 使用减号的时候, 执行__sub__方法  3
  print number * 2  # 输出结果： 使用乘号的时候, 执行__mul__方法  10
  ```

**14.__ slots __**：

* Python作为一种动态语言，可以在类定义完成和实例化后，给类或者对象继续添加随意个数或者任意类型的变量或方法，这是动态语言的特性。

* 正常情况下，当我们定义一个class， 创建了一个class实例后，我们可以给该实例绑定任何属性和方法。但是，如果我们想要限制实例的属性怎么办？这时我们可以在定义class的到时候，`定义一个特殊的__ slots __ 变量，来限制该class实例能添加的属性`。

  ```json
  class Dog(object):
  
      __slots__ = ('name', 'age')  # 用tuple定义允许绑定的属性名称
  
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
      def eat(self):
          print '{}它今年{}'.format(self.name, self.age)
  
  
  dog = Dog('金毛', 19)
  dog.eat()  # 打印结果： 金毛它今年19
  
  dog.name = '哈士奇'
  dog.age = 18
  dog.eat()  # 打印结果:  哈士奇它今年18  说明给属性赋值成功
  
  dog.sex = '公'  # 报错信息：AttributeError: 'Dog' object has no attribute 'sex'
  # 原因: 由于'sex' 没有被放到__slots__中，所以不能绑定sex属性
  ```

* **注意：使用__ slots __ 定义的属性仅对当前类实例起作用，对继承的子类是不起作用的。**

  ```json
  class Dog(object):
  
      __slots__ = ('name', 'age')  # 用tuple定义允许绑定的属性名称
  
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
      def eat(self):
          print '{}它今年{}'.format(self.name, self.age)
  
  
  class HaShiQi(Dog):
      """创建一个哈士奇的子类, 继承Dog类"""
      pass
  
  
  hashiqi = HaShiQi('哈士奇', 20)
  # 可以子类可以使用'sex'属性
  hashiqi.sex = '男'
  ```

* 除非在子类中也定义__ slots __ ，子类实例允许定义的属性就是自身的 __ slots __ 加上父类的 __ slots __。

  ```json
  class Dog(object):
  
      __slots__ = ('name', 'age')  # 用tuple定义允许绑定的属性名称
  
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
      def eat(self):
          print '{}它今年{}'.format(self.name, self.age)
  
  
  class HaShiQi(Dog):
      """创建一个哈士奇的子类, 继承Dog类"""
  
      __slots__ = ('sex')
  
  hashiqi = HaShiQi('哈士奇', 20)
  # 如果子类设置了__slots__, 那么子类的对象可以调用父类的 __slots__允许属性
  hashiqi.name = '哈士奇的名字'
  # 如果子类设置了__slots__, 那么子类的或父类中的__slots__ 都没有该属性的话, 
  # 直接报错: AttributeError: 'HaShiQi' object has no attribute 'nickname'
  hashiqi.nickname = '哈士奇的昵称'
  ```

## 八、访问限制：

* `如果想让内部属性不被外部访问，可以把属性的名称前加上两个下划线__， 在Python中，实例的变量如果以双下划线开头，就变成了一个私有变量，只有内部才可以访问，外部不能访问`。

  ```json
  class Dog(object):
  
      def __init__(self, name, age):
          self.__name = name
          self.__age = age
  
      def eat(self):
          print('%s今年%s' % self.__name, self.__age)
  
  
  dog = Dog('哈士奇', 20)
  dog.__name  # 报错信息：AttributeError: 'Dog' object has no attribute '__name'
  // 对于外部代码来说，没有什么变动，但是已经无法从外部访问
  ```

* **使用get_set_del方法操作私有成员**：

  ```json
  class Dog(object):
      def __init__(self, name):
          self.name = name
  
      def eat(self):
          print("%s在吃东西" % self.name)
  
      # 加上双下划线的就是私有变量, 只能在类的内部访问, 外部无法访问
      __age = 18
  
      # 如果在类中调用, 首先调用类方法
      @classmethod
      def method(cls):
          print(cls.__age)
  
      # 在调用set_ 方法, 改变__age的值
      @classmethod
      def set_age(cls, value):
          cls.__age = value
          return cls.__age
  
      # 调用get_ 方法, 直接返回__age的值
      @classmethod
      def get_age(cls):
          return cls.__age
  
      # 调用del_ 方法, 直接删除__age的值
      @classmethod
      def del_age(cls):
          del cls.__age
  
  
  print Dog.get_age()  # 执行结果：18 因为调用的是__age 私有属性
  print Dog.set_age(20)  # 执行结果：20  因为直接改变了私有属性__age 的值
  print Dog.del_age()  # 执行结果：None  因为直接删除了__age的值
  ```

* 双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。不能直接访问__ age 是`因为Python解释器对外把__ age 变成了 _Dog __ age` , 所以仍然可以通过 _ Dog __ age来访问 __ age变量。

  ```json
  print(dog._Dog__age)  // 对象._类名__私有属性
  ```

  * **强烈建议不要这么操作，因为不同版本的Python解释器可能会把__ age 改成不同的变量名**。
  * Python的访问限制没有那么严格，主要靠自觉。

## 九、Property装饰器：

* 把类的方法伪装成属性调用的方式，就是把`类里面的一个函数，变成一个属性`。

* 下面举个常规操作的栗子：

  ```json
  class Dog(object):
      def __init__(self, name, age):
          self.__name = name
          self.__age = age
  
      def get_age(self):
          return self.__age
  
      def set_age(self, value):
          if isinstance(value, int):
              self.__age = value
          else:
              raise ValueError('非整数类型')
  
      def del_age(self):
          print '删除该操作'
  
  
  dog = Dog('哈士奇', 18)
  print dog.get_age()  # 输出结果：18
  dog.set_age("20")  # 报错：ValueError: 非整数类型
  dog.set_age(20)
  print dog.get_age()   # 输出结果：20 已经将传入value进行了赋值__age
  ```

* 下面举个Property装饰器栗子：

  ```json
  class Dog(object):
      def __init__(self, name, age):
          self.__name = name
          self.__age = age
  
      # 将age()函数, 更改了属性
      @property
      def age(self):
          print '将age方法变成属性'
          return self.__age
  
      @age.setter
      def age(self, value):
          print '调用了setter方法'
          if isinstance(value, int):
              self.__age = value
  
          else:
              raise ValueError('非整数类型')
  
      @age.deleter
      def age(self):
          print '删除了__age'
  
  # 创建对象
  dog = Dog('哈士奇', 18)
  # 通过@prepety装饰器, 将age()方法, 变成了一个属性, 可以通过对象.属性 就可以获取
  print dog.age  # 输出结果：将age方法变成属性  18
  
  # 使用setter把值进行转换
  dog.age = 20
  print dog.age   # 输出结果：调用了setter方法;将age方法变成属性;20
  
  # 删除age
  del dog.age  # 输出结果：删除了__age
  ```

  * 这种的调用方法有些麻烦，每次都是一个个去实例类与对象，有个更简单的直观的方法。

* 下面举个`更加简单直观的使用property()函数`栗子：

  * 除了使用装饰器的方式将一个方法伪装成属性外，`Python内置的builtins模块中的property()函数，提供了第二种设置类属性的手段`。

  ```json
  class Dog(object):
  
      def __init__(self, name, age):
          self.__name = name
          self.__age = age
  
      def get_age(self):
          return self.__age
  
      def set_age(self, age):
          print '调用了赋值的方法'
          if isinstance(age, int):
              self.__age = age
          else:
              raise ValueError('非整数类型')
  
      def del_age(self):
          print '删除__age'
  
      age = property(get_age, set_age, del_age, '年龄')
  
  
  dog = Dog('哈士奇', 18)
  print dog.age  # 输出结果: 18
  dog.age = 20   # 输出结果： 调用了赋值的方法; 20
  print dog.age  # 输出结果:  20
  del dog.age    # 输出结果: 删除了__age
  ```

  * 通过语句age = property(get_age, set_age, del_age, '年龄') 将一个方法伪装成了属性，效果和装饰器的方法是一样的。

  * `property()函数的参数`:

    ```json
    第一个参数是方法名, 调用  实例.属性 时自动执行的方法
    第二个参数是方法名, 调用  实例.属性 = xxx 时自动执行的方法
    第三个参数是方法名, 调用  del 实例.属性 时自动执行的方法
    第四个参数时字符串, 调用  实例.属性.__doc__ 时的描述信息
    ```

    

    