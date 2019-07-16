mapping是定义文档及其字段是如何存储和索引的程序。例如，使用mapping定义：

* 哪个字符串字段应该视为全文字段
* 哪个字段包含数字，日期，或地理位置
* 日期的格式
* 自定义规则来控制动态添加字段

## mapping type

每个索引都有*mapping  type*来决定文档如何被索引。mapping type包含：

* meta-fields: 比如`_index`, `_type`, `_id`, `_source`字段，用来定制文档相关的元数据
* fields or properties：相关文档的字段列表或者`properties`

## 字段数据类型

每个字段都有数据类型，可以是如下：

* 简单的类型：text, keyword, date, long, double, boolean,  ip
* 支持JSON这种层次结构的：object, nested
* 专门的类型：geo_point, geo_shape, completion

根据不同的目的以不同的方式来索引相同的字段通常很有用。比如，同样的字段可以索引为`text`类型以便进行全文搜索，也可以作为`keyword`类型用来排序或聚合。或者同样的值使用不同的分词器。

大部分字段都支持`fields`参数的多字段搜索。

### 核心数据类型

* 字符串

  ```json
  PUT my_index
  {
    "mappings": {
      "properties": {
        "full_name": {
          "type":  "text"
        }
      }
    }
  }
  ```

  * text  用于全文索引，会被分词处理。适用于文章内容，产品介绍之类的信息。可以接收以下参数：

    * analyzer 指定分词器，同时适用于索引时和搜索时（除非被search_analyzer覆盖），默认值是索引默认的分词器，或者standard分词器
    * boost 字段级别的查询时提升，接收浮点数，默认是1.0
    * eager_global_ordinals 刷新时是否重新加载全局顺序？默认是`false`。建议对经常用于词条聚合的此段启动该特性。
    * fielddata  该字段是否能使用内存字段数据用来排序，聚合，或者脚本计算？默认`false`
    * fielddata_frequency_filter 专家设置：当fielddata启用时，是否允许决定哪些值被载入内存，默认情况下加载所有值
    * fields 多字段（multi-fields）允许出于不同的目的以多种方式来索引相同的字段，比如一个字段用于搜索，一个多字段用于排序和聚合，或者对同样的字符串使用不同的分词器
    * index 字段是否可被搜索，默认`true`
    * index_options 索引中应该存储什么信息用于搜索和高亮，默认是`positions`
    * index_prefixes 如果启用，2~5个字符的词条前缀将会被索引为一个单独的字段，有助于提高前缀搜索的效率。
    * index_phrases 如果允许，词组会被索引为单独的字段。这使得精确的短语查询更高效。注意，只有当停用词没有被移除时，这个才最有效，因为包含停用词的短语不会使用子字段，而是使用标准的短语查询。默认值是`false`
    * norms 打分时是否考虑字段长度，默认`true`
    * position_increment_gap 略
    * store  字段值是否在_source字段外，被独立存储和检索
    * search_analyzer 指定搜索时的分词器，默认是analyzer 参数的设置
    * search_quote_analyzer 没啥用
    * similarity 使用哪种打分算法，默认是`BM25`
    * term_vector 略

  * keyword  关键词，适用于标签，状态码，邮件地址，域名，商标等。用于过滤，排序，和聚合，关键词只支持精确搜索。

    ```json
    PUT my_index
    {
      "mappings": {
        "properties": {
          "tags": {
            "type":  "keyword"
          }
        }
      }
    }
    ```

    支持的参数如下：

    * ignore_above 默认是`2147483647`，超过这一长度的字符串无法索引。但是默认的动态mapping会覆盖这个值为`ignore_above:256`
    * 其他和text类似，具体参见[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)

* 数字
  * long  -2^63 ~ 2^63-1

  * integer  -2^31 ~ 2^31-1

  * short -32768 ~ 32768

  * byte -128 ~ 127

  * double 64位双精度浮点数

  * float 32位双精度浮点数

  * half_float  16位双精度浮点数

  * scaled_float  略

    ```json
    PUT my_index
    {
      "mappings": {
        "properties": {
          "number_of_bytes": {
            "type": "integer"
          },
          "time_in_seconds": {
            "type": "float"
          },
          "price": {
            "type": "scaled_float",
            "scaling_factor": 100
          }
        }
      }
    }
    ```

  * 数字类型的字段支持以下参数：

    * coerce  强制转换字符串为数字，截断分数为整数，默认是`true`
    * 其他参数参考[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)

* date，JSON没有日期类型，因此ES中的日期可以是

  * 包含日期格式的字符串，比如：`"2015-01-01"` or `"2015/01/01 12:10:30"`
  * 时间的毫秒数
  * 时间的整数

  在ES内部，日期会被转换为UTC时间（如果指定了时区），并且存储为代表时间毫秒数的长整数。对于日期的查询会被转化为对这个长整数的范围查询，结果再转化为以该字段定义的日期格式的字符串。日期格式可以自定义，如果没有指定`format`，会使用如下默认值：

  ```
  "strict_date_optional_time||epoch_millis"
  ```

  这意味它可以接受[strict_date_optional_time](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#strict-date-time)支持的格式，或者毫秒数，例如：

  ```json
  PUT my_index
  {
    "mappings": {
      "properties": {
        "date": {
          "type": "date" // 不指定format，那就使用上面提到的默认值
        }
      }
    }
  }
  
  PUT my_index/_doc/1
  { "date": "2015-01-01" }   // 纯日期
  
  PUT my_index/_doc/2
  { "date": "2015-01-01T12:10:30Z" } //日期+时间
  
  PUT my_index/_doc/3
  { "date": 1420070400001 }  // 毫秒数
  
  GET my_index/_search
  {
    "sort": { "date": "asc"} 
  }
  ```

  自定义多种日期格式：

  ```json
  PUT my_index
  {
    "mappings": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
  ```

  

* 布尔

  * boolean

* 二进制

  * binary

* 范围

  * integer_range
  * float_range
  * long_range
  * double_range
  * date_range

### 复杂数据类型

* 对象
  * object
* 嵌套
  * nested

### 地理信息数据类型

* geo_point
* geo_shape

### 专门数据类型

* ip
* completion
* token_count：字符串中的提条的数量
* murmur3  索引时计算内容的哈希值并存储在索引中
* annotated-text 索引包含特殊标记的文本（用于标识命名实体）
* join 定义索引中文档的父子关系
* alias 已存在字段定义别名
* rank  记录数字特征来提高查询的命中
* vector 向量

### 数组

ES中，数组不需要专门的字段。默认情况下，任何字段都可以包括零个或多个值，但是所有的值必须具有相同的数据类型。

## 动态mapping

字段和mapping类型不需要事先定义，因此在索引文档时，新字段的名字将会自动添加（可以添加到根字段，或者是object和nested子字段）

## 详尽mapping

比起Elasticsearch的“猜测”，你更清楚你的数据。所以一开始可以用动态mapping，到某个阶段后，你可以自己定制mapping。

你可以在创建索引（集）时创建字段mapping，然后通过PUT mapping API来向已经存在的索引（集）增加字段。

## 更新现存字段的mapping

除了文档之外，**现存字段的mapping无法更新**。更新字段意味着已经索引的文档会失效。相反，你应该用正确的mapping创建一个新的索引，然后reindex数据到新的索引中。如果你只是想重命名某个字段而不改变mapping，可以使用`alias`字段。

## 示例

创建索引时指定mapping：

```json
PUT my_index 
{
  "mappings": {
    "properties": {
      "title":    { "type": "text"  }, 
      "name":     { "type": "text"  }, 
      "age":      { "type": "integer" },  
      "created":  {
        "type":   "date", 
        "format": "strict_date_optional_time||epoch_millis"
      }
    }
  }
}
```

## 废除mapping types

7.0之后废除，原先需要指定type的地方，都用`_doc`代替。

由于第一版ES中，所有的文档存储在单个索引中，比如`user`类型的文档和`blog`类型的文档，他们的数据结构不同，为了区分，就分别定义了user类型，和blog类型。每个文档都有一个`_type`元字段来标明类型，搜索时也需要指定类型名。另外，在单个索引中，不同的类型的文档可以拥有相同的`_id`值，因此要唯一标识一个资源，需要通过`_type`和`_id`

更多信息查看具体查看[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)