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

### Doc value Fields 文档值字段

什么是`doc values`？

大部分字段会被默认索引，以备查询。倒排索引允许查找词条，并找到包含词条的相关文档。但是排序，聚合，以及在脚本中访问字段的值要使用的不同的数据访问模式，我们需要查找文档并找到字段中的词条，而不是先找到词条，再找到文档。

其他：

* copy_to  可以将多个字段的内容合并到一个新字段，在查询中使用新字段查询。
* 精确值和全文本
  * 精确值不需要做分词的处理，就是ES中的keyword
    * 数字，日期，状态，具体字符串（比如"apple store"），没有必要作分词处理。
  * 全文本会分词，ES中的text







