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

* match 查询
* term 查询

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
'

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

基于字段的值折叠搜索结果。比较复杂

### `From, Size`

from指定偏移，size指定返回的命中数，可以用于分页

###`highlighte`高亮

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

返回父文档基于子文档的匹配，或返回子文档基于父文档的匹配，在这种嵌套的情况下，文档的返回基于嵌套内部对象的匹配。

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
                  "_nested" : {
                    "field" : "comments",
                    "offset" : 1
                  },
                  "_score" : 1.0,
                  "_source" : {  // 
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







