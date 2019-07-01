# MongoDB数据建模介绍

数据建模需要在满足应用需求、数据库引擎的性能特征、以及数据检索模式之间取得平衡。在设计数据模型时，请始终考虑应用程序对数据的使用场景（比如，查询，更新，和数据处理）以及数据本身的结构。

## 灵活的模型

不同于SQL数据库，插入数据前必须声明表的模型。默认情况下，MongoDB的集合不要求其文档具有相同的模型：

* 单个集合内的文档不必拥有相同的字段，并且字段的数据类型也可以不同。
* 如果要更改集合内的文档结构（比如增加新字段，去除字段，或修改字段的值），直接更改即可。

这种灵活性有助于将文档映射到实体或对象上。集合内的每个文档都可以单独匹配其表示实体的数据字段。

不过在实际应用中，集合内的文档通常具有类似的结构，并且你也可以在执行更新和插入操作时，使用集合的文档验证规则。具体查看：https://docs.mongodb.com/manual/core/schema-validation/

## 文档结构

为MongoDB应用设计数据模型时，关键是文档结构以及应用程序如何表示数据之间的关系。MongoDB允许在单个文档中嵌入关联数据。

### 子文档

子文档可以存储关联数据，并嵌在文档的某个字段或者数组中。这些反范式的数据模型可以方便应用程序在单次数据库操作中检索和操作相关数据。在很多场景下，这种反范式的数据模型都是非常理想的。

```json
// a user document
{
    _id: <ObjectId1>,
    username: 'ayhan',
    gender: 'male',
    age: 18,
    extra: {
        email: 'abc@example.com',
        school: 'xxx univeirsity',
        hobby: 'reading',
	}
}
```

嵌入式数据模型允许应用在一条记录中存储关联的信息片段。这样的好处是，应用在完成常见的操作时，可以发出更少的查询和更新。

通常在以下情况下使用嵌入式数据模型：

- 实体之间存在“包含”关系，比如一对一。
- 实体之间存在“一对多”关系，在这种关系中，“多”的文档常常在“一”的文档的上下文中出现或被查看（反之亦然）。

通常，嵌入式能为读操作提供更好的性能，以及在单次数据库操作中请求和检索关联数据的能力。这也使得在单个原子性写操作中更新关联的数据成为可能。要访问嵌入的子文档，使用`.`操作即可。

包含子文档的文档大小也不得超过BSON文档尺寸的最大值，也就是16MB。

### 引用

通过文档间的链接（或引用），引用用来存储着数据间的关系。个人理解，类似于SQL中的外键。应用可以解析这些引用来访问相关的数据。广义上来讲，这些是范式化的数据模型。

```json
// a user document
{
    _id: <ObjectId1>,
    username: 'ayhan',
    gender: 'male',
    age: 18
}

// a extra document
{
    _id: <ObjectId2>,
    email: 'abc@example.com',
    school: 'xxx univeirsity',
    hobby: 'reading',
    user_id: <ObjectId1>  // 通过user的主键，引用user
}
```

通常在以下情况下使用范式化的数据模型：

* 嵌入式会导致数据冗余，但又不能提供足够的性能优势时
* 描述更复杂的多对多的关系时
* 模拟大型的分层数据集时（树形结构）

引用比起嵌入式能提供更大的灵活性。但是客户端应用必须发起后续查询来解析引用，也就是说，范式化的数据模型需要更多的查询请求。

## 写操作的原子性

### 单文档原子性

在MongoDB中，写操作在单个文档上保证原子性，即使这个操作修改了单个文档的多个子文档。如果单次写操作修改了多个文档（比如，`db.collection.update_many()`），那么对每个文档的修改是原子性的，但是这整个操作并不是原子性的。支持在单个文档中嵌入子文档的这种反范式化数据结构，比起跨多个文档和集合的范式化方式，更有助于原子性操作。

从4.0版本开始，对于需要多文档读写保证原子性的场景，MongoDB为副本集提供了多文档事务支持。具体查看：https://docs.mongodb.com/manual/core/transactions/，

### 多文档事务

>注意：
>
>在多数情况下，多文档事务会导致更多的性能开销。尽管有多文档事务支持，但也不应该以此来取代高效的模型设计。对于很多场景，反范式化的数据模型（子文档和数组）仍然是最好的选择。也就是说，合适的数据建模将最大限度地减少对多文档事务的需求。

## 数据的使用和性能

在设计数据模型时，请考虑应用程序如何使用你的数据库。例如，如果你的应用只使用最近插入的文档，请考虑使用[Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/)。或者你的应用主要是对集合的读操作，那么对常见的查询添加索引有助于提高性能。

更多信息可查看：[操作因素和数据模型](https://docs.mongodb.com/manual/core/data-model-operations/)

## 分片

MongoDB使用分片提供水平伸缩。这些集群支持可以支持大型数据集和超高的吞吐量的操作。分片允许用户对数据库内的集合做分区，然后在多个实例或分片间分发文档。

MongoDB使用分片键（shard key）在分片集合中分发数据和应用程序流量。选择合适的分片键对性能影响重大，并且可以启用或阻止查询隔离并增加写入容量。仔细考虑要用作分片键的字段非常重要。

见[分片](https://docs.mongodb.com/manual/sharding/)和 [分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/)以获取更多信息。

## 索引

使用索引可以提高常见查询的性能。请对查询中经常出现的字段创建索引，为返回排序结果的所有操作创建索引。MongoDB会自动为`_id`字段创建索引。创建索引时，需要考虑一下索引行为：

* 每个索引至少需要8kb的数据空间
* 增加索引会对写操作产生负面的性能影响。对于具有高**写读比**的集合，索引是昂贵的开销，因为每次插入都需要更新任何索引
* 具有高**读写比**的集合通常会从索引中受益
* 处于活动状态时，每个索引都会占据磁盘空间和内存。这种用量可能会非常重要，并且出于容量规划，也应该被监控起来，尤其要关注工作集的大小。

有关[索引](https://docs.mongodb.com/manual/applications/indexes/)以及[分析查询性能的](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)详细信息，请参阅[索引策略](https://docs.mongodb.com/manual/applications/indexes/)。此外，MongoDB [数据库分析器（database profiler）](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)可以帮助定位低效的查询。

## 大量的集合

在某些情况下，你可以选择将信息存储在多个不同的集合中，而不是单个集合。通常情况下，大量的集合并不会导致性能损失，并且可能对性能很有帮助。对于高吞吐量的批处理，不同的集合非常重要。在使用具有大量集合的模型时，请考虑以下行为：

* 每个集合都有几千字节的最小开销

* 每个索引（包括`_id`索引），都需要至少8kb的数据空间

* 每个数据库都有一个命名空间文件（namespace file，比如<database>.ns），用来存储该数据库所有的元数据，并且每个索引和集合在命名空间中有自己的条目。MongoDB对命名空间的大小有限制（[更多参考](https://docs.mongodb.com/manual/reference/limits/#Size-of-Namespace-File)）

* 使用`mmapv1`存储引擎的MongoDB 对命名空间的[数量有限制](https://docs.mongodb.com/manual/reference/limits/#Number-of-Namespaces)。如果你想知道当前当前命名空间的数量，可以在MongoDB shell中运行如下命令：

  ```javascript
  db.systems.namespaces.count()
  ```

  命名空间数量的限制取决于`<database>.ns` 大小。命名空间文件默认为16 MB。如果要更改新命名空间的大小，可以在启动服务增加选项：`--nssize <new size MB>`. 对于现有的数据库，以`--nssize`选项启动后，在MongoDB shell执行`db.repairDatabase()`[命令](https://docs.mongodb.com/manual/reference/command/repairDatabase/#dbcmd.repairDatabase)。

## 集合包含大量的小文档

出于性能原因，如果您的集合包含大量小文档，则应考虑嵌入。如果您可以通过某种逻辑关系对这些小文档进行分组，并且经常通过此分组检索这些文档，则可以考虑将小文档“汇总”为包含嵌入文档数组的较大文档。

将这些小文档“汇总”为逻辑分组意味着在检索一组文档时，查询是顺序读取的，且只需较少的随机磁盘访问。此外，“汇总”文档并将公共字段移动到较大的文档有益于这些字段的索引。只需要更少的公共字段的副本，索引时更少的相关键条目。有关[索引](https://docs.mongodb.com/manual/indexes/)的更多信息，请参阅 索引。

但是，如果您通常只需要检索分组内文档的子集，那么“汇总”文档可能无法提供更好的性能。此外，如果小的单独文档代表数据的自然模型，则应该维护该模型。

## 小文档的存储优化

每个MongoDB文档都包含一定的开销。这种开销通常是无关紧要的，但如果所有文档都只是几个字节，比如集合中的文档只有一个或两个字段的情况下，则另当别论。

请考虑以下建议和策略，以优化这些集合的存储利用率：

- `_id`明确使用该字段。

  MongoDB客户端自动为每个文档添加一个`_id`字段，并为该 字段生成唯一的12位字节的[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#term-objectid)`。此外，MongoDB总是索引`_id`字段。对于较小的文档，这可能占用大量空间。

  为了优化存储使用，用户可以在插入文档时显式指定该字段的值，只要能满足主键约束即可。

- 使用较短的字段名称。

  >注意
  >
  >缩短字段名称不仅降低表达能力，也不会为较大的文档提供的明显的好处，因为对于较大的文档，文档开销并没那么重要。此外，较短的字段名称不会减小索引的大小，因为索引具有预定义的结构。
  >
  >通常，没有必要使用短字段名称。

  MongoDB将所有字段名称存储在每个文档中。对于大多数文档，这只占文档使用空间的一小部分；但是，对于小型文档，这可能占用更多的空间比例。考虑一组类似于以下内容的小文档：

  ```json
  {  last_name  ： “Smith” ， best_score ： 3.9  }
  ```

  如果对字段名进行如下缩短，那么每个文档可以节约9个字节。

  ```json
  {  lname  ： “Smith” ， 得分 ： 3.9  }
  ```

- 嵌入文档。

  在某些情况下，您可能希望将文档嵌入到其他文档中，并节省每个文档的开销。请参见 [集合包含大量小文档](https://docs.mongodb.com/manual/core/data-model-operations/#faq-developers-embed-documents)。

## 数据生命周期管理

数据建模时应考虑数据的生命周期管理。

有[生存时间或TTL特性](https://docs.mongodb.com/manual/tutorial/expire-data/)的集合在一段时间后，其文档会过期。如果您的应用需要某些数据在数据库中保留一段有限的时间，请考虑使用TTL特性。

此外，如果您的应用仅使用最近插入的文档，请考虑[上限集合（Capped Collections）](https://docs.mongodb.com/manual/core/capped-collections/)，可对插入的文档进行先入先出的管理，有效地支持插入和读取基于插入顺序的文档操作。

## 文档的增长和MMAPv1

某些更新，比如往数组插入元素，或者增加字段，会增加文件的大小。

对于已经弃用的MMAPv1文档引擎，如果文档大小超出了该文档的分配空间，MongoDB会在磁盘上重新分配该文档。使用废弃的MMAPv1引擎时，出于对文档增长的考虑，将会影响对范式化化或反范式化数据的决策。

在已创建的文档中嵌入数据会导致文档的增长。使用已弃用的MMAPv1存储引擎，文档增长会影响写入性能并导致数据碎片化。从版本3.0.0开始，MongoDB使用Power of 2 Sized Allocations取代MMAPv1的默认分配策略，以最大限度地减少数据碎片的可能性。有关详细信息，请参阅 [Power of 2 Sized Allocations](https://docs.mongodb.com/manual/core/mmapv1/#power-of-2-allocation) 。

## 其他参考资源

* [Thinking in Documents Part 1 (Blog Post)](https://www.mongodb.com/blog/post/thinking-documents-part-1?jmp=docs)

# 数据模型示例和模式

## 文档间的模型关系

###  基于子文档的一对一

使用子文档来描述关联数据间的一对一关系。下面以顾客和地址为例子，每个顾客对应一个地址。

如果用范式化的数据模型，address文档包含对customer文档的引用：

```json
// customer 文档
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address 文档
{
   customer_id: "joe", 
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

如果要经常通过customer信息来获取address信息，在引用的情况下，你的应用需要发出多个查询来解析引用。但是如果使用子文档，应用通过一次查询就可以检索到所有信息：

```json
{
   _id: "joe",
   name: "Joe Bookreader",
   address: { // address 作为子文档
              street: "123 Fake Street",
              city: "Faketon",
              state: "MA",
              zip: "12345"
            }
}
```

这个示例说明了，如果要在一个数据实体的上下文中查看另一个数据实体，子文档对于引用的优势。

### 基于子文档的一对多

使用子文档来描述关联数据间的一对多关系。下面以顾客和地址为例子，每个顾客对应多个地址。

使用范式化数据模型，多个address文档包含对customer文档的引用：

```json
{
   _id: "joe",
   name: "Joe Bookreader"
}

{
   customer_id: "joe",
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}

{
   customer_id: "joe",
   street: "1 Some Other Street",
   city: "Boston",
   state: "MA",
   zip: "12345"
}
```

如果要经常通过customer信息来获取其address信息，在引用的情况下，你的应用需要发出多个查询来解析引用。更好的选择是把address数据实体嵌入customer的数组字段中，这样应用通过一次查询就可以检索到所有信息：

```json
{
   _id: "joe",
   name: "Joe Bookreader",
   addresses: [
                {
                  street: "123 Fake Street",
                  city: "Faketon",
                  state: "MA",
                  zip: "12345"
                },
                {
                  street: "1 Some Other Street",
                  city: "Boston",
                  state: "MA",
                  zip: "12345"
                }
              ]
 }
```

#### 适用情况

子文档的一对多，这里的多通常只有几个或者几十个，一般不超过3位数，是比较合适的。如果特别多，考虑使用接下来的引用。

### 基于引用的一对多

使用文档间的引用来描述关联数据间的一对多关系。下面以出版社和书为例子，每个出版社可以对应多本书。

如果使用子文档，将publisher文档嵌入book文档中，会导致publisher信息重复：

```json
{
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",
   publisher: {  // publisher
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }
}

{
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher: {  // publisher 重复
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }

}
```

为了避免这种重复，可以使用引用，将publisher的信息单独保存到一个集合中。

使用引用时，关系的增长决定在哪边存储引用关系。如果每个publisher发布的book很少且有限，那么在publisher中包含对book的引用是合适的。但是，如果publisher发布的book数量巨大且没有限制，这样的数据模型将导致不断地变化，数组不断增长，就像下面这样：

```json
{
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA",
   books: [123456789, 234567890, ...]  // 通过数组字段，存储对book的引用，但是数组将不断变大

}

{
    _id: 123456789,
    title: "MongoDB: The Definitive Guide",
    author: [ "Kristina Chodorow", "Mike Dirolf" ],
    published_date: ISODate("2010-09-24"),
    pages: 216,
    language: "English"
}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English"
}
```

为了避免上述这种情况，应当在book中包含对publisher的引用：

```json
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",

   publisher_id: "oreilly"  // book中引用publisher

}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",

   publisher_id: "oreilly" // book中引用publisher

}
```

#### 适用情况

通常来说，如果书不会超过数千本，将其书的实体放在出版社的数组中引用是合适的。但是如果更多甚至没有上限，那么应该考虑将出版社放在书的实体中引用。对于前者，当然也可以同时在书的实体中包含对出版社的引用，这种**双向关联**可以提供灵活的检索。

#### 利用冗余提高性能

另外我们也可以结合使用范式化化和反范式化模型，比如：

```json
{
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA",
   books: [
       {id: 123456789, title: '50 Tips and Tricks for MongoDB Developer'}  // 存储对书籍的引用，额外冗余书籍的标题
       {id: 234567890, title: '50 Tips and Tricks for MongoDB Developer'}
   ]

}
```

这样，如果要显示某个publisher出版的所有书名，就需不要额外的查询或者连表操作了。

结合使用反范式化模型虽然减少了读的代价，但是将book的title冗余到publisher中，如果想修改book的title，那么需要同时修改两处地方（非原子性操作）。因此，如果读写比特别高，经常需要读取冗余的数据，但几乎不需要变更数据的情况下，这么做是值得的。反之，对数据更新的需求越高，这种方案带来的好处越少。建议是，可以冗余关联实体中不变的字段（或读取频率远远大于更新频率的字段）。比如，可以冗余书籍的title，但别冗余书籍的库存。

结合使用反范式模型，也可以反过来应用，比如在直接在book实体中冗余publisher的name字段：

```json
{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher_id: "oreilly",  // book中引用publisher
   publisher_name: "O'Reilly Media"  // 冗余publisher的name字段

}
```

最后，也不要陷入极端，害怕在应用层使用联结查询：

如果建立正确的索引，并使用投影限制返回的字段，额外查询的开销也是可以接受的，并不会比关系型数据库的join操作开销大多少。

## 树结构模式

### 父节点引用模式

通过在子节点中存储对父节点的引用，来描述一种树状结构。如下图：

![](https://docs.mongodb.com/manual/_images/data-model-tree.bakedsvg.svg)

下面通过`parent`字段，引用父节点的`_id`，对以上数结构建模：

```javascript
db.categories.insert( { _id: "MongoDB", parent: "Databases" } )
db.categories.insert( { _id: "dbm", parent: "Databases" } )
db.categories.insert( { _id: "Databases", parent: "Programming" } )
db.categories.insert( { _id: "Languages", parent: "Programming" } )
db.categories.insert( { _id: "Programming", parent: "Books" } )
db.categories.insert( { _id: "Books", parent: null } )
```

* 检索某个节点的的父节点：

```javascript
db.categories.findone({_id: "MongoDB"}).parent
```

* 你可以在`parent`字段上创建索引，以提高父节点的搜索速度：

```javascript
db.categories.createIndex({parent: 1})
```

* 查找`Databases`节点的所有直接子节点：

```javascript
db.categories.find({parent: "Databases"})
```

* 如果要检索子树，请参阅[`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup)

### 子节点引用模式

通过在父节点中存储对子节点的引用，也能描述同样的树状结构：

![](https://docs.mongodb.com/manual/_images/data-model-tree.bakedsvg.svg)

下面通过`children`数组字段，引用子节点的`_id`，对以上树结构建模：

```javascript
db.categories.insert( { _id: "MongoDB", children: [] } )
db.categories.insert( { _id: "dbm", children: [] } )
db.categories.insert( { _id: "Databases", children: [ "MongoDB", "dbm" ] } )
db.categories.insert( { _id: "Languages", children: [] } )
db.categories.insert( { _id: "Programming", children: [ "Databases", "Languages" ] } )
db.categories.insert( { _id: "Books", children: [ "Programming" ] } )
```

* 检索某个节点的直接子节点

```javascript
db.categories.findOne({_id: "Databases"}).children
```

* 在`children`字段上建索引，以提高子节点的搜索速度

```javascript
db.categories.createIndex({children: 1})
```

* 查找`MongoDB`节点的所有父节点及其兄弟节点

```javascript
db.categories.find({children: "MongoDB"})
```

只要不需要对子树进行操作，子节点引用模式为树型存储提供了一个合适方案。该模式也适用于一个节点有多个父节点的情况（上图倒过来）。

### 祖先数组模式

通过在子节点中引用父节点，以及祖先数组(太爷爷，爷爷，父节点，......），来描述树型结构：

![](https://docs.mongodb.com/manual/_images/data-model-tree.bakedsvg.svg)

下面通过祖先数组，进行树型结构建模。除了`ancestors`数组字段，文档也通过`parent`字段存储对对直接父节点的引用。

```javascript
db.categories.insert( { _id: "MongoDB", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" } )
db.categories.insert( { _id: "dbm", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" } )
db.categories.insert( { _id: "Databases", ancestors: [ "Books", "Programming" ], parent: "Programming" } )
db.categories.insert( { _id: "Languages", ancestors: [ "Books", "Programming" ], parent: "Programming" } )
db.categories.insert( { _id: "Programming", ancestors: [ "Books" ], parent: "Books" } )
db.categories.insert( { _id: "Books", ancestors: [ ], parent: null } )
```

* 查询某个节点的祖先

  ```javascript
  db.categories.findOne({_id: "MongoDB"}).ancestors
  ```

* 为`ancestors`字段创建索引，以加快祖先节点的查询速度

  ```javascript
  db.categories.createIndex({ancestors: 1})
  ```

* 查询某个节点的所有后代节点

  ```javascript
  db.categories.find({ancestors: "Programming"})
  ```

祖先数组模式通过为祖先字段创建索引，为查找某个节点的后代和祖先提供了一种快速高效的解决方案。这使得该模式成为使用子树的不错选择。

祖先数组模式比接下来要讲的物化路径模式稍慢，但是更易用。

### 物化路径模式

通过存储文档间完整的关系路径，来描述树型结构：

![](https://docs.mongodb.com/manual/_images/data-model-tree.bakedsvg.svg)

下面通过在`path`字段中存储路径的字符串，来进行树型结构建模，路径之间以`，`逗号作为分隔符：

```javascript
db.categories.insert( { _id: "Books", path: null } )
db.categories.insert( { _id: "Programming", path: ",Books," } )
db.categories.insert( { _id: "Databases", path: ",Books,Programming," } )
db.categories.insert( { _id: "Languages", path: ",Books,Programming," } )
db.categories.insert( { _id: "MongoDB", path: ",Books,Programming,Databases," } )
db.categories.insert( { _id: "dbm", path: ",Books,Programming,Databases," } )
```

* 检索整个树，并按`path`字段排序：

  ```javascript
  db.categories.find().sort({path: 1})
  ```

* 对`path`字段使用正则来查找某个节点的所有后代：

  ```javascript
  db.categories.find({path: /,Programming,/})
  ```

* 你也可以检索最高层`Books`的所有后代：

  ```javascript
  db.categories.find{path: /^,Books,/}
  ```

* 为`path`字段创建索引：

  ```javascript
  db.categories.createIndex({path: 1})
  ```

  这个索引可能提高某些查询的性能：

  * 从根路径开始的子树查询将获得显著性能提升，比如：`/^,Books,/`或者`/^,Books,Programming,/`
  * 未从根路径开始的子树查询，如`/,Programming,/`，将检查整个索引。这种情况下，如果索引显著小于整个集合，也会获得一些性能提升。

### 嵌套集模式

该模式适合静态的树结构，暂不讨论。

## 特定应用模式

### 原子操作

如之前所述，在MongoDB中，写操作在单个文档上保证原子性。即便MongoDB从4.0开始支持多事务，但在很多情况下，采用反范式的数据模型（子文档），仍然是最佳选择。因此对于那些必须一起更新的字段，可以考虑将其嵌入一个文档中，以保证原子性操作。

比如，书籍的库存和结账信息应该是同步的，因此在同一文档中嵌入`available`和`checkout`字段可以确保对这两个字段的原子性更新：

```javascript
{
    _id: 123456789,
    title: "MongoDB: The Definitive Guide",
    author: [ "Kristina Chodorow", "Mike Dirolf" ],
    published_date: ISODate("2010-09-24"),
    pages: 216,
    language: "English",
    publisher_id: "oreilly",
    available: 3,
    checkout: [ { by: "joe", date: ISODate("2012-10-15") } ]
}
```

如果有新的结账信息要更新，像下面这样就可以了：

```javascript
db.books.updateOne({
    {_id: 123456789, available: {$gt: 0}},
    {
        $inc: {available: -1},
        $push: {checkout: {by: "abc", date: new Date()}}
    }
})
```

### 关键字搜索

如果你的应用需要对包含文本内容的字段进行查询，你可以进行完全匹配，或者使用正则进行匹配。然后，这种方式在很多时候无法满足要求。

下面提供一种支持关键字搜索的方法：将要用的关键字存储到文档的一个数组字段中，并创建为该字段创建多键索引。

比如，搜索书籍时：

````javascript
{ title : "Moby-Dick" ,
  author : "Herman Melville" ,
  published : 1851 ,
  ISBN : 0451526996 ,
  topics : [ "whaling" , "allegory" , "revenge" , "American" ,  // 将要搜索的关键词放在topic数组中
    "novel" , "nautical" , "voyage" , "Cape Cod" ]
}
````

对`topic`数组字段创建多键索引：

```javascript
db.books.createIndex({topics: 1})
```

多键索引将会为数组中的每个关键字创建单独的索引条目，然后你就可以依据关键字进行查询：

```javascript
db.books.findOne({topic: "voyage"}, {title: 1})  // 依据voyage关键字查询书籍，并只返回title字段
```

如果数组中包含大量关键字，比如成百上千个，将导致插入时更大的索引开销。

#### 关键字索引的局限

比起全文产品，关键字索引在以下方面存在不足或者不适用：

* 截取。不能解析根或相关单词关键字。
* 同义词。只能由应用层为同义词或相关查询提供支持。
* 排名。不提供加权结果的方法。
* 异步索引。MongoDB同步构建索引，这意味着索引始终是最新的，然后异步批量索引有些时候可能更高效。

关键字索引的

### 货币数据

暂不讨论

### 时间数据

MongoDB默认以UTC格式存储时间，并将任何本地时间表示转化为这种形式。时区信息由应用程序保存，并据此计算本地时间。

例如，你可以将当前时间，以及当前客户端与UTC的偏移量都存下来：

```javascript
var now = new Date();
var offset = now.getTimezoneOffset(); // -480, 比UTC快480分钟
db.data.save({
    date: now,
    offset: offset
});
```

重建本地原始时间：

```javascript
var record = db.data.findOne();
var localNow = new Date(record.date.getTime() - (record.offset * 60000))
```

# 数据建模引用

通常来说，采用反范式的数据模型（子文档）是最佳选择。但是，在某些情况下，将关联的信息分开存储到不同的集合或者数据库中也是合理的。

> 注意：MongoDB 3.2 引入了`$lookup`管道，以便对同一数据库内的集合进行左外连接操作（主动连表）。更多信息和示例，请参阅[`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup)
>
> 从3.4开始，还可以使用`$graphLookup`管道连接集合执行递归搜索。更多信息和示例，请参阅 [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup)。

在`$lookup`和`$graphLookup`出现前，你可以采用本部分所介绍的方案：

## 手动引用（Manual references）

在一个文档中保存另一个文档的`_id`字段作为引用。然后应用程序在需要时执行第二次查询来获取关联数据。大多数情况下，手动引用简单且够用。

但是，手动引用无法传达数据库和集合的名字，所以，如果一个集合内的文档与多个其它集合的文档关联，你可能需要考虑使用DBRefs。

## DBRefs

一个文档通过另一个文档的`_id`字段，集合名，以及（可选的）数据库名进行引用。通过包含这些名称，DBRefs使得跨集合的文档链接更容易。

如果要解析DBRefs，应用程序必须执行额外的查询已获得被引用的文档。很多驱动提供了辅助方法来自动执行DBRef查询，但不会自动将DBRefs解析为文档。

下面我们看看如何进行DBRef引用：

```javascript
// users集合
{
    "_id": 12345678
}

// post集合
{
    "_id": 87654321,
    "title": "a beautiful poem",
    "body": "asdif asd fiasdf jasdlf idf jdjifj idfj idlllle iidjfjjfi asssdf",
    "author": {
           "$ref": "users"，  // 被引用文档所在的集合名
           "$id": "12345678",  // 被引用文档的_id值
           "$db": "web-application"  // （可选）被引用文档所在的数据库
        }
}
```

DBRef格式如下：

```json
{ "$ref" : <value>, "$id" : <value>, "$db" : <value> }  // 注意，必须按照这个字段顺序
```

最后，除非你有令人信服的理由使用DBRefs，否则请选择手动引用。

