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

子文档可以存储关联数据，并嵌在文档的某个字段或者数组中。这些非标准的数据模型可以方便应用程序在单次数据库操作中检索和操作相关数据。在很多场景下，这种非标准的数据模型都是非常理想的。

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

有关子文档优缺点，具体查看：https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding

### 引用

通过文档间的链接（或引用），引用用来存储着数据间的关系。个人理解，类似于SQL中的外键。应用可以解析这些引用来访问相关的数据。广义上来讲，这些是标准的数据模型。

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

有关使用应用的优缺点，具体查看：https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-referencing

## 写操作的原子性

### 单文档原子性

在MongoDB中，一次写操作在单个文档上保证原子性，即使这个操作修改了该文档的多个子文档。支持在单个文档中嵌入子文档的这种非标准化数据结构，比起跨多个文档和集合的标准方式，更有助于原子性操作。

如果单次写操作修改了多个文档（比如，`db.collection.update_many()`），那么对每个文档的修改是原子性的，但是这整个操作并不是原子性的。

从4.0版本开始，对于需要多文档读写保证原子性的场景，MongoDB为副本集提供了多文档事务支持。具体查看：https://docs.mongodb.com/manual/core/transactions/，

### 多文档事务

>注意：
>
>在多数情况下，多文档事务会导致更多的性能开销。尽管有多文档事务支持，但也不应该以此来取代高效的模型设计。对于很多场景，非标准化的数据模型（子文档和数组）仍然是最好的选择。也就是说，合适的数据建模将最大限度地减少对多文档事务的需求。

## 数据的使用和性能

在设计数据模型时，请考虑应用程序如何使用你的数据库。例如，如果你的应用只使用最近插入的文档，请考虑使用[Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/)。或者你的应用主要是对集合的读操作，那么对常见的查询添加索引有助于提高性能。

更多信息可查看：[操作因素和数据模型](https://docs.mongodb.com/manual/core/data-model-operations/)

## 文档的增长和MMAPv1

某些更新，比如往数组插入元素，或者增加字段，会增加文件的大小。

对于已经弃用的MMAPv1文档引擎，如果文档大小超出了该文档的分配空间，MongoDB会在磁盘上重新分配该文档。使用废弃的MMAPv1引擎时，出于对文档增长的考虑，将会影响对标准化或非标准化数据的决策。

## 其他参考资源

* [Thinking in Documents Part 1 (Blog Post)](https://www.mongodb.com/blog/post/thinking-documents-part-1?jmp=docs)

# 数据建模相关概念

## 数据模型设计

有效的数据模型应满足应用的需求。文档结构的设计关键是使用嵌入式（embed），还是引用（references）的选择。

####嵌入式数据模型

#### 