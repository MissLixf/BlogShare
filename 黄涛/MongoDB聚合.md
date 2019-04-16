聚合操作处理数据并返回计算后的结果。它将来自多个文档的值组合在一起，并且可以对分组数据执行各种操作以返回单个结果。MongoDB提供了三种执行聚合的方式：聚合管道、map-reduce函数、和单用途聚合方法。

## 聚合管道

MongoDB的聚合基于管道的概念。文档进入一个多段管道，被转化为聚合后的结果。

最基本的管道阶段提供过滤器（filters）进行查询和文档转换（修改文档的输出格式）。其他管道提供文档的分组、排序，以及用于聚合数组内容的工具。此外，管道阶段还可以使用运算符完成计算平均值或拼接字符串之类的任务。

管道利用MongoDB的原生运算来提供高效的数据聚合，也是执行数据聚合的首先方法。

聚合管道可以使用索引来提高某些阶段的性能。此外，聚合管道有一个内部优化阶段。有关详细信息，请参阅 [管道运算符和索引](https://docs.mongodb.com/manual/core/aggregation-pipeline/#aggregation-pipeline-operators-and-performance)以及 [聚合管道优化](https://docs.mongodb.com/manual/core/aggregation-pipeline-optimization/)

![](https://docs.mongodb.com/manual/_images/aggregation-pipeline.bakedsvg.svg)

### 阶段运算符

MongoDB提供了`db.collection.aggregate()`和`db.aggreate()`方法用于管道聚合操作。这两个方法接收阶段运算符的数组，然后文档依次通过这些阶段：

```javascript
db.collection.aggregate( [ { <stage1> }, {<stage2>}, ... ] )
```

```javascript
db.aggregate( [ { <stage1> }, {<stage2>}, ... ] )
```

关于可用的阶段运算符，参阅的[聚合管道阶段运算符](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/#aggregation-pipeline-operator-reference)

### 限制

#### 尺寸限制

聚合运算可以返回一个游标或者将结果存储到一个集合中。结果中的每个文档遵守BSON文档的大小限制，当前是16MB，否则将抛出错误。不过在管道处理过程中，是可以超过这个限制的。`db.collection.aggregate()`方法默认返回一个游标。

#### 内存限制

管道阶段的内存限制是100MB，如果要处理大数据集，可以使用`allowDiskUse: true`选项，将数据写入临时文件。

## Map-Reduce函数

map-reduce操作有两个阶段：map阶段处理文档并为每个文档触发一个或多个对象，reduce阶段对reduce阶段的输出进行组合。此外map-reduce还有一个可选的最finalize阶段对结果作最后的修改。就像其他聚合操作一样，map-reduce也可以指定一个查询条件来选取输入的文档，并对结果进行排序和限定。

map-reduce使用自定义的JavaScript函数，尽管这提供了比管道更大的灵活性，但通常也相对低效且更加复杂。

![](https://docs.mongodb.com/manual/_images/map-reduce.bakedsvg.svg)

## 单用途聚合方法

MongoDB还提供了`db.collections.estimatedDocumentCount()`，`db.collection.count()`，以及`db.collection.distinct()`方法，这些操作可以简单地聚合单个集合内的文档。但是比起前文的两种方式，它缺乏灵活性且能力有限。

![](https://docs.mongodb.com/manual/_images/distinct.bakedsvg.svg)

## 其他特性和行为

有关以上三种方式的比较，请参阅 [聚合命令比较](https://docs.mongodb.com/manual/reference/aggregation-commands-comparison/)

