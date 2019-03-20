#准备

从官网下载合适的安装包。这里以win10为例，一路next即可完成安装。安装完成后，进入这个目录：

```bash
C:\Program Files\MongoDB\Server\4.0\bin\
```

在当前目录打开PowerShell窗口，先启动服务端：

```bash
 .\mongod.exe
```

服务端默认在本地的27017端口运行。

启动客户端：

```bash
 .\mongo.exe
```

将默认连接本地27017端口的服务端，并启动mongo shell交互式命令行窗口。

如果想指定端口或者连接远程的服务，可以在启动时指定其他参数，比如：

```bash
 .\mongo.exe --host www.example.com --port 28015
```

显示当前使用的数据库：

```bash
db
# 默认数据库是test
```

切换数据库：

```bash 
use <database>
```

你也可以切换到一个不存在的数据库。因为当你往其中插入数据时，mongo将自动创建这个数据库（或者集合），比如：

```BASH
> use first
switched to db first
> db.test_coll.insertOne({x:1})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5c52cdf38410c398987c5c0e")
}
```

上述的`insertOne()`操作将同时创建数据库`first`和集合`test_coll`。

# PyMongo

## 连接和指定数据库

PyMongo是mongo的python驱动，安装好之后我们连接MongoDB

方式一：指定主机和端口

```python
import pymongo
client = pymongo.MongoClient(host='localhost', port=27017)
```

方式二：mongodb连接字符串

```python
client = pymongo.MongoClient('mongodb://localhost:27017/')
db = db.test  #  默认数据库是test
```



## 插入数据

### 数据库

默认数据库是test，我们也可以指定其他数据库，比如school

```python
client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.school
# 或者
db = client['school']
```

注意，可以直接连接一个不存在的数据库。mongo会在需要时（比如插入数据），创建这个数据库。

### 集合

mongo中的集合（collection），类似于SQL中的表（table）。指定集合的操作，与指定数据库的方式类似：

```python
collection = db.teacher
# 或者
collection = db['teacher']
```

注意，可以指定一个不存在的集合。mongo会在需要时（比如插入数据），创建这个集合。

### 文档

MongoDB将数据存储为BSON文档（BSON是JSON文档的二进制表示，但是包含更多的[数据类型](https://docs.mongodb.com/manual/reference/bson-types/)）。文档由字段（field）和值（value）组成，具有如下结构：

```json
{
   field1: value1,
   field2: value2,
   field3: value3,
   ...
   fieldN: valueN
}
```

字段名是字符串，具有如下约束条件：

* `_id`是保留字段，用作主键，必须保证唯一且不可以变（因此不能是数组）
* 字段名不可包含`null`字符
* 不要以`$`或`.`开头
* 字段名不可重复

### 点

MongoDB使用`.`点来访问数组或嵌套文档中的元素：

```
{
   ...
   contribs: [ "Turing machine", "Turing test", "Turingery" ],
   ...
}
```

如果要访问contribs字段的第三个元素，使用`contribs.2`

```
{
   ...
   name: { first: "Alan", last: "Turing" },
   contact: { phone: { type: "cell", number: "111-222-3333" } },
   ...
}
```

如果要访问name字段的last字段，使用`name.last`

### 文档的限制

* BSON文档的最大大小是16MB
* 字段的顺序：MongoDB保持写操作时的字段顺序，除了以下情况：
  * `_id`字段总是文档的第一个字段
  * 重新命名字段等更新操作可能影响字段排序
* `_id`字段：
  * 如果插入文档没有指定`_id`字段，MongoDB会自动创建`ObjectId`对象作为该字段的值
  * 创建集合（collection）时，MongoDB默认为该字段创建唯一索引
  * 该字段的值通常是：
    * `ObjectId`
    * 自增数字
    * 代码生成的UUID

### 插入数据

MongoDB的所有写操作，在当个文档上是原子性的。插入文档时，如果集合不存在，MongoDB会自动创建。

#### `db.collection.insert()`

下面我们往teacher这个集合中插入一条数据

```python
# 数据用字典结构
teacher = {
    'name': 'Sven',
    'gender': 'male',
    'major': 'Math'
}

# 插入成功后，返回该条数据的 _id
res = db.teacher.insert(teacher)  # 5c53c429bddaf0ac186d935b
```

插入多条数据

```python
teacher1 = {
    'name': 'Sylvia',
    'gender': 'female',
    'major': 'Deutsch'
}

teacher2 = {
    'name': 'Lolle',
    'gender': 'female',
    'major': 'Geschichte'
}

res = db.teacher.insert([teacher1, teacher2]) 
# 返回_id的列表
# [ObjectId('5c53c628bddaf0b2bc028a73'), ObjectId('5c53c628bddaf0b2bc028a74')]
```

以上我们插入单条和多条都用`insert`方法，更推荐的方式是使用`insert_one()`和`insert_many()`

#### `db.collection.insert_one()`

插入单条数据

```python
res = db.inventory.insert_one(
    {"item": "canvas",
     "qty": 100,
     "tags": ["cotton"],
     "size": {"h": 28, "w": 35.5, "uom": "cm"}})

print(res)  # <pymongo.results.InsertOneResult object at 0x0332EF58>
print(res.inserted_id)  # 5c53c7a8bddaf0208cd858f5
```

#### `db.collection.insert_many()`

插入多条数据

```python
res = db.inventory.insert_many([
    {"item": "journal",
     "qty": 25,
     "tags": ["blank", "red"],
     "size": {"h": 14, "w": 21, "uom": "cm"}},
    {"item": "mat",
     "qty": 85,
     "tags": ["gray"],
     "size": {"h": 27.9, "w": 35.5, "uom": "cm"}},
    {"item": "mousepad",
     "qty": 25,
     "tags": ["gel", "blue"],
     "size": {"h": 19, "w": 22.85, "uom": "cm"}}])

print(res)  
# <pymongo.results.InsertManyResult object at 0x031C3F30>
print(res.inserted_ids)  
# [ObjectId('5c53e48bbddaf04c44880d70'), ObjectId('5c53e48bbddaf04c44880d71'), ObjectId('5c53e48bbddaf04c44880d72')]
```

## 查询数据 

查询使用`db.collection.find()`方法。

首先，准备一些原始数据

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test1

db.inventory.insert_many([
    {"item": "journal",
     "qty": 25,
     "size": {"h": 14, "w": 21, "uom": "cm"},
     "status": "A"},
    {"item": "notebook",
     "qty": 50,
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "status": "A"},
    {"item": "paper",
     "qty": 100,
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "status": "D"},
    {"item": "planner",
     "qty": 75, "size": {"h": 22.85, "w": 30, "uom": "cm"},
     "status": "D"},
    {"item": "postcard",
     "qty": 45,
     "size": {"h": 10, "w": 15.25, "uom": "cm"},
     "status": "A"}])
```

### 查询集合中的所有文档

传入一个空文档作为find的查询条件即可

```python
cursor = db.inventory.find({})
```

以上操作相当于SQL中的：

```mysql
SELECT * FROM inventory
```

`find`方法返回一个`Cursor`实例，遍历它即可访问所有文档：

```python
docs = [doc for doc in cursor]
"""
[{'status': 'A', 'item': 'journal', ...}, ...]
"""
```

### 指定相等条件

如果要指定相等条件，可以给`find`方法传入文档查询条件：

```
{ <field1>: <value1>, ... }
```

比如，下面的示例从inventory集合中查询所有符合status等于D的文档：

```python
cursor = db.inventory.find({'status': 'D'})
```

上述操作相当于SQL中的

```mysql
SELECT * FROM inventory WHERE status = "D"
```

### 使用查询运算符

文档查询条件：

```
{ <field1>: { <operator1>: <value1> }, ... }
```

下面的示例从inventory集合中查询所有status等于A或D的文档：

```python
cursor = db.inventory.find({'status': {'$in': ['A', 'D']}})
```

以上查询相当于SQL中的：

```mysql
SELECT * FROM inventory WHERE status in ("A", "D")
```

查询运算符一览，可查看官方文档：https://docs.mongodb.com/manual/reference/operator/query/#projection-operators

### AND查询

复合查询可以指定多个字段。下面的示例从inventory集合中查询所有status等于A，并且qty小于30的文档：

```python
cursor = db.inventory.find({'status': 'A', 'qty': {'$lt': 30}})
```

以上操作相当于SQL中的：

```mysql
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```

### OR查询

使用`$or`运算符，可以进行逻辑或查询：

```python
cursor = db.inventory.find(
    {'$or': [{'status': 'A'}, {'qty': {'$lt': 30}}]}
)
```

以上操作相当于SQL中的：

```mysql
SELECT * FROM inventory WHERE status = "A" OR qty < 3
```

### AND 和 OR 查询

```python
cursor = db.inventory.find({
    'status': 'A',
    '$or': [{'qty': {'$lt': 30}}, {'item': {'$regex': '^p'}}]
})
```

以上操作相当于SQL中的：

```mysql
SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
```

### 嵌套查询

适用于文档中嵌套文档的查询。下面先填充数据：

```python
import pymongo
from bson.son import SON

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test2

db.inventory.insert_many([
    {"item": "journal",
     "qty": 25,
     "size": SON([("h", 14), ("w", 21), ("uom", "cm")]),
     "status": "A"},
    {"item": "notebook",
     "qty": 50,
     "size": SON([("h", 8.5), ("w", 11), ("uom", "in")]),
     "status": "A"},
    {"item": "paper",
     "qty": 100,
     "size": SON([("h", 8.5), ("w", 11), ("uom", "in")]),
     "status": "D"},
    {"item": "planner",
     "qty": 75,
     "size": SON([("h", 22.85), ("w", 30), ("uom", "cm")]),
     "status": "D"},
    {"item": "postcard",
     "qty": 45,
     "size": SON([("h", 10), ("w", 15.25), ("uom", "cm")]),
     "status": "A"}])
```

注意，子文档键的顺序在这些示例中很重要，因此这里的子文档使用`bson.son.SON`的实例，而不是使用python的字典形式。

插入后，数据结构如下：

```json
[
    {  // 文档
    "_id" : ObjectId("5c540556bddaf0b00803d342"),
    "status" : "A",
    "item" : "journal",
    "size" : {  // 子文档（嵌套文档）
        "h" : 14,
        "w" : 21,
        "uom" : "cm"
    },
    "qty" : 25
    }
]
```

#### 根据子文档进行查询

如果某个字段是一个子文档，要为该字段指定相等条件时，使用如下文档查询条件：`{ <field>: <value> }`，其中`<value>`是作为匹配条件的子文档。

下面的示例从inventory集合中查询size字段等于子文档`{ h: 14, w:21, uom: "cm" }`的文档

```python
cursor = db.inventory.find(
    {'size': SON([("h", 14), ("w", 21), ("uom", "cm")])}
)
"""
[{
	u 'status': u 'A',
	u 'item': u 'journal',
	u '_id': ObjectId('5c540556bddaf0b00803d342'),
	u 'qty': 25,
	u 'size': {
		u 'h': 14,
		u 'uom': u 'cm',
		u 'w': 21
	}
}]
"""
```

注意：

根据子文档进行查询时，子文档必须完全匹配，包括字段的顺序。也就也是说，下面的查询无法匹配任何结果：

```python
cursor = db.inventory.find(
    {"size": SON([("w", 21), ("h", 14), ("uom", "cm")])}
)
# []
```

#### 根据嵌套字段进行查询

通过子文档的字段进行查询，需要使用`.`操作：`field.nestedField`

```python
# 查询 size.uom 等于 in 的文档
cursor = db.inventory.find(
    {'size.uom': 'in'}
)
"""
[{
	u 'status': u 'A',
	u 'item': u 'notebook',
	u '_id': ObjectId('5c540556bddaf0b00803d343'),
	u 'qty': 50,
	u 'size': {
		u 'h': 8.5,
		u 'uom': u 'in',
		u 'w': 11
	}
}, {
	u 'status': u 'D',
	u 'item': u 'paper',
	u '_id': ObjectId('5c540556bddaf0b00803d344'),
	u 'qty': 100,
	u 'size': {
		u 'h': 8.5,
		u 'uom': u 'in',
		u 'w': 11
	}
}]
"""
```

```python
# 查询 size.h 小于 10 的文档
cursor = db.inventory.find(
    {'size.h': {'$lt': 10}}
)
"""
[{
	u 'status': u 'A',
	u 'item': u 'notebook',
	u '_id': ObjectId('5c540556bddaf0b00803d343'),
	u 'qty': 50,
	u 'size': {
		u 'h': 8.5,
		u 'uom': u 'in',
		u 'w': 11
	}
}, {
	u 'status': u 'D',
	u 'item': u 'paper',
	u '_id': ObjectId('5c540556bddaf0b00803d344'),
	u 'qty': 100,
	u 'size': {
		u 'h': 8.5,
		u 'uom': u 'in',
		u 'w': 11
	}
}]
"""
```

```python
# 查询 size.h 小于 15，size.uom 等于 in, 并且status 等于D的文档
cursor = db.inventory.find(
    {'size.h': {'$lt': 15}, 'size.uom': 'in', 'status': 'D'}
)
"""
[{
	u 'status': u 'D',
	u 'item': u 'paper',
	u '_id': ObjectId('5c540556bddaf0b00803d344'),
	u 'qty': 100,
	u 'size': {
		u 'h': 8.5,
		u 'uom': u 'in',
		u 'w': 11
	}
}]
"""
```

### 数组查询

首先准备数据：

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test3

db.inventory.insert_many([
    {"item": "journal",
     "qty": 25,
     "tags": ["blank", "red"],
     "dim_cm": [14, 21]},
    {"item": "notebook",
     "qty": 50,
     "tags": ["red", "blank"],
     "dim_cm": [14, 21]},
    {"item": "paper",
     "qty": 100,
     "tags": ["red", "blank", "plain"],
     "dim_cm": [14, 21]},
    {"item": "planner",
     "qty": 75,
     "tags": ["blank", "red"],
     "dim_cm": [22.85, 30]},
    {"item": "postcard",
     "qty": 45,
     "tags": ["blue"],
     "dim_cm": [10, 15.25]},
    {"item": "mac",
     "qty": 12,
     "tags": ["silver", "golden"],
     "dim_cm": [25, 34]}
])
```

#### 查询整个数组

根据数组指定相等条件时，使用文档查询条件：`{ <field>: <value> }`，其中`<value>`就是要进行匹配的数组本身，包括其中元素的顺序。

```python
# 查询tags字段的值等于['red', 'blank']的文档
cursor = db.inventory.find(
    {'tags': ['red', 'blank']}
)
"""
[{
	u 'item': u 'notebook',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0ff'),
	u 'qty': 50,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank']
}]
"""
```

当然，如果你希望查询数组中包含red和blank元素（不关心元素顺序，或者是否有多余元素）的所有文档，可以使用`$all`操作符：

```python
cursor = db.inventory.find(
    {'tags': {'$all': ['red', 'blank']}}
)
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0fe'),
	u 'qty': 25,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'blank', u 'red']
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0ff'),
	u 'qty': 50,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank']
}, {
	u 'item': u 'paper',
	u '_id': ObjectId('5c54138fbddaf0a1e075f100'),
	u 'qty': 100,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank', u 'plain']
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c54138fbddaf0a1e075f101'),
	u 'qty': 75,
	u 'dim_cm': [22.85, 30],
	u 'tags': [u 'blank', u 'red']
}]
"""
```

#### 查询数组中的某个元素（指定单个查询条件）

查询数组中包含某个元素的文档时，使用`{ <field>: <value> }`，其中`<value>`是数组中的元素

```python
# 查询tags中包含red的文档
cursor = db.inventory.find(
    {'tags': 'red'}
)
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0fe'),
	u 'qty': 25,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'blank', u 'red']
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0ff'),
	u 'qty': 50,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank']
}, {
	u 'item': u 'paper',
	u '_id': ObjectId('5c54138fbddaf0a1e075f100'),
	u 'qty': 100,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank', u 'plain']
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c54138fbddaf0a1e075f101'),
	u 'qty': 75,
	u 'dim_cm': [22.85, 30],
	u 'tags': [u 'blank', u 'red']
}]
"""
```

如果要给数组元素指定多个条件，可以在查询条件中使用查询运算符：`{<array filed>: {<operator1>: <value1>, ...}`，具体如下个部分

#### 为数组内的元素指定多个查询条件

当为数组内的元素指定多个条件时，你可以指定要么单个元素满足这些条件，或者数组内元素的任意组合满足这些条件。

```python
# 查询dim_cm数组中元素的组合满足查询条件：
# 一个元素大于15，另一个元素小于20，或者一个元素满足两种条件的的文档
cursor = db.inventory.find(
    {'dim_cm': {'$gt': 15, '$lt': 20}}
)
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0fe'),
	u 'qty': 25,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'blank', u 'red']
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0ff'),
	u 'qty': 50,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank']
}, {
	u 'item': u 'paper',
	u '_id': ObjectId('5c54138fbddaf0a1e075f100'),
	u 'qty': 100,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank', u 'plain']
}, {
	u 'item': u 'postcard',
	u '_id': ObjectId('5c54138fbddaf0a1e075f102'),
	u 'qty': 45,
	u 'dim_cm': [10, 15.25],
	u 'tags': [u 'blue']
}]
注意："dim_cm": [25, 34]的元素并不会被过滤出来。
"""
```

##### 一个数组元素同时满足多个条件

使用`$elemMatch`运算符，指定多个条件，只要有1个元素同时满足这些条件即可。

```python
# 查询dim_cm数组中，至少有一个大于15且小于20的元素的文档
cursor = db.inventory.find(
    {'dim_cm': {'$elemMatch': {'$gt': 15, '$lt': 20}}}
)
"""
[{
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8ba39bbddaf00ea0365693'),
	u 'qty': 45,
	u 'dim_cm': [10, 15.25],
	u 'tags': [u 'blue']
}]
15.25 满足既大于15，又小于20的条件
"""
```

##### 元素的组合满足复合查询条件

```python
# 查询dim_cm数组中元素的组合满足查询条件：
# 存在某些元素大于15，且某些元素小于20，或者一个元素满足两种条件的的文档
# （若instock下的所有元素的组合都不能满足以上条件，则无法被过滤出来）
# 注意："dim_cm": [25, 34]的元素并不会被过滤出来。
cursor = db.inventory.find(
    {'dim_cm': {'$gt': 15, '$lt': 20}}
)
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0fe'),
	u 'qty': 25,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'blank', u 'red']
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c54138fbddaf0a1e075f0ff'),
	u 'qty': 50,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank']
}, {
	u 'item': u 'paper',
	u '_id': ObjectId('5c54138fbddaf0a1e075f100'),
	u 'qty': 100,
	u 'dim_cm': [14, 21],
	u 'tags': [u 'red', u 'blank', u 'plain']
}, {
	u 'item': u 'postcard',
	u '_id': ObjectId('5c54138fbddaf0a1e075f102'),
	u 'qty': 45,
	u 'dim_cm': [10, 15.25],
	u 'tags': [u 'blue']
}]
"""
```

##### 通过数组的索引查询元素

使用`.`点，根据数组中指定索引的元素进行查询。索引从0开始。

```python
# 根据dim_cm数组的第一个元素，满足大于20，且小于25进行查询
cursor = db.inventory.find(
    {'dim_cm.0': {'$gt': 20, '$lt': 25}}
)
"""
[{
	u 'item': u 'planner',
	u '_id': ObjectId('5c8ba39bbddaf00ea0365692'),
	u 'qty': 75,
	u 'dim_cm': [22.85, 30],
	u 'tags': [u 'blank', u 'red']
}]
"""
```

### 包含子文档的数组

准备数据：

```python
import pymongo
from bson.son import SON

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test4

db.inventory.insert_many([
    {"item": "journal",
     "instock": [
         SON([("warehouse", "A"), ("qty", 5)]),
         SON([("warehouse", "C"), ("qty", 15)])]},
    {"item": "notebook",
     "instock": [
         SON([("warehouse", "C"), ("qty", 5)])]},
    {"item": "paper",
     "instock": [
         SON([("warehouse", "A"), ("qty", 60)]),
         SON([("warehouse", "B"), ("qty", 15)])]},
    {"item": "planner",
     "instock": [
         SON([("warehouse", "A"), ("qty", 40)]),
         SON([("warehouse", "B"), ("qty", 5)])]},
    {"item": "postcard",
     "instock": [
         SON([("warehouse", "B"), ("qty", 15)]),
         SON([("warehouse", "C"), ("qty", 35)])]}])
```

#### 根据子文档进行匹配

```python
# instock数组的某个元素能匹配指定的文档
cursor = db.inventory.find({
    'instock': SON([('warehouse', 'A'), ('qty', 5)])
})

"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e89'),
	u 'instock': [{   // 该元素能够匹配指定的文档条件
		u 'warehouse': u 'A',
		u 'qty': 5
	}, {
		u 'warehouse': u 'C',
		u 'qty': 15
	}]
}]
"""
```

根据文档进行匹配是严格匹配，包括文档的顺序，因此如下查询无法匹配任何结果：

```python
cursor = db.inventory.find({
    'instock': SON([('qty', 5), ('warehouse', 'A')])
})
```

#### 对子文档的 某个字段指定单个查询条件

##### 对所有子文档内的某个字段指定一个查询条件

```python
# 查询instock数组内至少包含一个子文档，其满足qty字段小于等于10的所有文档
cursor = db.inventory.find({
    'instock.qty': {'$lte': 10}
})
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e89'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 5
	}, {
		u 'warehouse': u 'C',
		u 'qty': 15
	}]
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8a'),
	u 'instock': [{
		u 'warehouse': u 'C',
		u 'qty': 5
	}]
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}, {
		u 'warehouse': u 'B',
		u 'qty': 5
	}]
}]
"""
```

##### 使用数组索引来指定子文档的某个字段进行查询

```python
# 查询instock数组的第一个子文档的qty字段满足大于20的所有文档
cursor = db.inventory.find({
    'instock.0.qty': {'$gt': 20}
})
"""
[{
	u 'item': u 'paper',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8b'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 60
	}, {
		u 'warehouse': u 'B',
		u 'qty': 15
	}]
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}, {
		u 'warehouse': u 'B',
		u 'qty': 5
	}]
}]
"""
```

#### 对子文档指定多个字段查询条件

当对子文档指定多个字段查询条件时，你可以指定要么单个子文档满足这些条件，或者任意子文档的组合满足条件（类似上面的数组内元素指定多个条件）

##### 单个子文档满足多个字段的查询条件

使用`$elemMatch`运算符对包含子文档的数组指定多个条件，至少有一个子文档能满足这些条件即可

```python
# 查询instock数组至少包含一个子文档同时满足qty等于5，warehouse等于4的所有文档
cursor = db.inventory.find({
    'instock': {'$elemMatch': {'qty': 5, 'warehouse': 'A'}}
})

"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e89'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 5
	}, {
		u 'warehouse': u 'C',
		u 'qty': 15
	}]
}]
"""
```

```python
# 查询instock数组至少包含一个子文档，其qty大于30，小于等于50的所有文档
cursor = db.inventory.find(
    {'instock': {'$elemMatch': {'qty': {'$gt': 30, '$lte': 50}}}}
)
"""
[{
	u 'item': u 'planner',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}, {
		u 'warehouse': u 'B',
		u 'qty': 5
	}]
}, {
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8d'),
	u 'instock': [{
		u 'warehouse': u 'B',
		u 'qty': 15
	}, {
		u 'warehouse': u 'C',
		u 'qty': 35
	}]
}]
"""
```

##### 子文档组合满足复合查询条件

如果复合查询不使用`$elemMatch`运算符，那么查询将选择那些数组包含任何能满足条件的组合。比如：

```python
# 查询instock下的某些子文档的qty字段大于30，且某些子文档的qty字段小于等于50，（或同时满足）的所有文档
# （若instock下的所有文档的组合都不能满足以上条件，则无法被过滤出来）
cursor = db.inventory.find(
    {'instock.qty': {'$gt': 30, '$lte': 50}}
)
"""
[{
	u 'item': u 'paper',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8b'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 60
	}, {
		u 'warehouse': u 'B',
		u 'qty': 15
	}]
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}, {
		u 'warehouse': u 'B',
		u 'qty': 5
	}]
}, {
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8d'),
	u 'instock': [{
		u 'warehouse': u 'B',
		u 'qty': 15
	}, {
		u 'warehouse': u 'C',
		u 'qty': 35
	}]
}]
"""
```

```python
# 查询instock下的某些子文档的qty字段等于5，且某些子文档的warehouse等于A，（或同时满足）的所有文档
cursor = db.inventory.find({
    'instock.qty': 5, 'instock.warehouse': 'A'
})
"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e89'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 5
	}, {
		u 'warehouse': u 'C',
		u 'qty': 15
	}]
}, {
	u 'item': u 'planner',
	u '_id': ObjectId('5c8c5f4abddaf049404c5e8c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}, {
		u 'warehouse': u 'B',
		u 'qty': 5
	}]
}]
"""
```

### 返回字段

默认情况下，MongoDB的查询返回匹配文档的所有字段，如果想限制返回数据的大小，可以使用projection文档（投影文档）来指定返回的字段。

准备数据：

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test5

db.inventory.insert_many([
    {"item": "journal",
     "status": "A",
     "size": {"h": 14, "w": 21, "uom": "cm"},
     "instock": [{"warehouse": "A", "qty": 5}]},
    {"item": "notebook",
     "status": "A",
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "instock": [{"warehouse": "C", "qty": 5}]},
    {"item": "paper",
     "status": "D",
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "instock": [{"warehouse": "A", "qty": 60}]},
    {"item": "planner",
     "status": "D",
     "size": {"h": 22.85, "w": 30, "uom": "cm"},
     "instock": [{"warehouse": "A", "qty": 40}]},
    {"item": "postcard",
     "status": "A",
     "size": {"h": 10, "w": 15.25, "uom": "cm"},
     "instock": [
         {"warehouse": "B", "qty": 15},
         {"warehouse": "C", "qty": 35}]}])
```

我们看以下查询：

```python
cursor = db.inventory.find({"status": "A"})
```

该查询相当于SQL的：

```mysql
SELECT * FROM inventory WHERE status = 'A'
```

#### 只返回指定字段和 `_id` 字段

在投影文档中，将需要的字段设置为1，就可以指定查询后返回的字段。比如：

```python
cursor = db.inventory.find(  # find的第二个参数是投影文档
    {'status': 'A'}, {'item': 1, 'status': 1}
)

"""
[{
	u 'status': u 'A',
	u 'item': u 'journal',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd49')
}, {
	u 'status': u 'A',
	u 'item': u 'notebook',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4a')
}, {
	u 'status': u 'A',
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4d')
}]
"""
```

上述查询相当于SQL:

```mysql
SELECT _id, item, status from inventory WHERE status = "A"
```

#### 丢弃 `_id` 字段

`_id`字段会默认返回，如果不需要，可以在投影文档中指定 `_id: 0`

```python
cursor = db.inventory.find(
    {'status': 'A'}, {'item': 1, 'status': 1, '_id': 0}
)
```

#### 返回所有字段，排除某些字段

将要排除的字段在投影文档中设置为0即可

```python
cursor = db.inventory.find(
    {'status': 'A'}, {'status': 0, 'instock': 0}
)

"""
[{
	u 'item': u 'journal',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd49'),
	u 'size': {
		u 'h': 14,
		u 'w': 21,
		u 'uom': u 'cm'
	}
}, {
	u 'item': u 'notebook',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4a'),
	u 'size': {
		u 'h': 8.5,
		u 'w': 11,
		u 'uom': u 'in'
	}
}, {
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4d'),
	u 'size': {
		u 'h': 10,
		u 'w': 15.25,
		u 'uom': u 'cm'
	}
}]
"""
```

**注意**：

除了`_id`字段，projection中不能既有指定字段，又有排除字段。下面的查询会报错：

```python
cursor = db.inventory.find(
    {'status': 'A'}, {'status': 0, 'instock': 0, 'item': 1}
)

"""
pymongo.errors.OperationFailure: Projection cannot have a mix of inclusion and exclusion.
"""
```

#### 返回子文档中的指定字段

通过`.`点标记指定嵌套字段并设置为1即可，比如

```python
# 查询所有文档，并且只返回size子文档中的uom字段
cursor = db.inventory.find(
    {}, {'item': 1, 'status': 1, 'size.uom': 1}
)

"""
[{
	u 'status': u 'A',
	u 'item': u 'journal',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd49'),
	u 'size': {
		u 'uom': u 'cm'
	}
}, 
// ...
{
	u 'status': u 'A',
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4d'),
	u 'size': {
		u 'uom': u 'cm'
	}
}]
"""
```

#### 丢弃子文档中的指定字段

通过`.`点标记指定嵌套字段并设置为0即可，比如

```python
# 查询satus等于D的所有文档，且子文档不返回uom字段
cursor = db.inventory.find(
    {'status': 'D'}, {'size.uom': 0}
)

"""
[{
	u 'status': u 'D',
	u 'item': u 'paper',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4b'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 60
	}],
	u 'size': {
		u 'h': 8.5,
		u 'w': 11
	}
}, {
	u 'status': u 'D',
	u 'item': u 'planner',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4c'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 40
	}],
	u 'size': {
		u 'h': 22.85,
		u 'w': 30
	}
}]
"""
```

#### 投射数组内的子文档指定字段

使用`.`标记来投影数组内子文档的字段即可，比如

```python
cursor = db.inventory.find(
    {'status': 'D'}, {'item': 1, 'status': 1, 'instock.qty': 1}
)

"""
[{
	u 'status': u 'D',
	u 'item': u 'paper',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4b'),
	u 'instock': [{
		u 'qty': 60
	}]
}, {
	u 'status': u 'D',
	u 'item': u 'planner',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4c'),
	u 'instock': [{
		u 'qty': 40
	}]
}]
"""
```

#### 投射指定的数组元素

MongoDB为包含数组的字段提供了一些运算符，来操作数组：

`$elemMatch, $slice, $`

下面使用切片运算符`$slice`来返回数组中的最后一个元素：

```python
cursor = db.inventory.find(
    {'status': 'A'},
    {'item': 1, 'status': 1, 'instock': {'$slice': -1}}
)

"""
[{
	u 'status': u 'A',
	u 'item': u 'journal',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd49'),
	u 'instock': [{
		u 'warehouse': u 'A',
		u 'qty': 5
	}]
}, {
	u 'status': u 'A',
	u 'item': u 'notebook',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4a'),
	u 'instock': [{
		u 'warehouse': u 'C',
		u 'qty': 5
	}]
}, {
	u 'status': u 'A',
	u 'item': u 'postcard',
	u '_id': ObjectId('5c8ca21dbddaf046e0efdd4d'),
	u 'instock': [{
		u 'warehouse': u 'C',
		u 'qty': 35
	}]
}]
"""
```

### 查询Null或缺失字段

MongoDB中的不同查询运算符对null值的处理也不同。

准备数据：

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost:27017/')
db = client.test6

# 插入两个文档，一个item为空值，一个item字段缺失
db.inventory.insert_many([{"_id": 1, "item": None}, {"_id": 2}])
```

#### 相等性过滤

`{'item': None}`查询将匹配要么item字段为空值，要么不包含item字段的文档：

```python
cursor = db.inventory.find(
    {'item': None}
)

"""
[{
	u 'item': None,
	u '_id': 1
}, {
	u '_id': 2
}]
"""
```

#### 类型检查

`{'item': {'$type': 10}}`查询只匹配item为空值的文档。空值的类型码是10

```python
cursor = db.inventory.find(
    {'item': {'$type': 10}}
)

"""
[{u'item': None, u'_id': 1}]
"""
```

#### 存在检查

`{'item': {'$exists': False}}`查询只匹配不包含item字段的文档

```python
cursor = db.inventory.find(
    {'item': {'$exists': False}}
)

"""
[{u'_id': 2}]
"""
```



## 更新文档

更新文档主要使用如下几个方法：

- `update_one()`
- `update_many()`
- `replace_one()`

准备数据：

```python
db.inventory.insert_many([
    {"item": "canvas",
     "qty": 100,
     "size": {"h": 28, "w": 35.5, "uom": "cm"},
     "status": "A"},
    {"item": "journal",
     "qty": 25,
     "size": {"h": 14, "w": 21, "uom": "cm"},
     "status": "A"},
    {"item": "mat",
     "qty": 85,
     "size": {"h": 27.9, "w": 35.5, "uom": "cm"},
     "status": "A"},
    {"item": "mousepad",
     "qty": 25,
     "size": {"h": 19, "w": 22.85, "uom": "cm"},
     "status": "P"},
    {"item": "notebook",
     "qty": 50,
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "status": "P"},
    {"item": "paper",
     "qty": 100,
     "size": {"h": 8.5, "w": 11, "uom": "in"},
     "status": "D"},
    {"item": "planner",
     "qty": 75,
     "size": {"h": 22.85, "w": 30, "uom": "cm"},
     "status": "D"},
    {"item": "postcard",
     "qty": 45,
     "size": {"h": 10, "w": 15.25, "uom": "cm"},
     "status": "A"},
    {"item": "sketchbook",
     "qty": 80,
     "size": {"h": 14, "w": 21, "uom": "cm"},
     "status": "A"},
    {"item": "sketch pad",
     "qty": 95,
     "size": {"h": 22.85, "w": 30.5, "uom": "cm"},
     "status": "A"}])
```

### 更新表中的文档

如果要修改字段的值，可以使用更新运算符，比如`$set`，基本格式如下：

```python
{
    <更新运算符>: {<字段1>: <值1>, ...},
    <更新运算符>: {<字段2>: <值2>, ...},
}
```

#### 更新单个文档：`update_one()`

```python
res = db.inventory.update_one(
    {'item': 'paper'},  # filter
    {                   # update
        '$set': {
            'size.uom': 'cm',
            'status': 'P'
        },
        '$currentDate': {
            'lastModified': True  # 新增lastModified字段，值是当前时间
        }
    }
)


print(res)  # <pymongo.results.UpdateResult object at 0x0000024A22FFB3C8>
print(res.matched_count)  # 1
print(res.modified_count)  # 1

cursor = db.inventory.find({'item': 'paper'})
docs = [doc for doc in cursor]
print(docs)
"""
[{
	'_id': ObjectId('5c8f8dcfbddaf03aa4b6a409'),
	'item': 'paper',
	'qty': 100,
	'size': {
		'h': 8.5,
		'w': 11,
		'uom': 'cm'
	},
	'status': 'P',
	'lastModified': datetime.datetime(2019, 3, 18, 12, 36, 45, 391000)
}]
"""
```

#### 更新多个文档：`update_many()`

```python
# 更新qty小于50的文档
res = db.inventory.update_many(
    {'qty': {'$lt': 50}},  # filter
    {  # update
        '$set': {'size.uom': 'in', 'status': 'P'},
        '$currentDate': {'lastModified': True}
     }
)

print(res)  # <pymongo.results.UpdateResult object at 0x0000014249CF7488>
print(res.matched_count)  # 3
print(res.modified_count)  # 3

cursor = db.inventory.find({'qty': {'$lt': 50}})
docs = [doc for doc in cursor]
print(docs)
"""
[{
	'_id': ObjectId('5c8f8dcfbddaf03aa4b6a405'),
	'item': 'journal',
	'qty': 25,
	'size': {
		'h': 14,
		'w': 21,
		'uom': 'in'
	},
	'status': 'P',
	'lastModified': datetime.datetime(2019, 3, 19, 12, 26, 33, 410000)
}, {
	'_id': ObjectId('5c8f8dcfbddaf03aa4b6a407'),
	'item': 'mousepad',
	'qty': 25,
	'size': {
		'h': 19,
		'w': 22.85,
		'uom': 'in'
	},
	'status': 'P',
	'lastModified': datetime.datetime(2019, 3, 19, 12, 26, 33, 413000)
}, {
	'_id': ObjectId('5c8f8dcfbddaf03aa4b6a40b'),
	'item': 'postcard',
	'qty': 45,
	'size': {
		'h': 10,
		'w': 15.25,
		'uom': 'in'
	},
	'status': 'P',
	'lastModified': datetime.datetime(2019, 3, 19, 12, 26, 33, 414000)
}]
"""
```

### 替换文档：`replace_one()`

如果要替换整个文档的内容（除了`_id`字段），只需传入一个新的文档，作为`replace_one（）`的第二个参数即可。替换的文档只能包含键值对，不能巴博涵其他表达式，比如更新运算符。替换的文档可以和原文的的字段不同，由于`_id`不可变，你可以省略`_id`字段（当然，如果你非要在替换文档中包含`_id`字段，那么必须给出与原文档一样的值）

```python
# 替换满足筛选条件的第一个文档
res = db.inventory.replace_one(
    {'item': 'paper'},  # filter
    {'item': 'paper',   # replacement_doc
     'instock': [
         {"warehouse": "A", "qty": 60},
         {"warehouse": "B", "qty": 40}
     ]}
)

print(res)  # <pymongo.results.UpdateResult object at 0x000001AD62170448>
print(res.matched_count)  # 1
print(res.modified_count)  # 1

cursor = db.inventory.find({'item': 'paper'})
docs = [doc for doc in cursor]
print(docs)
"""
[{
	'_id': ObjectId('5c8f8dcfbddaf03aa4b6a409'),
	'item': 'paper',
	'instock': [{
		'warehouse': 'A',
		'qty': 60
	}, {
		'warehouse': 'B',
		'qty': 40
	}]
}]
"""
```

### 行为

#### 原子性

MongoDB的所有写操作在单个文档上都是原子性的。

#### `_id`字段

一旦设置，该字段的值不可修改，也不能被替换。

#### 字段顺序

MongoDB保持写操作中文档的顺序，除了以下情况：

* `_id`字段总是文档中的第一个字段
* 如果更新包含对字段进行命名的操作，可能导致文档的字段重新排序

#### `upsert`选项

调用`update_one(), update_many(), replace_one()`这些方法时，如果指定`upsert=True`，那么当匹配的文档不存在时，将创建新的文档并插入。

## 运算符列表：

|      名称      |             描述             |
| :------------: | :--------------------------: |
|     `$or`      |            逻辑或            |
|     `$eq`      |             等于             |
|     `$gt`      |             大于             |
|     `$gte`     |           大于等于           |
|     `$in`      |           在数组中           |
|     `$lt`      |             小于             |
|     `$lte`     |           小于等于           |
|     `$ne`      |            不等于            |
|     `$nin`     |          不在数组中          |
|  `$elemMatch`  |         数组元素满足         |
|     `$all`     |         数组包含元素         |
|    `$slice`    |           数组切片           |
|    `$type`     |        字段值类型检查        |
|   `$exists`    |       字段是否存在判断       |
|     `$set`     |         更新字段的值         |
| `$currentDate` | 设定某字段的值为当前日期时间 |



# mongoengine

待完善。









## 