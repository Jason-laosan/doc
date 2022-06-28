# ElasticSearch 版本号 7.12学习

> 命令行可以用 postman、kibana、curl，建议用kibana进行学习和练习

### 创建索引

```json
PUT student
{
  "mappings": {
    "properties": {
      "uid":{
        "type":"integer"
      },
      "name":{
        "type":"keyword"
      },
      "age":{
        "type":"integer"
      }
    }
  },
  "settings": {
    "index":{
      "number_of_shards":10,
      "number_of_replicas":1
    }
  }
}
```

### 创建文档

```json
POST student/_doc/1?routing=1
{
  "uid":1,
  "name":"张三",
  "age":10
}
```



### 获取文档

```json
GET student/_search
{
  "query":{
    "match": {
      "uid": 1
    }
  },
  "explain": true
}
```



### 创建文档

```json
POST student/_doc/1?routing=2
{
  "uid":1,
  "name":"张三",
  "age":10
}
```

### 查看索引

```json
GET student
```

### 删除索引

```json
DELETE student
```

### 创建索引时指定Mapping

``` json
PUT school
{
  "mappings": {
    "properties": {
      "name":{
        "type": "keyword"
      }
    }
  }
}
```

### 删除索引

```json
DELETE school
```




### 建立索引时指定Mapping和Settings
``` jsonshell
PUT school
{
  "mappings": {
    "properties": {
      "name":{
        "type":"keyword"
      }
    }
  },
  "settings": {
    "index":{
      "number_of_shards":1,
      "number_of_replicas":2
    }
  }
}
```



### ES 6 中在创建索引时指定 Mapping 和 Setting

> 因为 ES 7 之前的版本，一个index下支持多个type，所以`mappings` 下一层要指定 `type`。

```json
PUT school
{
  "mappings" : {
    "student": {
      "properties" : {
        "name" : {
          "type" : "keyword"
        }
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 2
    }
  }
}
```

## 设置索引副本数量和分片数量
### 默认分片数、副本数
``` json
PUT movie
```



### 获取索引信息

```json
GET movie
```

>  可以看到，分片数量`number_of_shards` 为1，副本数量为`number_of_replicas` 1 

### 修改副本数量

```json
PUT movie/_settings
{
  "index":{
    "number_of_replicas":3
  }
}
```

> 注意，当副本创建完成后（可能耗时很长），才会响应。如果耗时很长，可以不用等待响应。过一段时间后去查询索引信息确认即可。

### 修改分片数量
> **分片数量必须在创建索引时指定，创建后无法修改，否则会报错**

```json
PUT movie/_settings
{
  "index":{
    "number_of_shards":3
  }
}
错位信息返回
response:
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Can't update non dynamic settings [[index.number_of_shards]] for open indices [[movie/ZJEZuQhsQ--aGqGlHHvyvg]]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "Can't update non dynamic settings [[index.number_of_shards]] for open indices [[movie/ZJEZuQhsQ--aGqGlHHvyvg]]"
  },
  "status" : 400
}

```

## 查看索引信息命令

### 查看所有索引
```json
GET _cat/indices?v
```



### 查看s开头的索引
```json
GET _cat/indices/s*?v
```



## 所有的数据类型

[点击查看官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping-types.html)

### 字符串类型 **keyword**、**text**

> **ES5 之前，字符串类型是`string`，5.0版本之后开始废弃，引入 `keyword`、`text`**

#### 两者的区别:

- `keyword`不支持全文搜索。所以，只能使用精确匹配查询，例如`iterm`查询
- `text`默认支持全文检索

####  长度区别

- `keyword `的最长长度是 32766 字节。（原因应该是底层lucene做倒排索引时，限制了单词的长度。`UTF-8`中，英文字母是1个字节，中文一般是3个字节，表情符号是4个字节）
- `text` 无长度限制。（但被分析器处理后的单词不应该超过 32766 个字节。-> 这是我推论出来的，待验证）

### 数组

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/array.html)

在 ES 中，没有专用的数组类型，任何字段都可以变成数组。

#### 创建索引

```json
PUT student
{
  "mappings":{
    "properties": {
      "scores":{
        "type":"integer"
      }
    }
  }
}	
```

#### 插入数据

> 使用_bulk 创建文档，注意文档2的scores是数组

```json
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "scores" : 80 }
{ "index" : { "_index" : "student", "_id" : "2" } }
{ "scores" : [80, 90] }
```

#### 查看数据

```json
GET student/_search

## 查询会命中包含90的
GET student/_search
{
  "query":{
    "term": {
      "scores": 90
    }
  }
}
```

#### 为文档1的scores字段添加一个值

```json
POST student/_update_by_query
{
  "query": { 
    "match": {
      "_id": 1
    }
  },
  "script": {
    "source": "if (ctx._source.containsKey(\"scores\")) {if (ctx._source.scores instanceof List) ctx._source.scores.add(88); else ctx._source.scores = [ctx._source.scores, 88];} else {ctx._source.scores = [88]}"
  }
}
```

关于 script 语法的介绍：

- https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html
- https://www.elastic.co/guide/en/elasticsearch/painless/7.2/painless-lang-spec.html

### 添加和更新文档

#### 创建索引

```json
PUT student
{
  "mappings": {
    "properties": {
      "name":{
        "type":"keyword"
      },
      "age":{
        "type":"integer"
      }
    }
  },
  "settings": {
    "index":{
      "number_of_replicas":0,
      "number_of_shards":1
    }
  }
}
```

#### 使用 `POST`

```json
POST student/_doc
{
  "name":"张三"
}

POST student/_doc/2
{
  "name":"李四"
}
```



```json
POST student/_doc/2
{
  "age":2
}
```

结果是 version 从1变成了2，而 name 字段不见了。

> 原因是 `POST student/_doc/2` 这种语法的效果是覆盖数据。可以理解为先把原文档删除，再索引新文档。

`POST student/_doc/2` 的效果相同。

#### 使用 `_update` 更新文档

如何在 name 不消失的情况下更新 age 呢？用 `_update`。

```json
##更新为原始数据
POST student/_doc/2
{
  "name":"李四"
}
7.X版本写法
更新字段
POST student/_update/2
{
  "doc":{
    "age":11
  }
}
```

使用 `_update` 时，ES 做了下面几件事：

1. 从旧文档构建 JSON
2. 更改该 JSON
3. 删除旧文档
4. 索引一个新文档

```json

# 7.X以下版本
POST student/_doc/2/_update
{
  "doc": {
    "age": 10
  }
}

#! Deprecation: [types removal] Specifying types in document update requests is deprecated, use the endpoint /{index}/_update/{id} instead.

```

#### 使用`_update_by_query`更新文档

```json
POST student/_update_by_query
{
  "query": {
    "match": {
      "_id": "2"
    }
  },
  "script": {
    "source": "ctx._source.age=13"
  }
}

```

### `_bulk`批量添加文档

```json
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三", "age": 12 }
{ "index" : { "_index" : "student", "_id" : "2" } }
{ "name" : "李四", "age": 10 }
{ "index" : { "_index" : "student", "_id" : "3" } }
{ "name" : "王五", "age": 11 }
{ "index" : { "_index" : "student", "_id" : "4" } }
{ "name" : "陈六", "age": 11 }
```

`_bulk`也支持 delete、update 等操作。具体可参考官方文档：[Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-bulk.html) 。

###  `from`、`size`进行分页查询

```json
GET student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_id": {
        "order": "desc"
      }
    }
  ],
  "from":0,
  "size": 2
}
```

#### 使用`sort`进行排序

```json
GET  student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      },
      "_id":{
        "order": "asc"
      }
    }
  ]
}
```

#### 多字段进行排序

```json
准备数据
POST _bulk
{ "index" : { "_index" : "student", "_id" : "5" } }
{ "name" : "张三", "score": [80, 90]}
{ "index" : { "_index" : "student", "_id" : "6" } }
{ "name" : "李四", "score": [78, 95] }
```

按照score的最小值进行排序

```json
GET student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "score": {
        "order": "asc","mode": "min"
      }
    }
  ]
}
```

### 查询结果只展示部分字段
```json
GET student/search
{
  "query":{
    "match": {
      "age": "11"
    }
  },
  "_source":{
      "includes":[
        "name"
        ]
    }
}
```

> 将 includes 换成 include 也可以，但会提示 include 已经废弃。所以不建议使用 include 。

### 查询结果中展示`_version`	字段

```json
### 查询结果只展示部分字段
GET student/_search
{
  "query":{
    "match": {
      "age": "11"
    }
  },
  "_source":{
      "includes":[
        "name"
        ]
  },
  "version": true
}
```

### 使用ignore_version 限制字符串长度

创建 mapping 时，可以为字符串（专指 keyword） 指定 `ignore_above` ，用来限定字符长度。超过 `ignore_above` 的字符会被存储，但不会被索引。

注意，是字符长度，一个英文字母是一个字符，一个汉字也是一个字符。

在动态生成的 mapping 中，`keyword`类型会被设置`ignore_above: 256`。

`ignore_above` 可以在创建 mapping 时指定。

```json
PUT my_index
{
  "mappings" : {
    "properties" : {
      "note" : {
        "type" : "keyword",
        "ignore_above": 4
      }
    }
  }
}
```

使用 `_bulk` 创建文档

```json
POST _bulk
{ "index" : { "_index" : "my_index", "_id" : "1" } }
{ "note" : "一二三"}
{ "index" : { "_index" : "my_index", "_id" : "2" } }
{ "note" : "一二三四"}
{ "index" : { "_index" : "my_index", "_id" : "3" } }
{ "note" : "一二三四五"}
```

使用下面的指令可以查询所有数据：

```plain
GET my_index/_search
```

Copy

可以看到，上面创建的三个文档都存起来了。

我们用下面的查询验证 `ignore_above`：

```json
 能查到数据
GET my_index/_search
{
  "query": {
    "match": {
      "note": "一二三"
    }
  }
}

# 能查到数据
GET my_index/_search
{
  "query": {
    "match": {
      "note": "一二三四"
    }
  }
}

# 不能查到数据
GET my_index/_search
{
  "query": {
    "match": {
      "note": "一二三四五"
    }
  }
}
```

#### 能够修改 ignore_above 吗 ？

可以通过下面的方式改：

```plain
PUT my_index/_mappings
{
  "properties" : {
    "note" : {
      "type" : "keyword",
      "ignore_above": 2
    }
  }
}
```

#### text 类型支持 ignore_above 吗？

不支持。

```json
# 删除索引
DELETE my_index

# 尝试重建索引，note字段为text类型，并指定了 ignore_above，执行时会报错
PUT my_index
{
  "mappings" : {
    "properties" : {
      "note" : {
        "type" : "text",
        "ignore_above": 2
      }
    }
  }
}

# 报错结果如下
{
  "error": {
    "root_cause": [
      {
        "type": "mapper_parsing_exception",
        "reason": "Mapping definition for [note] has unsupported parameters:  [ignore_above : 2]"
      }
    ],
    "type": "mapper_parsing_exception",
    "reason": "Failed to parse mapping [_doc]: Mapping definition for [note] has unsupported parameters:  [ignore_above : 2]",
    "caused_by": {
      "type": "mapper_parsing_exception",
      "reason": "Mapping definition for [note] has unsupported parameters:  [ignore_above : 2]"
    }
  },
  "status": 400
}
```

### 动态映射

> dynamic用来配置处理新出现字段的行为，有true，false，strict 三种。
> true 是默认值，会自动在 mapping 中添加字段，并在文档中保存新字段的值。
> false 代表不会在 mapping 中添加字段，但会将数据存起来。
> strict 代表既不会新增字段，也不会保存新字段的内容，遇到新字段直接报错。 另外，对于嵌套类型的字段，可以单独设置。

默认情况下，我们向索引中写入数据，如果有些字段之前未定义过，ES会自动猜测其类型，然后在mapping中生成。

举个例子：

```json
PUT student
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword",
        "doc_values": false
      }
    }
  },
  "settings": {
    "number_of_shards": "8",
    "number_of_replicas": "2"
  }
}
```

### 动态映射

```json
#创建索引
#默认dynamic为true
PUT student
{
  "mappings": {
  	"dynamic": "true",
    "properties": {
      "name":{
        "type": "keyword",
        "doc_values": false
      }
    }
  },
  "settings": {
    "number_of_replicas": "2",
    "number_of_shards": 8
  }
}

#批量添加数据

POST bulk
{ "index" : { "index" : "student", "_id" : "1"} }
{ "name" : "张三", "age" : 12}

```

```json
GET student/_search

# 查询结果
{
  "took" : 18,
  "timed_out" : false,
  "_shards" : {
    "total" : 8,
    "successful" : 8,
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
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三",
          "age" : 12
        }
      }
    ]
  }
}

```

查看`mapping`

```json
GET student/_mapping
```

age 出现在了 mapping 中，且自动判断为 long 类型。

有时，自动判断的类型可能是不符合我们的预期的，但是呢，类型一旦确定，又是不能更改的。所以我们要谨慎对待这种特性。

#### `dynamic` 为false

dynamic 可以用来配置相关行为。

```json
PUT student
{
  "mappings": {
    "dynamic": "false",
    "properties": {
      "name": {
        "type": "keyword",
        "doc_values": false
      }
    }
  },
  "settings": {
    "number_of_shards": "8",
    "number_of_replicas": "2"
  }
}
```

插入数据：

```json
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三", "age": 12 }
```

查询：

```json
GET student/_search

# 查询结果
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 8,
    "successful" : 8,
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
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三",
          "age" : 12
        }
      }
    ]
  }
}
```

查看 mapping:

```json
GET student/_mapping

# 查询结果
{
  "student" : {
    "mappings" : {
      "dynamic" : "false",
      "properties" : {
        "name" : {
          "type" : "keyword",
          "doc_values" : false
        }
      }
    }
  }
}
```

age 没有出现在 mapping 中。

#### dynamic 为 strict 时

```json
PUT student
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "keyword",
        "doc_values": false
      }
    }
  },
  "settings": {
    "number_of_shards": "8",
    "number_of_replicas": "2"
  }
}
```

插入数据：

```json
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三", "age": 12 }
```

会报错：

```json
{
  "took" : 29,
  "errors" : true,
  "items" : [
    {
      "index" : {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "status" : 400,
        "error" : {
          "type" : "strict_dynamic_mapping_exception",
          "reason" : "mapping set to strict, dynamic introduction of [age] within [_doc] is not allowed"
        }
      }
    }
  ]
}
```

查询：

```json
GET student/_search

# 查询结果为空
```

查看 mapping:

```json
GET student/_mapping

# 查询结果
{
  "student" : {
    "mappings" : {
      "dynamic" : "strict",
      "properties" : {
        "name" : {
          "type" : "keyword",
          "doc_values" : false
        }
      }
    }
  }
}
```

#### 单独设置嵌套类型字段

可参考 [这篇文章](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html).

##  聚合查询

```json
#准备数据
PUT student
{
  "mappings" : {
    "properties" : {
      "name" : {
        "type" : "keyword"
      },
      "age" : {
        "type" : "integer"
      },
      "height": {
        "type": "integer"
      }
    }
  }
}

POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三", "age": 12 }
{ "index" : { "_index" : "student", "_id" : "2" } }
{ "name" : "李四", "age": 10,  "height": 112 }
{ "index" : { "_index" : "student", "_id" : "3" } }
{ "name" : "王五", "age": 11, "height": 108 }
{ "index" : { "_index" : "student", "_id" : "4" } }
{ "name" : "陈六", "age": 11, "height": 111 }

GET student/_search


```

查询总数

```json
GET student/_count

#11岁的总人数 查询结果集换_search
POST student/_count
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "age":11
          }
        }
      ]
    }
  }
}
```

查询后聚合

```json
# 请求
POST student/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"age": 11} }
      ]
    }
  },
  "aggs":{
    "age_count": {
      "terms": {"field": "age"}
    }
  },
  "size": 0
}
```

各个年龄的人数分布

```json
POST student/_search
{
	"aggs":{
		"age_count":{
			"terms":{"field":"age"}
		}
	},
	"size":0
}
等价于
# 请求
POST student/_search
{
  "size": 0,
  "aggregations": {
    "group_by_age": {
      "aggregations": {
        "count_age": {
          "value_count": {
            "field": "_index"
          }
        }
      },
      "terms": {
        "field": "age"
      }
    }
  }
}

```

> *注意，在 ES 的关键词中 aggs 和 aggregations 等价。*

各个身高的人数分布

```json
POST student/_search
{
	"aggs":{
		"group_by_height":{
			"terms" :{"field":"height"}
		}
	},
	"size":0
}
```

学生的平均年龄、最小年龄、最大年龄、年龄之和

```json
# 请求
POST student/_search
{
  "aggs":{
    "age_stat": {
      "stats": {"field": "age"}
    }
  },
  "size": 0
}
```

`stats` 指令，会计算出指定字段的 count、min、max、avg、sum。

方式2：

```json
# 请求
POST student/_search
{
  "aggs":{
    "age_avg": {
      "avg": {"field": "age"}
    },
    "age_sum": {
      "sum": {"field": "age"}
    },
    "age_min": {
      "min": {"field": "age"}
    },
    "age_max": {
      "max": {"field": "age"}
    },
    "age_count": {
      "value_count": {"field": "age"}
    }
  },
  "size": 0
}
```

每个年龄的平均身高是多少？

```json
# 请求
POST student/_search
{
  "size": 0,
  "aggregations": {
    "group_by_age": {
      "aggregations": {
        "avg_height": {
          "avg": {
            "field": "height"
          }
        }
      },
      "terms": {
        "field": "age"
      }
    }
  }
}
```

获取每个年龄的平均身高，并按照年龄从小到大排序

方式1：

```json
# 请求
POST student/_search
{
  "size": 0,
  "aggregations": { 
    "group_by_age": {
      "aggregations": {
        "avg_height": {
          "avg": {
            "field": "height"
          }
        }
      },
      "terms": {
        "field": "age",
        "order": {
          "_term": "asc"
        }
      }
    }
  }
}
# 响应 （响应中指出 _term 已经废弃，应使用 _key）
#! Deprecation: Deprecated aggregation order key [_term] used, replaced by [_key]
```

方式2：

```json
POST student/_search
{
  "size": 0,
  "aggregations": { 
    "group_by_age": {
      "aggregations": {
        "avg_height": {
          "avg": {
            "field": "height"
          }
        },
        "bucket_sort_by_avg_height": {
          "bucket_sort": {
            "sort": [
              {"_key": {"order": "asc"}}
            ]
          }
        }
      },
      "terms": {
        "field": "age"
      }
    }
  }
}
```

获取每个年龄的平均身高，并按照平均身高从大到小排序

```json
# 请求

POST student/_search
{
  "size": 0,
  "aggregations": { 
    "group_by_age": {
      "aggregations": {
        "avg_height": {
          "avg": {
            "field": "height"
          }
        },
        "bucket_sort_by_avg_height": {
          "bucket_sort": {
            "sort": [
              {"avg_height": {"order": "desc"}}
            ]
          }
        }
      },
      "terms": {
        "field": "age"
      }
    }
  }
}
```