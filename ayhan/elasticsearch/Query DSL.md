"ES提供了基于JSON的查询DSL，它由两种语句组成：

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

`common`查询将要查询的词条分为两组：重要的（比如，低频词条）和相对不重要的（比如，像停用词这样的高频词条）。

首先，查询能匹配重要词条的文档，包含这类词条的文档相对较少并且对相关性影响较大。

然后，对不重要的词条执行第二次查询，这类词条在文档中高频出现并且对相关性影响较小。但是，这里不会计算所有匹配文档的相关性打分，只计算在第一次查询中已经匹配的文档。通过这种方式，高频词条可以改进相关性计算，却不用付出过大的性能代价。

如果查询只包含高频词条，那么每次查询会以`AND`连接，也就是所有的词条都需要。尽管单个词条会匹配到大量文档，但是词条的联合可以缩小范围为最相关的文档。不过，通过指定`minimum_should_match`，每次查询也可以用`OR`连接，但是这种情况下，需要指定一个足够大的值。

词条是分布在高频组还是低频组，是基于`cutoff_frequency`，其可以指定为绝对频率（`>=1`），或者是相对频率（`0.0 .. 1.0`），高于设定值的属于高频词条，反之低频词条。

注意：common不支持多字段查询

#### 示例

下面的示例中，文档频率大于0.1%的单词（比如，"this"和"is"）视为*common terms*

```json
GET /_search
{
    "query": {
        "common": {
            "body": {
                "query": "this is bonsai cool",
                "cutoff_frequency": 0.001
            }
        }
    }
}
```

要匹配的词条数可以通过以下参数控制：

* `minimum_should_match`  指定低频词至少要匹配几个
  * `high_freq`  指定高频词至少匹配几个
  * `low_freq`  指定低频词至少匹配几个
* `low_freq_operator`（默认"or"）
* `high_freq_oprator`（默认"or"）

对于低频词条，设置`low_freq_operator`为`and`，来要求所有词条都满足：

```json
GET /_search
{
    "query": {
        "common": {
            "body": {  // 查询body字段
                "query": "nelly the elephant as a cartoon",
                "cutoff_frequency": 0.001,
                "low_freq_operator": "and"
            }
        }
    }
}
```

以上相当于：

```json
GET /_search
{
    "query": {
        "bool": {
            "must": [
            { "term": { "body": "nelly"}},
            { "term": { "body": "elephant"}},
            { "term": { "body": "cartoon"}}
            ],
            "should": [
            { "term": { "body": "the"}},
            { "term": { "body": "as"}},
            { "term": { "body": "a"}}
            ]
        }
    }
}
```

也可以通过`minimum_should_match`参数指定低频词（重要词）出现的最小百分比，比如：

```json
GET /_search
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant as a cartoon",
                "cutoff_frequency": 0.001,
                "minimum_should_match": 2
            }
        }
    }
}
```

以上约等于：

```json
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "bool": {
                    "should": [
                    { "term": { "body": "nelly"}},
                    { "term": { "body": "elephant"}},
                    { "term": { "body": "cartoon"}}
                    ],
                    "minimum_should_match": 2  // nelly、elephant、cartoon中至少匹配到两个，才影响相关性
                }
            },
            "should": [
                { "term": { "body": "the"}},
                { "term": { "body": "as"}},
                { "term": { "body": "a"}}
                ]
        }
    }
}
```

如果想同时指定低频词和高频词的匹配数，可以指定`minimum_should_match`的`low_freq`和`high_freq`：

```json
GET /_search
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant not as a cartoon",
                "cutoff_frequency": 0.001,
                "minimum_should_match": {
                    "low_freq" : 2,
                    "high_freq" : 3
                }
            }
        }
    }
}
```

以上相当于：

```json
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "bool": {
                    "should": [
                    { "term": { "body": "nelly"}},
                    { "term": { "body": "elephant"}},
                    { "term": { "body": "cartoon"}}
                    ],
                    "minimum_should_match": 2  // 以上3个词条至少匹配两个
                }
            },
            "should": {
                "bool": {
                    "should": [
                    { "term": { "body": "the"}},
                    { "term": { "body": "not"}},
                    { "term": { "body": "as"}},
                    { "term": { "body": "a"}}
                    ],
                    "minimum_should_match": 3  // 以上四个词条至少匹配3个
                }
            }
        }
    }
}
```

如果为高频词指定了`minimum_should_match`，并且查询字符串中只有高频词时：

```
GET /_search
{
    "query": {
        "common": {
            "body": {
                "query": "how not to be",
                "cutoff_frequency": 0.001,
                "minimum_should_match": {
                    "low_freq" : 2,
                    "high_freq" : 3
                }
            }
        }
    }
}
```

以上相当于：

```json
GET /_search
{
    "query": {
        "bool": {
            "should": [
            { "term": { "body": "how"}},
            { "term": { "body": "not"}},
            { "term": { "body": "to"}},
            { "term": { "body": "be"}}
            ],
            "minimum_should_match": "3<50%"  // 老实说，我也没看明白
        }
    }
}
```

最后，`common`查询也支持`boost`和`analyzer`参数。

### simple_query_string



## term查询

term查询是一种适用于结构化数据的精确查询，它不会像全文检索一样对搜索做分词。结构化数据包括日期，IP，价格，商品条码等。注意：要避免对text字段使用term查询（text字段的内容经过语义处理，可能发生变化）。

### term

```json
GET /_search
{
    "query": {
        "term": {
            "user": {  // 要查找的字段
                "value": "Kimchy",  // 该字段要精确包含的值
                "boost": 1.0
            }
        }
    }
}
```

### terms

同term，但是可以搜索多个值：

```json
GET /_search
{
    "query" : {
        "terms" : {
            "user" : ["foo", "bar"],
            "boost" : 1.0
        }
    }
}
```

以上只要user字段包含foo, 或者bar即可（Foo， bars不算）

## 复合查询

### constant_score

包装一个过滤查询，所有匹配的文档会给予相同的打分1.0（因为过滤上下文不计算打分）

```json
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}
```

### bool

匹配布尔连接的多个查询，类别如下：

* must  必须匹配，影响打分()
* filter 必须匹配，但是忽略打分（0分）
* should  应该匹配，影响打分
* must_not  不能匹配（语句会在过滤上下文中执行，因此打分也会忽略（0分））类似于exclude

注意：must和should的打分会累加为匹配文档最终打分

```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,  // 是否是针对should生效？
      "boost" : 1.0
    }
  }
}
```

另外，must, filter, must_not也可以使用列表的形式指定多个条件。

bool查询中有一个`match_all`查询，会给所有文档打1.0分

```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

另外，`constant_score`和`match_all`是同样的效果：

```json
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

### dis_max

### function_score

### boosting

权重查询，返回能够匹配`positive`的文档，并减小匹配`negative`文档的打分

```json
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```



