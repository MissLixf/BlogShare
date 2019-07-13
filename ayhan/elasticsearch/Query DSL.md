ES提供了基于JSON的查询DSL，它由两种语句组成：

* 叶子查询（Leaf Query）：查询特定字段的特定值，比如`match`, `term`, `range`查询，这些查询可以单独使用。
* 复合查询（Compound Query）：复合查询包装其他叶子查询和复合查询，以逻辑运算的方式（比如`bool`，`dis_max`查询）连接多个查询，或更改它们的行为（比如，`constant_score`查询）

依据它们是在查询上下文（query context）还是过滤上下文（filter context），查询语句的行为也有所不同。

##  查询和过滤上下文

#### 查询上下文

在查询上下文解决的是这种问题：“此文档与此查询的匹配程度如何？”，除了决定文档是否匹配外，查询语句还会计算一个`_score`字段，表示该文档相对其他文档的匹配程度。

位于`query`参数下的语句，处于查询上下文。

#### 过滤查询

过滤上下文解决的是这种问题：“此文档是否匹配该查询”，回答只有是或者否，不会计算相关性分值。过滤上下文主要用于过滤结构化的数据，比如：

* 这个时间是否属于2015-2016年之间
* status字段的值是否是`1`

ES会缓存经常使用的查询，以提升性能。

位于`filter`参数下的语句，处于过滤上下文中。比如在`bool`查询中的`filter`或`must_not`参数，在`constant_score`查询中的`filter`参数，或`filter`聚合。

下面的示例展示搜索API中，在查询上下文和过滤上下文中的语句。这条查询会匹配满足下列条件的文档：

* `title`字段包含单词`search`
* `content`字段包含单词`elasticsearch`
* `satus`字段指就是`published`
* `publish_date`字段的日期大于等于2015.1.1

```json
GET /_search
{
  "query": {  // 查询上下文
    "bool": {
      "must":[
        {"match": {"title": "search"}},
        {"match": {"content": "elasticsearch"}}
        ],
      "filter":[  // 过滤上下文
        {"term": {"status": "published"}},
        {"range": {"publish_data": {"gte": "2015-01-01"}}}
        ]
    }
  }
}
```

说明：

* `bool`和`match`语句用在查询上下文中，这意味着他们被用来评估每个文档的匹配程度（分值）。
* `term`和`range`语句用在过滤上下文中，它们会过滤掉不匹配的文档，但是不会影响匹配文档的分值

## 匹配所有

`match_all`查询，匹配所有文档，每个文档`_score`值为1：

```json
GET {index}/_search
{
  "query": {
    "match_all": {}
  }
}
```

与之相反的是`match_none`，不匹配任何文档

## 全文查询

全文查询允许你搜索分词后的文本字段，比如电子邮件的内容。包括：

* `match` ：全文查询的标准方式，包括模糊匹配（fuzzing），短语或邻近查询。
* `match_phrase` ：类似`match`，但只用于精确匹配短语或单词邻近匹配
* `match_phrase_prefix`：同上，但是只对最后一个单词进行通配符搜索
* `match_bool_prefix`：创建一个bool查询，对每个词条创建一个`term`查询，最后一个词条除外，该词条作为`prefix`查询匹配
* `multi_match`：`match`查询的多字段形式
* `common`词条查询：相对专门的查询，适用于不常用的单词
* `query_string`查询：支持紧凑的Lucene查询字符串语法，支持在单个查询字符串中指定`AND|OR|NOT`逻辑条件和多字段搜索。仅限专业用户使用
* `simple_query_string`查询：比起`query_string`语法更简单，更健壮，适用于普通用户使用
* `intervals`查询：允许对匹配词条的顺序和邻近度进行细粒度（fine-grained）的控制

### match

我们先创建一条文档：

```json
POST twitter/_doc
{
  "user": "ayhan",
  "post_date": "2009-11-15T14:12:12",
  "message": "let me out from here"
}
```

基本用法：

```json
GET twitter/_search
{
  "query": {
    "match": {
      "message": "out me"  // messag是搜索的字段，值是要查询的字符串
    }
  }
}
```

因此上述查询的结果是message字段中包含词条out, 或me的所有文档。

match查询会对提供的文本进行分词，并构建布尔查询。逻辑运算符`operator`默认是`or`，我们也可以手动设为`and`，来控制布尔语句。

```json
GET twitter/_search
{
  "query": {
    "match": {
      "message": {
        "query": "out me",  // 指定查询字符串
        "operator": "and"  // 指定逻辑运算符
      }
    }
  }
}
```

这样查询的结果是message字段中同时包含词条out 和 me的所有文档，范围将大大缩小。

除了指定`operator`，我们还可以指定如下参数：

* `analyzer` 指定分词器
* [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)，没看懂
* `lenient` ：异常处理，是否忽略由数据类型错误导致的异常，默认`false`，比如使用文本查询字符串来查询数字字段
* `zero_terms_query`：零项查询，指定查询词条为空时的行为（比如指定的分词器移除了所有词条），其值为：
  * `none`，相当于`match_none`，默认值
  * `all` ，相当于`match_all`
* `cutoff frequency` ：指定文档高低频的分界线
* `synonyms` 对于同义词定义：`ny, new york`，相当于`ny OR (new york)`

### match phrase

短语匹配，示例：

```json
GET twitter/_search
{
  "query": {
    "match_phrase": {
      "message": "huawei mate pro"  // 查询message字段包含huawei mate pro的文档
    }
  }
}
```

类似match查询，短语匹配首先也会将查询字符串解析为词条列表，然后对这些词条进行搜索。但是只保留同时包含所有词条，且位置顺序与搜索词条一致，中间不夹杂其他单词的文档。

短语匹配是利用倒排索引中的位置信息实现的。

判断文档是否和短语`huawei mate pro`匹配，要同时满足如下条件：

* `huawei`、`mate`和`pro`必须全部出现
* `mate`的位置比`huawei`大1
* `pro`的位置比`mate`大2

### multi_match

multi_match基于match查询，但是支持查询多个字段：

```json
GET article/_search
{
  "query": {
    "multi_match": {
      "query": "hello world",  // 查询字符串
      "fields": ["title", "content"]  // 要查找的字段
    }
  }
}
```

如果不指定具体字段，将查询所有字段。multi_match查询支持match查询的所有参数。

#### 通配符

fields参数可以支持通配符，比如：

```json
"fields": [ "title", "*_name" ]
```

可以查询`title`, `first_name`和`last_name`字段

#### 字段权重

可以使用`^`标记，增大个别字段的权重：

```json
"fields": ["title^3", "content"]
```

如此一来，`title`字段的权重3倍于`content`字段，这将会影响搜索结果中`_score`的打分（打分越高，排名越靠前），`^`标记的权重的值越大，对打分影响越大。

#### 查询方式

`multi_match`的查询方式取决于`type`参数，默认值是`best_fields`，全部可选项及区别具体参考[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)

##### best_fields

如果你想搜索多个单词在某个字段中的最好匹配，`best_fields`最合适。例如，"huawei mate pro"出现在单个字段中，比"huawei"出现在一个字段，"mate pro"出现在另一个字段更有意义。

`best_fields`方式为每个字段创建一个`match`查询，并包在一个`dis_max`查询中，来找到单个最匹配的字段。比如：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```

会这样执行：

```json
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

打分方式：通常情况下，`best_fields`使用匹配最好的字段的打分，但是如果指定了`tie_breaker`参数，则有所不同，具体查看官网。

注意：

`best_fiels`和`most_fields`方式都会为每个字段创建一个`match`查询，因此`operator`和`minimum_should_match`参数将对每个字段单独应用，这可能不是你想要的。比如：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

将会以这样的逻辑执行：

```
(+first_name:will +first_name:smith) | (+last_name:will  +last_name:smith)
```

也就是说，如果一个文档要匹配，必须在单个字段中出现查询的所有词条。

##### most_fields

如果查询的多个字段的文本经过分词处理后（不管以怎样的方式），包含同样的文本，那么`most_fields`方式最合适。比如，主字段的查询结果可能包含同义词，词干，没有变音符的词条等（可以理解为关联性较强的文档）。第二个字段可能包含原始的查询词条（更准确，但是范围更小，但是打分高）。通过合并这些字段的打分，我们可以匹配尽可能多的文档（主字段），但是最接近的结果又可以获得优先展示（第二个字段）。

如下查询：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown fox",
      "type":       "most_fields",
      "fields":     [ "main_filed", "strict_field" ]
    }
  }
}
```

将会这样执行：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "main_filed":  "quick brown fox" }},
        { "match": { "strict_field": "quick brown fox" }}
      ]
    }
  }
}
```

打分方式：每个`match`语句的打分被加总，然后除以`match`语句的数量。

##### cross_fields

`cross_fields`方式特别适合要匹配多个字段的结构化文档。比如，要从first_name和last_name字段中查询"Will Smith"，最匹配的应该是Will出现firs_name字段，而Smith出现在last_name字段。

一种处理这种查询的方式是只需要将first_name和last_name索引到一个full_name字段。但是，这只能在索引时操作。而cross_fields方式尝试在查询时解决这种问题。它先将查询字符串分词为一个个词条，然后在指定字段中查找每个词条。

下面的查询：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "cross_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

将会这样执行：

```
+(first_name:will  last_name:will)
+(first_name:smith last_name:smith)
```

也就是说，一个文档如果要匹配，所有的词条必须出现在至少一个字段中。

### common

`common`查询时可以将停用词考虑在内，对于能提高查询精度，影响查询结果的停用词是不错的选择，并且不会牺牲性能。

#### 问题

查询的每个词条都有花销。比如，搜索"The brown fox"将产生3次词条查询，也就是"the"，"brown"，"fox"，每次查询都要搜索索引中的所有文档。对于"the"的查询很可能会匹配到大量文档，但是比起另外两次查询，"the"对于相关性的影响要小得多。

之前的方案是，忽略哪些高频出现的词条。也就是将"the"作为停用词对待，这样不仅能减少索引的大小，还可以较少需要执行查询的词条的数量。

这种方案的问题是，尽管停用词对相关性的影响很小，但它们仍然很重要。如果移除了停用词，不仅影响查询的精度（比如，无法区分"happy"和"not happy"），甚至是丢失查询结果（"To be or not to be"压根不会被搜到）。

#### 解决方案



