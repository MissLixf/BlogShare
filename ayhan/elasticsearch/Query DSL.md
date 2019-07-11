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

## 同义词图过滤器 Synonym Graph Token Filter

`synonym_graph`词条过滤器（token filter）用于处理同义词。

注意：该过滤器仅适用于作为搜索分词器的一部分。如果想在索引期间应用同义词，需要使用标准的[同义词过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html)

同义词可以使用配置文件指定，如下：

```json
PUT /{index}
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "search_synonyms" : { // 自定义分词器名字
                        "tokenizer" : "whitespace",
                        "filter" : ["graph_synonyms"]  // 使用自定义filter
                    }
                },
                "filter" : {
                    "graph_synonyms" : {
                        "type" : "synonym_graph",  // 指定类型为同义词图
                        "synonyms_path" : "analysis/synonym.txt"  // 指定同义词词库位置
                    }
                }
            }
        }
    }
}
```

说明：同义词词库的位置是相对于`config`目录的相对地址。另外以下参数也是可用的：

* `expand` 默认为`true`
* `leninet` 默认为`false`，是否忽略同义词配置的解析异常，记住，如果设置为`true`，只要那些无法被解析到同义词规则会被忽略。如下示例：

```json
PUT /{test_index}
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "synonym" : {
                        "tokenizer" : "standard",
                        "filter" : ["my_stop", "synonym_graph"]
                    }
                },
                "filter" : {
                    "my_stop": {
                        "type" : "stop",
                        "stopwords": ["bar"]
                    },
                    "synonym_graph" : {
                        "type" : "synonym_graph",
                        "lenient": true,
                        "synonyms" : ["foo, bar => baz"]  // 手动定义同义词映射
                    }
                }
            }
        }
    }
}
```

说明：bar在自定义的my_stop的filter中被剔除，但是在synonym_graph的filter中，foo => baz仍然被添加成功。但如果添加的是"foo, baz => bar", 那么什么也不会被添加到同义词列表。这时因为映射到目标单词"bar"本身作为停用词已被剔除。相似的，如果映射是"bar, foo, baz"并且`expand`设置为`false`，那么不会添加任何同义词映射，因为当`expand`为`false`时，目标映射是第一个单词，也就是"bar"。但是如果`expand=true`，那么映射将会添加为"foo, baz => foo, baz"，即所有词相互映射，除了停用词。

同义词配置文件如下：

```tcl
# 空行和#号开头的都是注释

# 近义词以 => 映射，这种映射会忽略模式中的 expand 参数
# 匹配到"=>"左侧的词条后，被替换为"=>"右侧的词条。
i-pod, i pod => ipod,
sea biscuit, sea biscit => seabiscuit

# 近义词以逗号分隔，映射行为由模式中的expand参数决定。如此同样的近义词文件，可以用于不同的策略
ipod, i-pod, i pod
foozball , foosball
universe , cosmos
lol, laughing out loud

# 当expand==true, "ipod, i-pod, i pod" 等价于:
ipod, i-pod, i pod => ipod, i-pod, i pod
# 当expand==false, "ipod, i-pod, i pod" 仅映射第一个单词， 等价于:
ipod, i-pod, i pod => ipod

# 多个同义词映射将会合并
foo => foo bar
foo => baz
# 等价于：
foo => foo bar, baz
```

虽然可以直接在filter中使用`synonyms`定义同义词，比如：

```json
PUT /test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "filter" : {
                    "synonym" : {
                        "type" : "synonym_graph",
                        "synonyms" : [ // 直接定义
                            "lol, laughing out loud",
                            "universe, cosmos"
                        ]
                    }
                }
            }
        }
    }
}
```

但是对于大型的同义词集合，还是推荐使用`synonyms_path`参数在文件中定义。

### 测试

定义mapping

```json
PUT article
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "ik_max_word",
            "filter": ["my_synonyms"]
          }
        },
        "filter": {
          "my_synonyms": {
            "type": "synonym_graph",
            "synonyms_path": "analysis/synonyms.txt"
          }
        },
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

在ES的`config/analysis`目录下新建`synonyms.txt`文件，内容如下：

```tcl
北京大学, 北大
```

索引一篇文档：

```json
PUT article/_doc/1
{
  "content": "北京大学今年考研一飞冲天"
}
```

试着搜索：

```json
GET article/_search
{
  "query": {
    "match": {
      "content": {
        "query": "北大"
      }
    }
  }
}
```

搜索结果：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "article",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "content" : "北京大学今年考研一飞冲天"
        }
      }
    ]
  }
}
```

## 同义词过滤器 Synonym Token Filter

同义词过滤器配置示例：

```json
PUT /test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "my-synonym" : {  // 自定义分词器名字
                        "tokenizer" : "whitespace",
                        "filter" : ["my_synonym_fltr"]  // 指定过滤器
                    }
                },
                "filter" : {
                    "my_synonym_fltr" : {  // 自定义同义词过滤器名字
                        "type" : "synonym",  // type 为synonym，
                        "synonyms_path" : "analysis/synonym.txt"
                    }
                }
            }
        }
    }
}
```

其实同义词过滤器和以上面的同义词图过滤器配置用法都是一样的。区别如下：

* 前者`type=synonym`，后者`type=synonym_graph`
* 前者可以在文档索引期间使用，后者只能作为搜索分词的一部分。

另外在同义词图中的测试，将type更改后，测试仍然适用。



