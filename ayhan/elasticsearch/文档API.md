## Index API

index api用来新增文档，支持如下几种方式：

```shell
# 指定id创建，如果id已存在，则会进行更新，`_version` + 1
PUT {index}/_doc/{id}

# 强制创建，如果id已经存在，409错误(以下二者等价)
PUT {index}/_doc/{id}?op_type=create
PUT {index}/_create/{id}

# POST创建，自动生成ID
POST {index}/_doc/
```

指定ID请求方式为PUT，自动生成ID的请求方式为POST

具体示例：

```ini
// 示例1 往"twitter"中插入文档，指定id是1
PUT twitter/_doc/1
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

结果：

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

`_shards`字段提供了索引操作的副本处理信息：

* `total` 应执行的分片数
* `successful` 成功执行的分片数
* `failed` 失败数

只要`successful`的值至少为1，那么索引操作就是成功了。

id已经存在的情况下会执行更新操作：

```json
PUT twitter/_doc/1
{
    "user" : "Jack2",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

```json
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

### 自动创建索引

通过索引插入文档时，如果索引（集）不存在，比如上述的`twitter`，将会自动创建索引。当然我们可以修改这个设定：

```
PUT _cluster/settings
{
  "persistent": {
    "action.auto_create_index": "false" // false 禁止 true 允许
  }
}
```

`action.auto_create_index`的值支持更为复杂的设定，比如：

```
"action.auto_create_index": "twitter, facebook,-tieba,+topic*"
```

`+`是允许，`-`是禁止，`*`是通配符。

以上配置的含义是，允许为`twitter`, `facebook`, 以及任何匹配`topic*`的自动创建索引，禁止为`tieba`创建索引。

### 操作类别

索引操作可以接收`op_type`参数，来强制进行`create`操作，允许如果确实就修改（put-if-absent）的行为。当指定`create`时，如果文档的id在索引集中已存在，那么索引操作失败。

示例：

```json
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

```json
{
  "error": {
   .......
  },
  "status": 409
}
```

`op_type=create`也可以使用如下方式，二者效果是一样的：

```
PUT twitter/_doc/1?op_type=create  <=>  PUT twitter/_create/1
```

### 自动生成ID

索引操作时如果不指定ID，将自动生成，并且会默认`op_type`是`create`，注意，请求方式是POST

```JSON
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "A6y0umsBAkV3IICsYCLL",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

## Get API

get api根据id，从索引（集）中获取JSON文档，支持如下几种方式

```shell
# 获取文档及元信息
GET {index}/_doc/{id}

# 仅获取文档字段，不包含元信息
GET {index}/_source/{id}
```

以上两种方式都支持字段过滤。

示例：

```json
GET twitter/_doc/1
```

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "user" : "Jack2",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
  }
}
```

如果没找到，返回：

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "found" : false
}

```

我们也可以发一个`HEAD`请求，来查询文档是否存在：

```
HEAD /twitter/_doc/0
200 - OK  # 存在
404 - Not Found  # 不存在
```

### 字段过滤

get操作默认会返回`_source`字段的所有内容一级文档的元信息，除非你指定`stored_fields`参数，或者禁用`_source`字段：

```json
GET twitter/_doc/1?_source=false
```

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true
}
```

如果，你只需要`_source`字段中某几个字段，你可以使用如下两个参数指定：

* `_source_includes` 指定包含字段，字段间以`,`号分隔
* `_source_excludes`指定丢弃字段，字段间以`,`号分隔
* 以上两个参数可以通过`&`连接一起使用

```json
GET twitter/_doc/2?_source_includes=user,message&_source_excludes=date
```

如果只是需要指定包含字段，可以简化为`_source`：

```shell
GET twitter/_doc/1?_source=user,message
```

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "message" : "trying out Elasticsearch",
    "user" : "Jack2"
  }
}
```

字段过滤可以节省网络开销。

### Stored Fields

创建索引（集）facebook, 并定义mappings

```json
put facebook
{
  "mappings": {
    "properties": {
      "counter": {
        "type": "integer",
        "store": false
      },
      "tags": {
        "type": "keyword",
        "store": true
      }
    }
  }
}
```

插入一个文档：

```JSON
PUT facebook/_doc/2
{
  "counter": 1,
  "tags": ["red"]
}
```

检索刚插入的文档，通过`stored_fields`参数指定字段：

```shell
GET facebook/_doc/2?stored_fields=tags,counter
```

```json
{
  "_index" : "facebook",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "fields" : {
    "tags" : [
      "red"
    ]
  }
}
```

由于`counter`字段在mappings中`store: false`，当进行检索时，将忽略该字段。

### 仅获取`_source`内容

使用`{index}/_source/{id}`可以仅获取文档的`_source`字段内容：

```shell
GET twitter/_source/1
```

```json
{
  "user" : "Jack2",
  "post_date" : "2019-05-15T14:12:12",
  "message" : "trying out Elasticsearch"
}
```

也可以通过`HEAD`请求检测文档的`_source`是否存在。另外，如果在mapping中禁用了`_source`，存在的文档也不存在`_source`

```shell
HEAD twitter/_source/1
```

## Multi Get API

multi get 支持一次请求，查询多个文档，形式如下：

```shell
# 可以从多个索引中查询，需要分别指定_index和_id
GET /_mget

# 指定索引，可以直接指定ids
GET /{index}/_mget
```

示例1：

```json
GET /_mget
{
  "docs": [
    {
      "_index": "twitter",
      "_id": "1"
    },
    {
      "_index": "facebook",
      "_id": "2"
    }
    ]
}
```

说明：

* 查询条件放在`docs`字段的列表中
* 可以基于index, id进行查询。基于type查询`"_type" : "_doc"` 在7.x中已废弃，不需要传_type条件
* GET `/_mget`或者`_mget`都行（其他请求也是，`/`不影响）

返回结果放在`docs`字段的列表中

```json
{
  "docs" : [
    {
      "_index" : "twitter",
      "_type" : "_doc",
      "_id" : "1",
      "_version" : 2,
      "_seq_no" : 2,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "user" : "Jack2",
        "post_date" : "2019-05-15T14:12:12",
        "message" : "trying out Elasticsearch"
      }
    },
    {
      "_index" : "facebook",
      "_type" : "_doc",
      "_id" : "2",
      "_version" : 1,
      "_seq_no" : 1,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "counter" : 1,
        "tags" : [
          "red"
        ]
      }
    }
  ]
}

```

示例2：

```json
GET twitter/_mget
{
  "docs": [
    {
      "_id": "1"
    },
    {
      "_id": "2"
    }]
}
```

在上面这种情况，只根据id过滤时，可以简写如下：

```json
GET twitter/_mget
{
  "ids": ["1", "2"]  // ids字段
}
```

### 字段过滤

可以通过`_source`参数指定返回的字段：

```json
GET _mget
{
  "docs": [
    {
      "_index": "twitter",
      "_id": "1",
      "_source": false
    },
    {
      "_index": "twitter",
      "_id": "2",
      "_source": ["user", "message"]
    },
    {
      "_index": "facebook",
      "_id": "2",
      "_source": {
        "include": ["counter"],
        "exclude": []
      }
    }
    ]
}
```

也可以通过`stored_fields`来指定。

## Update API

update api允许基于脚本更新文档，每次修改后，`_version` + 1， 基本方式：

```shell
# 基于script更新
POST {index}/_update/{id}
{
    "script": {...}
}

# 基于文档更新，doc中的字段将会自动与目标文档合并
POST {index}/_update/{id}
{
    "doc": {...}
}
```

### 脚本更新

首先我们插入一个文档：

```json
PUT facebook/_doc/3
{
  "counter": 1,
  "tags": ["red"]
}
```

下面执行更新脚本：

script 1: 增加counter

```json
POST facebook/_update/3
{
  "script": {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",  // 使用painless函数
    "params": {
      "count": 4
    }
  }
}
```

script 2: 添加tags元素

```json
POST facebook/_update/3
{
  "script": {
    "source": "ctx._source.tags.add(params.tag)",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```

script 3: 移除tags元素

```json
POST facebook/_update/3
{
  "script": {
    "source": "if(ctx._source.tags.contains(params.tag)){ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag))}",
    "lang": "painless",
    "params": {
      "tag": "red"
    }
  }
}
```

查看最终结果：

```json
{
  "_index" : "facebook",
  "_type" : "_doc",
  "_id" : "3",
  "_version" : 4,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "counter" : 5,
    "tags" : [
      "blue"
    ]
  }
}
```

### 文档更新

更新请求体中使用`doc`字段传递一个文档，该文档就会与目标文档进行合并

```json
POST facebook/_update/3
{
  "doc": {
    "name": "wahaha"
  }
}
```

由于目标文档不存在name字段，合并后将会新增name字段。

### noop更新

如果更新没有改变任何东西，将返回`noop`结果，`_version`不变。比如讲上述文档更新执行两次，第二次将得到如下结果：

```json
{
  "_index" : "facebook",
  "_type" : "_doc",
  "_id" : "3",
  "_version" : 5,
  "result" : "noop",
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "failed" : 0
  }
}
```

当然你也可以禁止`noop`结果：

```json
POST facebook/_update/3
{
  "doc": {
    "name": "wahaha"
  },
  "detect_noop": false  // 禁止
}
```

再次执行同样的更新，`_version` 将 +1

###   Upserts

存在则更新，不存在则创建：

```json
POST facebook/_update/5
{
  "script": {  // 更新脚本
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params": {
      "count": 4
    }
  },
  "upsert": {  // 如果目标文档不存在，以upsert中的内容创建新的文档
    "counter": 1
  }
}
```

#### scripted_upsert

如果希望脚本不论目标文档是否存在都执行，可以指定`scripted_upsert`为`true`，这样脚本将代替upsert字段执行初始化文档。官网示例本地无法执行，暂不讨论。

#### doc_as_upsert

适用于文档更新的upsert操作：

```json
POST facebook/_update/6
{
  "doc": {
    "name": "wahaha"
  },
  "doc_as_upsert": true
}
```

### 请求参数

更新操作还支持查询字符串参数，具体参见官网：`<https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update.html#_parameters_2>`

## Update By Query API

`_update_by_query`适用于执行对索引（集）中全部文档的更新，比如添加新的字段，或者其他mapping更改。该操作也可以指定条件，只对特定文档进行更新。

```shell
POST {index}/_update_by_query
```

<https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html>

## Delete API

用于从索引中删除文档，用法如下：

```shell
DELETE {index}/_doc/{id}
```

比如，从twitter中删除id为1点文档：

```shell
DELETE twitter/_doc/1
```

返回响应：

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 6,
  "_primary_term" : 1
}
```

## Delete By Query API

该接口可以实现删除一个索引中的所有文档：

```shell
POST {index}/_delete_by_query
```

当然也支持指定筛选条件，实现部分删除。
<https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-delete-by-query.html>

## Bulk API

bulk api允许在一次接口调用中，实现多个增删改操作

```json
POST /_bulk
{ "index" : { "_index" : "facebook", "_id" : "2" } }
{ "connter" : 2 }  // index的source
{ "delete" : { "_index" : "twitter", "_id" : "2" } }
{ "create" : { "_index" : "twitter", "_id" : "7" } }
{ "name" : "seven" }  // create的source
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }  // update的doc
```

说明：

* index 和 create操作需要在紧接着的下一行提供source，以便进行添加文档，或者更新
* delete不需要source
* update需要在下一行指定doc，upsert,  script等细节

返回结果：

```json
{
  "took" : 217,
  "errors" : true,
  "items" : [
    {
      "index" : {  // 第一个index因为id已经存在，于是根据source更新了文档的内容
        "_index" : "facebook",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 4,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 14,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "delete" : {  // 第二个delete没找到目标文档
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "not_found",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 8,
        "_primary_term" : 1,
        "status" : 404
      }
    },
    {
      "create" : {  // 第三个以给出的source成功创建文档
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "7",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 9,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "update" : {  // update指定的index不存在，返回404
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "status" : 404,
        "error" : {
          "type" : "document_missing_exception",
          "reason" : "[_doc][1]: document missing",
          "index_uuid" : "k7zYEGp8ROuGWkfOprCKvQ",
          "shard" : "0",
          "index" : "test"
        }
      }
    }
  ]
}

```

## Reindex API

reindex最基本的用法是将一个索引中的文档拷贝到另一个文档：

```json
POST _reindex
{
  "source": {
    "index": "{index1}"
  },
  "dest": {
    "index": "{index2}"
  }
}
```

<https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html>

## 并发控制

Elasticsearch是分布式的。创建，修改，删除文档后，新版的文档必须复制到集群内的其他节点。Elasticsearch同时也是异步和并发的，意味着这些复制请求是并行发送的，不能保证到达目的地的顺序。因此ES需要一种方式确保旧版本的文档不会覆盖较新的文档。

对文档的每次操作都会由主分片（primary shard）分配一个序号，每次操作文档都会增大序号。ES通过序号判断文档的新旧，保证旧文档不会覆盖新文档。

GET获取文档时，可以查看到序号：

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 5,  // 序号
  "_primary_term" : 1, // 主条目
  "found" : true,
  "_source" : {
    "user" : "Jack2",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
  }
}
```

因此，通过记下返回的`_seq_no`和`_primary_term`，可以确保你仅在检索到文档且未有其他任何对该文档的更改时，才更改该文档，比如：

```json
DELETE twitter/_doc/2?if_seq_no=5&if_primary_term=1
```

如果在删除前，发生了对该文档的其他修改，删除操作将会失败：409响应，并提示版本冲突。

## 关于文档读写

### 基本写模型

ES中的每个索引操作首先通过路由（routing）解析到一个副本组，这通常是基于文档ID。副本组确定后，操作将在内部转发到当前的主分片上。主分片负责验证操作并转发给其他副本。因为副本可以脱机，所以主分片不必复制给所有副本。事实上，es会维护一个接收操作的副本列表，称之为同步副本，并由主节点维护。主分片必须保证将所有的操作复制到同步副本中每个副本。主分片遵循如下基本流程：

1. 验证操作，比如字段是否相符
2. 本机执行操作，比如索引或删除相关文档，必要时也会拒绝，比如一个keyword值过长，以至于无法在Lucene中索引
3. 转发操作给当前同步副本组中的每个副本（并行操作）
4. 一旦所有副本成功执行了操作并报告给主分片，主分片就会确认成功完成了客户端的请求。

#### 失败处理

索引时可能会出错，比如磁盘故障，节点丢失，或者配置错误。主分片需要对此进行处理。

如果主分片自身故障了，那么其所在的节点将通知主节点。索引操作将最多等待1分钟，直到主节点将某个副本提升为新的的主分片。然后操作被转发到新的主分片进行处理。

在主分片成功执行了操作，副本出错的情况下（比如副本本身故障或者网络问题），主分片将请求主节点将故障副本从同步副本中移除。

### 基本读模型

在ES中，读取可以是根据id的非常轻量级的查找，也可以是具有复杂聚合，占用cpu的搜索请求。主备模型的优点是所有分片副本保持一致，因此每个副本都能提供读取请求。

处理当前客户端请求的节点称为协调节点（coordinating node），基本流程如下：

1. 解析请求到相关的分片上。因为大多数搜索会被发送到一个或多个索引，因此需要从多个分片进行读取，每个分片代表数据的不同子集。
2. 从相关分片的副本组中选取一个副本，这可以是主分片，也可以是副本分片（默认情况下es在会轮流选择）
3. 发送读取请求给选中的副本
4. 合并结果并响应。如果是通过id查找，因为只有一个相关分片，所以这步会被跳过。

#### 分片失败

当某个分片响应读取请求时，协调节点将请求发往同一副本组的另一个分片。多次失败会导致没有可用的分片。

为了确保快速响应，下列API在出现分片失败时，会返回部分结果：

* Search
* Multi Search
* Bulk
* Multi Get

这时仍然是200响应，但是通过`time_out`和`_shards`字段可以知道出现了分片失败。











