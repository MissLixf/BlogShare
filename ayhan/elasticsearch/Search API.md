## Search

搜索条件可以通过查询字符串，也可以在请求体中传递。

搜索接口支持从多个索引中查找文档vj。

基本格式：

```shell
# 单索引内检索文档
GET /{index}/_search?q={field}:xxx

# 多索引内检索文档
GET /{index1, index2}/_search?q={field}: xxx

# 全部索引内检索文档
GET /_all_/_search?q={field}: xxx
```

## URI Search

通过URI传参的方式比较简单，但是不能支持所有的搜索选项。

 RUI支持传参如下：
<https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-uri-request.html>

## Request Body Search

搜索请求可以使用search DSL，并包含Query DSL，比如：

```json
GET /twitter/_search
{
    "query": {
        "term": {"user": "kimchy"}
    }
}
```

注意：其实GET请求也是可以带请求体的。考虑到不是所有的客户端支持GET携带请求体，因此，也上请求也可以通过POST发送。

返回结果如下：

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.47000363,
    "hits" : [
      {
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "A6y0umsBAkV3IICsYCLL",
        "_score" : 0.47000363,
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T14:12:12",
          "message" : "trying out Elasticsearch"
        }
      },
      {
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "BKzNumsBAkV3IICsUCJn",
        "_score" : 0.47000363,
        "_routing" : "kimchy",
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T14:12:12",
          "message" : "trying out Elasticsearch"
        }
      }
    ]
  }
}
```

如果只是想知道是否有匹配条件的文档，可以设置`size`参数为`0`，这表示你并不需要搜索的结果。或者，将`terminate_after`设置为`1`，表示只要找到一个一个文档，查询操作就可以结束了。

```shell
curl -X GET "localhost:9200/_search?q=message:number&size=0&terminate_after=1"
```

如果查询时不指定任何条件，将返回所有结果，比如：

```shell
GET /twitter/_search
```

查询的方式：

* match 查询，针对全文检索
* term 查询，词条精确搜索

查询的逻辑运算：

搜索时如果有多个关键字，ES默认他们是`或`的关系：

```json
GET twitter/_search
{
  "query": {
    "match": {
      "message": "out me"
    }
  }
}
```

以上搜索查找message字段中包含`out`或`me`的文档。

如果要指定`and`搜索，需要使用布尔查询：

```json
GET twitter/_search
{
  "query": {
    "bool":{
      "must": [
        {"match": {"field1": "out"}},
        {"match": {"field2": "me"}}
        ]
    }
  }
}
```



### `docvalue fields` 字段的文档值

为每个命中返回字段的文档值表示，比如：

```json
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "docvalue_fields" : [
        "my_ip_field", // 直接用字段名
        {
            "field": "my_keyword_field"   // 也可以使用对象标记
        },
        {
            "field": "my_date_field",  // 字段名也支持通配符，比如 *_date_field
            "format": "epoch_millis"   // 在对象标记中可以自定义格式
        }
    ]
}
```

`docvalue_fields`支持两种两种用法：

* 直接指定字段名
* 对象标记

`docvalue_fields`支持所有启动了文档值的字段，无论这些字段是否被存储。

如果`docvalue_fields`中指定了未启用文档值的字段，它将尝试从字段数据的缓存中加载值，从而导致该字段的词条加载到内存中，消耗更多的内存。

另外，大部分字段类型不支持自定义格式，但是：

* `Date`类型的字段可以指定日期格式
* `Numeric`类型的字段可以指定Decimal样式

注意：

`docvalue_fields`不能加载嵌套对象中的字段，如果某字段的路径中包含嵌套对象，那么无法返回任何数据。要访问嵌套的字段，只能在`inner_hits`块中使用`docvalue_fields`

#### 什么是`doc values`？

大部分字段会被默认索引，以备查询。倒排索引允许查找词条，并找到包含词条的相关文档。但是排序，聚合，以及在脚本中访问字段的值要使用的不同的数据访问模式，我们需要查找文档并找到字段中的词条，而不是先找到词条，再找到文档。

文档值是磁盘存储的数据结构，在文档索引时构建。其值与`_source`字段相同，但是是以列的方式，使其对排序和聚合操作更高效。几乎所有的字段类型都支持文档值，除了`analyzed`字符串字段。

支持文档值的字段默认都已开启这一特性。如果你确定不需要基于某字段进行排序、聚合，或者在脚本中访问该字段的值，你可以禁用这一特性，以节约磁盘空间。

```json
PUT my_index
// curl -X PUT "localhost:9200/my_index" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "status_code": {  // 默认启用文档值
        "type":       "keyword"
      },
      "session_id": { 
        "type":       "keyword",
        "doc_values": false  // 禁用文档值
      }
    }
  }
}
```

### `Explain`详细信息

指定`explain`为`true`可以查看每次命中的详细计算

```json
GET twitter/_search
{
  "explain": true, 
  "query": {
    "term": {"user": "kimchy"}
  }
}
```

### `collapese` 字段折叠

基于字段的值折叠搜索结果，相当于分组并排序后，取每组第一个文档。比如下面的查询检索每个用户获赞最高的tweet：

```json
GET /twitter/_search
{
    "query": {
        "match": {
            "message": "elasticsearch"
        }
    },
    "collapse" : {
        "field" : "user" // 根据user字段折叠
    },
    "sort": ["likes"], 
}
```



### `From, Size`

from指定偏移，size指定返回的命中数，可以用于分页

```json
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

注意：from + size 不能大于`index.max_result_window`的默认值10000

###`highlight`高亮

基本用法：高亮需要指定字段

```json
GET /_search
{
  "query": {
    "match": {"content": "mate"}
  },
  "highlight": {
    "fields": {"content":{}}  // 使用默认样式高亮content字段
  }
}
```

返回结果如下，默认样式是加`<em>`标签：

```json
{
  "took" : 261,
  "timed_out" : false,
  "_shards" : {
    "total" : 18,
    "successful" : 18,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.8328784,
    "hits" : [
      {
        "_index" : "article",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.8328784,
        "_source" : {
          "content" : "huawei mate pro 销量一飞冲天"
        },
        "highlight" : {
          "content" : [
            "huawei <em>mate</em> pro 销量一飞冲天"
          ]
        }
      },
      {
        "_index" : "article",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.7942397,
        "_source" : {
          "content" : "mate pro 销量以前销量普通"
        },
        "highlight" : {
          "content" : [
            "<em>mate</em> pro 销量以前销量普通"
          ]
        }
      }
    ]
  }
}
```

更多自定义设置参考[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html)

### `indices_boost` 索引提升

为不同的索引（集）设置不同的提升级别，当一个索引的命中结果比其他索引的命中结果更重要时，可以使用。

```json
GET /_search
{
    "indices_boost" : [
        { "index1" : 1.4 },
        { "index2" : 1.3 }
    ]
}
```

### `inner_hits`内部命中

`join`父子字段和`nested`嵌套字段（对象数组）可以返回不同域内匹配的文档。

inner_hits可以告诉你哪个嵌套对象或者父/子文档导致了特定信息被返回。inner_hits可以定义在`nested`, `has_child`, `has_parent`查询和过滤中。

#### nested inner hits

示例：

```json
// 定义test001，comments字段类别为nested
PUT test001
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}

// 索引文档
PUT test/_doc/1
{
  "title": "Test title",
  "comments": [  // 对象数组
    {
      "author": "kimchy",
      "number": 1
    },
    {
      "author": "nik9000",
      "number": 2
    }
  ]
}

// 
POST test001/_search
{
  "query": {
    "nested": {  // nested查询
      "path": "comments",
      "query": {
        "match": {"comments.number" : 2}
      },
      "inner_hits": {} 
    }
  }
}

// 查询结果如下：
{
  "took" : 34,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test001",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Test title",
          "comments" : [
            {
              "author" : "kimchy",
              "number" : 1
            },
            {
              "author" : "nik9000",
              "number" : 2
            }
          ]
        },
        "inner_hits" : {
          "comments" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "test001",
                  "_type" : "_doc",
                  "_id" : "1",
                  "_nested" : {  // 内部命中了哪个对象
                    "field" : "comments",
                    "offset" : 1
                  },
                  "_score" : 1.0,
                  "_source" : {  // 这个资源是 _nested中命中的那个对象
                    "author" : "nik9000",
                    "number" : 2
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

上面的例子中，`_nested`元数据很关键，因为它定义了这个内部命中来自哪个内部嵌套对象：这里是comments字段中，偏移为1的那个嵌套对象。不过，由于排序和打分，命中位置通常和该对象定义时的位置不一致。

特别要注意的是，嵌套对象存储在根文档中，而根文档在`_source`字段下，嵌套对象其实没有`_source`字段。为嵌套对象返回这个字段是有性能开销的，尤其是在`size`和内部命中的`size`设置的比默认值大的时候。为了避免这一点，可以在inner_hits中禁止包含_source字段，而是使用docvalue_fields字段。就像这样：

```json
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.text" : "words"}
      },
      "inner_hits": {
        "_source" : false,
        "docvalue_fields" : [
          "comments.number"
        ]
      }
    }
  }
}
```

#### 多层嵌套字段和内部命中

示例：以下comments嵌套字段中又包含一个votes嵌套字段：

```json
// 定义mapping
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "votes": {
            "type": "nested"
          }
        }
      }
    }
  }
}

// 索引文档
PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}

// inner hits查询
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}
```

返回结果如下：

```json
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.6931472,
        "_source": ...,
        "inner_hits": {
          "comments.votes": { 
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.6931472,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1,
                    "_nested": {
                      "field": "votes",
                      "offset": 0
                    }
                  },
                  "_score": 0.6931472,
                  "_source": {
                    "value": 1,
                    "voter": "kimchy"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

#### 父子内部命中

示例：

```json
PUT test
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "my_parent": "my_child"
        }
      }
    }
  }
}

// 索引父文档
PUT test/_doc/1?refresh
{
  "number": 1,
  "my_join_field": "my_parent"
}

// 索引子文档
PUT test/_doc/2?routing=1&refresh
{
  "number": 1,
  "my_join_field": {
    "name": "my_child",
    "parent": "1"
  }
}

// has_child搜索
POST test/_search
{
  "query": {
    "has_child": {
      "type": "my_child",
      "query": {
        "match": {
          "number": 1
        }
      },
      "inner_hits": {}    
    }
  }
}
```

返回结果：

```json
{
    ...,
    "hits": {
        "total" : {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "test",
                "_type": "_doc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "number": 1,
                    "my_join_field": "my_parent"
                },
                "inner_hits": {
                    "my_child": {
                        "hits": {
                            "total" : {
                                "value": 1,
                                "relation": "eq"
                            },
                            "max_score": 1.0,
                            "hits": [
                                {
                                    "_index": "test",
                                    "_type": "_doc",
                                    "_id": "2",
                                    "_score": 1.0,
                                    "_routing": "1",
                                    "_source": {
                                        "number": 1,
                                        "my_join_field": {
                                            "name": "my_child",
                                            "parent": "1"
                                        }
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        ]
    }
}
```



### 更过查询字段，略

## Search Template

`_search/template`接口允许使用模板字符预渲染搜索请求：

```json
GET /_search/template
{
    "source" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "message",
        "my_value" : "some message",
        "my_size" : 5
    }
}
```

或者：

```JSON
GET _search/template
{
    "source": {
        "query": {
            "term": {
                "message": "{{query_string}}"
            }
        }
    },
    "params": {
        "query_string": "search for these words"
    }
}
```



其他：

* copy_to  可以将多个字段的内容合并到一个新字段，在查询中使用新字段查询。
* 精确值和全文本
  * 精确值不需要做分词的处理，就是ES中的keyword
    * 数字，日期，状态，具体字符串（比如"apple store"），没有必要作分词处理。
  * 全文本会分词，ES中的text







