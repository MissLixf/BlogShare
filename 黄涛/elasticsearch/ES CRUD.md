## 总览

| 类别   | 请求方式                                                     |
| ------ | ------------------------------------------------------------ |
| Index  | PUT `my_index/_doc/1`                                        |
| Create | PUT `my_index/_create/1` 指定ID<br />POST `my_index/_doc` 不指定ID，自动生成 |
| Read   | GET `my_index/_doc/1`                                        |
| Update | POST `my_index/_update/1`                                    |
| DELETE | DELETE `my_index/_doc/1`                                     |

说明：

* Create PUT请求可以指定ID，POST则自动生成ID
* Index和Create区别，当使用Index索引文档时：
  * 如果文档不存在，创建新文档；如果存在，会删除原来的文档，重新创建，并对version +1
* Update 的请求体中需要指定`doc`字段

## Index API

新增或更新文档

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

```

```



