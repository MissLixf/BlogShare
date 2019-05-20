# python 操作redis数据库

## 一、安装redis-py：

首先要有一台启动的redis服务器，接下来进入本机的虚拟环境，可以使用命令进行安装：

* 使用：`pip install redis`命令进行安装

#### 1.1：redis连接：

**StrictRedis跟Redis的区别在于，StrictRedis用于实现大部分官方命令，并使用官方的语法和命令，Redis是StrictRedis的子类，兼容一些老版本。**

**Redis连接实例是线程安全的，可以直接将redis连接实例设置为一个全局变量，直接使用。**

* 举个栗子：

  ```json
  import redis  # 导入安装的redis包
  r = redis.StrictRedis(host='127.0.0.1', port=6379, db=0)  # 指定IP地址、端口、指定存放数据库
  # 如果指定一些其他参数，可以看源码，将需要的参数进行指定。
  ```

  ```json
  import redis
  r = redis.Redis(host='localhost', port=6379, db=0)
  ```

#### 1.2：连接池：

**redis-py使用connection pool来管理对一个redis server的所有连接，避免每次建立、释放连接的开销。每个redis实例都会维护一个自己的连接池。**

**可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池。**

* 举个栗子：

  ```json
  # 导入redis模块
  import redis
  pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
  r = redis.Redis(connection_pool=pool)
  ```

## 二、字符串(String)相关操作：

|     方法      |           简介           |
| :-----------: | :----------------------: |
|     set()     |         设置键值         |
|    setnx()    |    name不存在才会执行    |
|    setex()    |    设置值过期时间(秒)    |
|   psetex()    |   设置值过期时间(毫秒)   |
|   strlen()    | 获取name对应值的字节长度 |
|     get()     |          获取值          |
|   getset()    |          更新值          |
|    mset()     |        设置多个值        |
|    mget()     |        获取多个值        |
|  getrange()   |        获取子序列        |
|  setrange()   |      修改字符串内容      |
|    incr()     |     value(整数)自增      |
| incrbyfloat() |    value(浮点数)自增     |
|    decr()     |     value(整数)自减      |
|   append()    |        追加字符串        |

#### 2.1、set()：

```json
result = r.set('key', 'value: hello redis') # key 代表是键  value:hello redis 代表是值
print result  # 打印结果：True 说明设置成功
```

**字符串操作，redis中的String再内存中按照一个`name对应一个value`来存储。**

`set(name, value, ex=None, px=None, nx=False, xx=False)`

* 再Redis中设置值，不存在则创建、存在即修改

* 参数：

  ex：过期时间(秒)

  px：过期时间(毫秒)

  nx：如果为True，当name不存在时，当前set操作会执行

  xx：如果为True，当name存在时，当前set操作会执行

##### 2.1.1：ex-过期时间(单位:秒)：

```json
# ex: 设置过期时间(单位:秒)  'name'为Key  'fe_cow' 为value
r.set('name', 'fe_couw', ex=3)

# 输出结果：fe_cow  但是3秒后 输出结果：None
print r.get('name')
```

##### 2.1.2：px-过期时间(单位:毫秒)

```json
# px: 设置过期时间(单位:豪秒)  'name'为Key  'fe_cow' 为value
r.set('name', 'fe_couw', ex=3)

# 输出结果：fe_cow  但是3豪秒后 输出结果：None
print r.get('name')
```

##### 2.1.3：nx：

**'name'不存在，set才会执行**

```json
# 若Redis数据库中没有name='name'
print r.set('name', 'fe_cow', nx=True)
# 输出结果： True  说明set设置成功

#  若再执行一次, 因redis数据库中已经存在name='name'
print r.set('name', 'fe_cow', nx=True)
# 输出结果：None 说明set设置失败, 没有重复添加
```

##### 2.1.4：xx:

**'name'存在，set才会执行**

```json
# 若redis数据库中存在name='name'
print r.set('name', 'fe_cow', xx=True)
# 输出结果：True  设置成功, 看到数据库中有追加一条相同信息

# 若redis数据库中不存在name='nickname'
print r.set('nickname', 'fe_cow', xx=True)
# 输出结果: None 说明set 失败
```

#### 2.2、setnx(name, value)：

**name不存在才会执行**

```json
# 若redis数据库中不存在name='nickname'
print r.setnx('nickname', 'fecow')
# 输出结果： True 添加成功
```

#### 2.3、setex(name, time, value)：

**设置值与过期时间**

```json
# name='age'  5代表过期时间(单位：秒)  value='18'
r.setex('age', 5, '18')  
# 输出结果： '18'   5秒过后 输出结果： None
print r.get('age')
```

#### 2.4、psetex(name, time_ms, value)：

**设置值与过期时间(毫秒)**

```json
# name='mobile' value='130xxxxxx'  50代表过期时间(单位毫秒)
r.psetex('mobile', 50, '130xxxxxx')
# 输出结果：'130xxxxxx'   50毫秒过后输出结果为: None
print r.get('mobile')
```

#### 2.5、strlen(name):

**获取name对应值的字节长度**

```json
# 获取name='title' 所对应的value(gold)长度
print r.strlen('title')
# 输出结果: 4

# 获取name='detail', 所对应的value('详情信息')长度
print r.strlen('detail')
# 输出结果： 12  一个汉字=3个字节
```

#### 2.6、get-获取值：

```json
result = r.get('key')  # 获取键的名字
print result  # 打印结果：value: hello redis   是键对应的值
```

#### 2.7、getset(name, value)：

**设置新值并获取原来的值：**

```json
# 找到redis 数据库中name='mobile' 将它的value 替换成'131xxxxx'，相当于update()操作
r.getset('mobile', '131xxxxx')
print r.get('mobile')
# 输出结果： 131xxxxx
```

#### 2.8、mset(**kw):

**设置多个键值：**

```json
# 可以准备一个字典, 每个键对应的值，进行批量添加
mset_dict = {
    'key1': 'value1',
    'key2': 'value2'
}
result = r.mset(mset_dict)
print result   # 打印结果：True 设置多个键值成功
```

#### 2.9、mget(**kw)

**获取多个键值**

```json
# mget接收的参数, 是多个key；如：'key1', 'key2'
mget_list = ['key1', 'key2']
result = r.mget(mget_list) 
print result  # 打印结果：['value1', 'value2'] 是每个键所对应的值，返回一个列表
```

#### 2.10、getrange(key, start, end):

**获取子序列(根据字节获取,非字符)**

`一个汉字3个字节 1个字母一个字节 每个字节8bit`

* 参数：

  name：redis的name

  start：起始位置(字节)

  end：结束位置(字节)

* 举个汉字栗子：

  ```json
  r.set('detail', '详情信息')
  # 取索引号是0-2 前3位字节
  print r.getrange('detail', 0, 2)
  # 输出结果：详
  
  # 取所有的信息
  print r.getrange('detail', 0, -1)
  # 输出结果：详情信息
  ```

* 举个字母栗子：

  ```json
  r.set('title', 'good')
  print r.getrange('title', 0, 2)
  # 输出结果：goo
  
  print r.getrange('title', 0, -1)
  # 输出结果：good
  ```

#### 2.11、setrange(name, offset, value):

**修改字符串内容，从指定字符串索引开始向后替换(新值过长时，向后添加)**

* 参数：

  offset：字符串索引

  value：要设置的值

  ```json
  # 原'title'所对应的值：'good'，将索引位置1开始, 替换成'old' 
  r.setrange('title', 1, 'old')
  
  # 输出结果：'gold'
  print r.get('title')
  ```

#### 2.12、incr(name, amount=1)：

**自增name对应的值，当name不存在时，创建name=amount，存在则自增**

* 参数：

  name：redis的name

  amount：自增数(必须是整数)

  ```json
  # 自增name='incr' 每次自增 +1
  r.incr('incr', amount=1)
  print r.get('incr')
  # 输出结果：+1
  ```

* 应用场景：记录页面的点击次数。

#### 2.13、incrbyfloat(name, amount=1.0):

**自增name对应得值，当name不存在时，创建name=amount，存在则自增**

* 参数：

  name：redis的name

  amount：自然数(浮点数)

  ```json
  # 设置name='num'
  r.set('num', '0.00')
  print r.get('num')
  # 输出结果：0.00
  
  # 自增浮点数
  r.incrbyfloat('num', amount=1.0)
  ```

#### 2.14、decr(name, amout=1):

**自减name对应的值，当name不存在时，创建name=amount，存在则自增**

* 参数：

  name：redis的name

  amount：自减数(整数)

  ```json
  # 先查询name='num' 是否存在, 若存在 将对应的value 自减 1 ，不存在创建
  r.decr('num', amount=1)
  print r.get('num')
  # 输出结果： -1
  ```

#### 2.15、append(key, value):

**再redis  name对应的值后面追加内容**

* 参数：

  key：redis的name

  value：追加的字符串

  ```json
  # 原name='name' 对应的value='fe_cow'
  r.append('name', 'au_cow')
  print r.mget('name')
  # 输出结果：['fe_cowau_cow'] 将value 拼接再一起
  ```

## 三、列表(List)相关操作：

|     方法      |                          简介                           |
| :-----------: | :-----------------------------------------------------: |
|    lpush()    |            增加（从左边新加，不存在就新建）             |
|    rpush()    |           增加（从右边新增加，不存在就新建）            |
|    llen()     |                      获取列表长度                       |
|   lpushx()    |   已经有的name的列表的左边添加元素，没有的话无法创建    |
|   rpushx()    |   已经有的name的列表的右边添加元素，没有的话无法创建    |
|   linsert()   |             新增（固定索引号位置插入元素）              |
|    lset()     |               修改（指定索引号进行修改）                |
|    lrem()     |                 删除（指定值进行删除）                  |
| lpop()/rpop() |                       删除并返回                        |
|    ltrim()    |                    删除索引之外的值                     |
|   lindex()    |                 取值（根据索引号取值 ）                 |
| brpoplpush()  | 移动 （元素从一个列表移动到另外一个列表 可以设置超时 ） |

#### 3.1、lpush(name, values):

**从左边新增加，不存在就新建**

```json
# 往name='list' 中从左添加value=[11, 22, 33]
r.lpush('list', 11, 22, 33)

print r.lrange('list', 0, -1)
# 输出结果：['33', '22', '11']  保存顺序为 33，22，11
```

#### 3.2、rpush(name, values):

**从右边新增加，不存在就新建**

```json
# name='list1' 中从右添加value=[66, 55, 44]
r.rpush('list1', 66, 55, 44)

print r.lrange('list1', 0, -1)
# 输出结果：['66', '55', '44']
```

#### 3.3、llen(name):

**获取列表长度**

```json
# 获取name='list1' 列表中的长度 value=[66, 55, 44]
print r.llen('list1')
# 输出结果： 3
```

#### 3.4、lpushx(name, value):

**往已经有的name的列表的左边添加元素，没有的话无法创建 **

```json
# 首先redis数据库中没有name='list2'
r.lpushx('list2', 10)
print r.lrange('list2', 0, -1)
# 输出结果： []  
```

#### 3.5、rpushx(name, value):

**往已经有的name的列表的右边添加元素，没有的话无法创建**

```json
# 首先redis数据库中没有name='list2'
r.rpushx('list2', 10)
print r.lrange('list2', 0, -1)
# 输出结果： []
```

#### 3.6、linsert(name, where, refvalue, value):

**在name对应的列表的某一个值前或后插入一个新值**

* 参数：

  name：redis的name

  where：BEFORE或AFTER

  refvalue：标杆值 (以它为基础, 前后插入新值)

  value：要插入的数据

  ```json
  # 往列表中左边第一个出现的元素"66"前插入元素"77", 若name不存在, 不会新创建, 会返回[]
  r.linsert('list1', 'before', '66', '77')  # 目前数据库list1=[66, 55, 44]
  print r.lrange('list1', 0, -1)
  # 输出结果： ['77', '66', '55', '44']
  ```

#### 3.7、lset(name, index, value)：

**对name对应的list中的某一个索引位置重新赋值**

* 参数：

  name：redis的name

  index：list的索引位置

  value：要设置的新值

  ```json
  # 将list1中索引为3, 替换成'33', list1=['77', '66', '55', '44']
  r.lset('list1', 3, '33')
  print r.lrange('list1', 0, -1)
  # 输出结果: ['77', '66', '55', '33']
  ```

#### 3.8、lrem(name, value, num):

**name对应的list中删除指定的值**

* 参数：

  name：redis的name

  value：要删除得值

  num：num=0，删除列表中所有的值

  ​	    num=2，从前向后，删除2个；

  ​	    num=-2， 从后向前，删除2个；

  ```json
  # list1=['77', '66', '55', '33']
  
  # 从左向右, 找到value='66' 删除一个
  r.lrem("list1", 1, "66")
  print r.lrange('list1', 0, -1)
  # 输出结果：['77', '55', '33']
  
  # list1=['77', '55', '33']
  # 从右向左, 找到value='55', 删除一个
  r.lrem('list1', -1, '55')
  print r.lrange('list1', 0, -1)
  # 输出结果：['77', '33']
  
  # list1=['77', '77', '33']
  # 删除name='list1'中 value='77'的所有值
  r.lrem('list1', 0, '77')
  print r.lrange('list1', 0, -1)
  # 输出结果：
  ```

#### 3.9、lpop(name) 、rpop(name):

**在name对应的列表的左侧/右侧获取第一个元素并在列表中移除，返回值则是第一个元素**

```json
# list1 = ['77', '66', '55']

result = r.lpop('list1')
print result  # 输出结果：77
print r.lrange('list1', 0, -1)  # 输出结果：['66', '55']

# list1 = ['66', '55']

result = r.rpop('list1')
print result  # 输出结果：55
print r.lrange('list1', 0, -1)  # 输出结果：['66']
```

#### 3.10、ltrim(name, start, end):

**name对应的列表中移除没有在start-end索引之间的值**

* 参数：

  name：redis的name

  start：索引的起始位置

  end：索引的结束位置

  ```json
  # list1=['77', '66', '55', '44', '33']
  
  # 删除name='list1'中, 不包含索引0-2的value
  r.ltrim('list1', 0, 2)
  print r.lrange('list1', 0, -1)
  # 输出结果：['77', '66', '55']
  ```

#### 3.11、lindex(name, index):

**在name对应的列表中根据索引获取列表元素**

```json
# list1=['77', '66', '55']

# 取出name='list1'中索引为1的值
print r.lindex('list1', 1)  # 输出结果：66
```

#### 3.12、brpoplpush(src, dst, timeout=0):

**从一个列表的右侧移除一个元素并将其添加到另一个列表的左侧**

* 参数：

  src：取出并要移除元素的列表对应的name

  dst：要插入元素的列表对应的name

  timeout：当src对应的列表中没有数据时，阻塞等待其有数据的超时时间（秒），0 表示永远阻塞 

  ```json
  # list = ['33', '22', '11']
  # list1 = ['77', '66', '55']
  
  # 将name='list1' 中的Value 全部插入到name='list'中
  r.brpoplpush('list1', 'list', timeout=2)
  print r.lrange('list', 0, -1)
  # 输出结果：['77', '66', '55', '33', '22', '11'] 
  ```

## 四、集合(Set)相关操作：

|     方法      |               简介               |
| :-----------: | :------------------------------: |
|    sadd()     |               新增               |
|    scard()    |           获取元素个数           |
|  smembers()   |        获取集合中所有成员        |
|    sscan()    |   获取集合中所有成员(元组形式)   |
| sscan_iter()  | 获取集合中所有成员(迭代器的方式) |
|    sdiff()    |               差集               |
| sdiffstore()  |   差集--差集存在一个新的集合中   |
|   sinter()    |               交集               |
| sinterstore() |   交集--交集存在一个新的集合中   |
|   sunion()    |               并集               |
| sunionstore() |    并集--并集存在一个新的集合    |
|  sismember()  |     判断value是否存在name中      |
|    smove()    |               移动               |
|    spop()     |  删除--随机删除并且返回被删除值  |
|    srem()     |         删除--指定值删除         |



#### 4.1、sadd(name, values)：

**name对应的集合中添加元素**

```json
# 往集合中添加元素 name='set1'
r.sadd('set1', 1, 2, 3, 4, 5, 6)
print r.smembers('set1')
# 输出结果:set(['1', '3', '2', '5', '4', '6'])
```

#### 4.2、scard(name):

**name对应集合中元素个数**

```json
# 'set1' = set(['1', '3', '2', '5', '4', '6'])
print r.scard('set1')
# 输出结果： 6
```

#### 4.3、smembers(name):

**name对应的集合所有成员**

```json
# 获取name='set1' 中的所有values值
print r.smembers('set1')
# 输出结果: set(['1', '3', '2', '5', '4', '6'])
```

#### 4.4、sscan(name, cursor=0, match=None, count=None):

**name对应集合所有成员(元组形式)**

```json
print r.sscan('set1')
# 输出结果：(0L, ['1', '2', '3', '4', '5', '6'])
```

#### 4.5、sscan_iter(name, match=None, count=None):

**name对应集合所有成员(迭代器的方式)**

```json
for item in r.sscan_iter('set1'):
    print item
# 输出结果如下：
1
2
3
4
5
6
```

#### 4.6、sdiff(key, *args)：

**求集合中的差集**

```json
# name是set1、set2中values的值
# set1 = set(['1', '3', '2', '5', '4', '6'])
# set2 = set(['8', '5', '7', '6'])

# 在集合set1中但是不在集合set2中的values值
print r.sdiff('set1', 'set2')
# 输出结果:set(['1', '3', '2', '4'])

# 在集合set2中但是不在集合set1中的values值
print r.sdiff('set2', 'set1')
# 输出结果：set(['8', '7'])
```

#### 4.7、sdiffstore(dest, keys, *args)：

**将两个集合中的差集，存储到第三个集合中**

```json
# 在集合set1但是不再集合set3中的values值, 存储到set3集合中
r.sdiffstore('set3', 'set1', 'set2')
print r.smembers('set3')
# 输出结果：set(['1', '3', '2', '4'])
```

#### 4.8、sinter(keys，*args)：

**获取两个集合中的交集**

```json
# set1=set(['3', '5', '6'])
# set2=set(['8', '5', '4', '7', '6'])

# 求集合set1与set2的交集
print r.sinter('set1', 'set2')
# 输出结果：set(['5', '6'])
```

#### 4.9、sinterstore(dest, keys, *args)：

**将两个集合中的交集，存储到第三个集合中**

```json
# 将set1与set2的交集，存储到set3中
print r.sinterstore('set3', 'set1', 'set2')
# 输出结果： 2 说明存储进去两个

print r.smembers('set3')
# 输出结果：set(['5', '6'])
```

#### 4.10、sunion(keys, *args):

**获取多个name对应的集合并集**

```json
# 获取set1与set2集合中的并集
print r.sunion('set1', 'set2')
# 输出结果：set(['1', '3', '2', '5', '4', '7', '6', '8'])
```

#### 4.11、sunionstore(dest, keys, *args)：

**将两个集合中的并集，存储到第三个集合中**

```json
# 将set1与set2集合中的并集, 存储到set3中
print r.sunionstore('set3', 'set1', 'set2')
print r.smembers('set3')
# 输出结果：set(['1', '3', '2', '5', '4', '7', '6', '8'])
```

#### 4.12、sismember(name, value):

**判断是否是集合的成员**

```json
# 校验value=3 是否在name=set1集合中
print r.sismember('set1', 3)
# 输出结果：True表示在集合中

print r.sismember('set1', 33)
# 输出结果： False表示不在集合中
```

#### 4.13、smove(src, dst, value):

**将某个成员从一个集合中移动到另外一个集合**

```json
# 目前集合set1、set2中的元素
# set1=set(['1', '3', '2', '5', '4', '6'])
# set2=set(['8', '5', '7', '6'])

# 将set1中的value=4的元素, 移动到set2集合中
r.smove('set1', 'set2', 4)
print r.smembers('set1')
# 输出结果：set(['1', '3', '2', '5', '6'])

print r.smembers('set2')
# 输出结果：set(['8', '5', '4', '7', '6'])
```

#### 4.14、spop(name):

**从集合移除一个成员, 并将其返回(集合是无序的，所有移除也是随机的)**

```json
# 目前set1集合中元素 set1=set(['1', '3', '2', '5', '6'])
# 随机删除'set1'中的Value值
print r.spop('set1')
# 输出结果：1  说明删除的个数

print r.smembers('set1')
# 输出结果：set(['3', '2', '5', '6'])
```

#### 4.15、srem(name, values):

**在name对应的集合中删除某些值**

```json
# 目前set1=set(['3', '2', '5', '6'])
print r.srem('set1', 2)
# 输出结果：1

print r.smembers('set1')
# 输出结果：set(['3', '5', '6'])
```

## 五、有序集合(Set)相关操作：

|        方法        |                       简介                       |
| :----------------: | :----------------------------------------------: |
|       zadd()       |                       新增                       |
|      zcard()       |               获取有序集合元素个数               |
|      zrange()      |              获取有序集合的所有元素              |
|    zrevrange()     |   从大到小排序(同zrange，集合是从大到小排序的)   |
|      zscan()       |        获取所有元素--默认按照分数顺序排序        |
|    zscan_iter()    |               获取所有元素--迭代器               |
|      zcount()      | name对应的有序集合中分数 在 [min,max] 之间的个数 |
|     zincrby ()     |                       自增                       |
|      zrank()       |                  获取值的索引号                  |
|       zrem()       |                 删除--指定值删除                 |
| zremrangebyscore() |              删除--根据分数范围删除              |
|      zscore()      |                 获取值对应的分数                 |

* `Set操作，Set集合就是不允许重复的列表，本身是无序的。`
* 有序集合，在集合的基础上，为每元素排序；元素的排序需要根据另外一个值来进行比较，  所以，`对于有序集合，每一个元素有两个值，即：值和分数，分数专门用来做排序`。

 #### 5.1、zadd(name, *args, **kwargs)：

**在name对应的有序集合中添加元素**

```json
# 新建有序集合 name='zset1' value='n1', 'n2'  score='11', '22'
r.zadd('zset1', n1=11, n2=22)
print r.zrange('zset1', 0, -1)
# 输出结果：['n1', 'n2'] 显示的是value值
```

```json
# 新建有序集合 name='zset2', value='m1', 'm2'  score='22', '33'
r.zadd('zset2', m1=22, m2=33)
# withscores=True 获取有序集合中所有元素和分数
print r.zrange('zset2', 0, -1, withscores=True)  
# 输出结果：[('m1', 22.0), ('m2', 33.0)] 
```

#### 5.2、zcard(name):

**获取name对应的有序集合元素的数量**

```json
# zset2=[('m1', 22.0), ('m2', 33.0)]

# 获取name='zset2'中value的个数
print r.zcard('zset2')
# 输出结果：2
```

#### 5.3、zrange( name, start, end, desc=False, withscores=False, score_cast_func=float):

**按照索引范围获取name对应的有序集合的元素**

* 参数：

  name：redis的name

  start：有序集合索引起始位置（非分数）

  end：有序集合索引结束位置（非分数） 

  desc：排序规则，默认按照分数从小到大排序

  withscores：是否获取元素的分数，默认只获取元素的值

  score_cast_func：对分数进行数据转换的函数 

##### 5.3.1：zrevrange(name, start, end, withscores=False, score_cast_func=float):

**从大到小排序(同zrange，集合是从大到小排序的)**

```json
# 仅获取元素、不显示分数, 分数倒序
print r.zrevrange('zset2', 0, -1)
# 输出结果：['m2', 'm1']

# 获取有序集合中所有元素和分数, 分数倒序
print r.zrevrange('zset2', 0, -1, withscores=True)
# 输出结果：[('m2', 33.0), ('m1', 22.0)]
```

#### 5.4、zscan(name, cursor=0, match=None, count=None, score_cast_func=float)：

**获取所有元素--默认按照分数顺序排序**

```json
# 获取name=zset2所有的元素, 按照分数顺序排序
print r.zscan('zset2')
# 输出结果：(0L, [('m1', 22.0), ('m2', 33.0)])
```

#### 5.5、zscan_iter(name, match=None, count=None,score_cast_func=float):

**获取所有元素--迭代器**

```json
# 遍历迭代器
for item in r.zscan_iter('zset2'):
    print item
# 输出结果如下：
('m1', 22.0)
('m2', 33.0)
```

#### 5.6、zcount(name, min, max)：

**获取name对应的有序集合中分数 在 [min,max] 之间的个数**

```json
print r.zrange('zset2', 0, -1, withscores=True)
# 输出结果：[('m1', 22.0), ('m2', 33.0)]

print r.zcount('zset2', 22, 33)
# 输出结果：2
```

#### 5.7、zincrby(name, value, amount)：

**自增name对应的有序集合的name对应分数**

```json
# 每次将value='m2' 的分数自增2
r.zincrby('zset2', 'm2', amount=2)
print r.zrange('zset2', 0, -1, withscores=True)
# 输出结果：[('m1', 22.0), ('m2', 39.0(m2所对应的分数每次执行+2))]
```

#### 5.8、zrank(name, value)：

**获取某个值在 name对应的有序集合中的索引（从 0 开始）**

```json
# name=[('m1', 22.0), ('m2', 39.0)], value='m2' 所对应的索引是多少
print r.zrank('zset2', 'm2')
# 输出结果：1
```

#### 5.9、zrem(name, values)：

**删除name对应的有序集合中值是values的成员**

```json
# 删除有序集合'zset2'中 value='m2'
r.zrem('zset2', 'm2')
print r.zrange('zset2', 0, -1, withscores=True)
# 输出结果：[('m1', 22.0)]
```

#### 5.10、zremrangebyscore(name, min, max)：

**根据分数范围删除**

```json
# zset1=[('n1', 11.0), ('n2', 22.0), ('n3', 33.0), ('n4', 44.0), ('n5', 55.0)]
# 删除有序集合'zset1'中, value范围在 33-44间的值
r.zremrangebyscore('zset1', 33, 44)
print r.zrange('zset1', 0, -1, withscores=True)
# 输出结果：[('n1', 11.0), ('n2', 22.0), ('n5', 55.0)]
```

#### 5.11、zscore(name, value)：

**获取name对应有序集合中 value 对应的分数**

```json
# zset=[('n1', 11.0), ('n2', 22.0), ('n5', 55.0)]
# 获取name='zset1' 中value='n1'所对应的分数
print r.zscore('zset1', 'n1')
# 输出结果：11.0
```

## 六、散列(Hash)相关操作：

|     方法     |                   简介                   |
| :----------: | :--------------------------------------: |
|    hset()    | 修改(单个取出)--没有就新增，有的话就修改 |
|   hmset()    |                 批量增加                 |
|   hmget()    |                 批量获取                 |
|  hgetall()   |             取出所有的键值对             |
|    hlen()    |      得到所有键值对的格式 hash长度       |
|   hkeys()    |              得到所有的keys              |
|   hvals()    |             得到所有的value              |
|  hexists()   |             判断成员是否存在             |
|    hdel()    |                删除键值对                |
|  hincrby()   |               自增自减整数               |
| hscan_iter() |            取值查看--分片读取            |

#### 6.1、hset(name, key, value)：

**name对应的hash中设置一个键值对（不存在，则创建；否则，修改）**

* 参数：

  name：redis的name

  key：name对应的hash中的key  

  value：name对应的hash中的value  

  ```json
  # 设置 name='hash1' key='k1' value='v1'
  r.hset('hash1', 'k1', 'v1')
  # 设置 name='hash1' key='k2'  value='v2'
  r.hset('hash1', 'k2', 'v2')
  
  print r.hget('hash1', 'k1')
  # 输出结果：v1
  print r.hget('hash1', 'k2')
  # 输出结果：v2
  ```

#### 6.2、hmset(name, mapping) ：

**在name对应的hash中批量设置键值对**

* 参数：

  name：redis的name

  mapping：字段：如{'k1': 'v1',  'k2': 'v2'}

  ```json
  # 批量设置hash
  create_k = {'k2': 'v2', 'k3': 'v3'}
  r.hmset('hash2', create_k)
  ```

#### 6.3、hmget(name, keys, *args):

**在name对应的hash中获取多个key的值**

* 参数：

  name：redis的name

  keys：要获取key集合，如：['k1', 'k2', 'k3']

  *args：要获取key集合，如：'k1', 'k2', 'k3'

  ```json
  print r.hmget('hash2', 'k2')  # 单个取出'hash2' 输出结果：['v2']
  print r.hmget('hash2', 'k2', 'k3')  # 批量取出'hash2', 输出结果：['v2', 'v3']
  print r.hmget('hash2', ['k2', 'k3'])  # 批量取出'hash2', 输出结果：['v2', 'v3']
  ```

#### 6.4、hgetall(name)：

**获取name对应hash的所有键值**

```json
# 获取name='hash2' 全部key, value
print r.hgetall('hash2')
# 输出结果：{'k3': 'v3', 'k2': 'v2'}
```

#### 6.5、hlen(name) :

**获取name对应的hash中键值对的个数**

```json
# 'hash2'={'k3': 'v3', 'k2': 'v2'}
print r.hlen('hash2')
# 输出结果：2
```

#### 6.6、hkeys(name):

**获取name对应的hash中所有的key的值**

```json
# 仅获取name='hash2' 取出key的值 
print r.hkeys('hash2')
# 输出结果：['k3', 'k2']
```

#### 6.7、hvals(name):

**获取name对应的hash中所有的value的值**

```json
# 仅获取name='hash2', 取出value的值
print r.hvals('hash2')
# 输出结果：['v3', 'v2']
```

#### 6.8、hexists(name, key)：

**检查name对应的hash是否存在当前传入的key**

```json
print r.hexists('hash2', 'k4')
# 输出结果：False

print r.hexists('hash2', 'k2')
# 输出结果：True
```

#### 6.9、hdel(name,*keys):

**将name对应的hash中指定key的键值对删除**

```json
# hash2={'k3': 'v3', 'k2': 'v2'}
# 将name='hahs2' key='k3' 删除
r.hdel('hash2', 'k3')
print r.hgetall('hash2')
# 输出结果：{'k2': 'v2'}
```

#### 6.10、hincrby(name, key, amount=1):

**自增name对应的hash中的指定key的值，不存在则创建key=amount**

* 参数：

  name：redis的name

  key：hash对应的key

  amount：自然数(整数)

  ```json
  r.hincrby('hash2', 'k4', amount=1)
  print r.hgetall('hash2')
  # 输出结果：{'k2': 'v2', 'k4': '2'(+1)}  如果key不存在, value默认就是1
  ```

#### 6.11、hscan_iter(name, match=None, count=None):

**利用yield封装hscan创建生成器，实现分批去redis中获取数据**

* 参数：

  match，匹配指定key，默认None 表示所有的key  

  count，每次分片最少获取个数，默认None表示采用Redis的默认分片个数 

  ```json
  for item in r.hscan_iter('hash2'):
      print item
  # 输出结果：
  # ('k2', 'v2')
  # ('k5', '4')
  # ('k4', '2')
  
  print r.hscan_iter('hash2')  # 生成器内存地址
  # 输出结果：<generator object hscan_iter at 0x000000000389B2D0>
  ```

## 七、其他常规操作：

|   方法   |       简介       |
| :------: | :--------------: |
| delete() |       删除       |
| exists() | 检查名字是否存在 |
| expire() |   设置超时时间   |
| rename() |      重命名      |
|  type()  |     获取类型     |

#### 7.1、delete(*names) :

**根据删除redis中的任意数据类型（string、hash、list、set、有序set）**

```json
r.delete('hash1')  # 删除key为'hash1'的键值对
```

#### 7.2、exists(name)：

**检测redis的name是否存在，存在就是True，False 不存在**

```json
print r.exists('hash1')  # 输出结果：False 说明key 不存在
print r.exists('set1')  # 输出结果：True  说明key存在
```

#### 7.3、expire(name ,time) :

**为某个redis的某个name设置超时时间**

```json
r.lpush('list5', 11, 22)
r.expire('list5', time=3)
print r.lrange('list5', 0, -1)
# 输出结果：['22', '11']
import time
time.sleep(3)
print r.lrange('list5', 0, -1)
# 输出结果：[]
```

#### 7.4、rename(src, dst) ：

**对redis的name重命名**

```json
r.lpush('list5', 11, 22)
print r.rename('list5', 'list-5')
# 输出结果： True 说明重命名成功
```

#### 7.5、type(name) :

**获取name对应值的类型**

```json
print r.type('set1')
# 输出结果：zset

print r.type('zset2')
# 输出结果：zset

print r.type('list-5')
# 输出结果：list
```

#### 7.6、查看所有元素：

```json
print(r.hscan("hash"))
print(r.sscan("set"))
print(r.zscan("zset"))
print(r.getrange("string", 0, -1))
print(r.lrange("list", 0, -1))
print(r.smembers("set3"))
print(r.zrange("zset3", 0, -1))
print(r.hgetall("hash1"))
```

