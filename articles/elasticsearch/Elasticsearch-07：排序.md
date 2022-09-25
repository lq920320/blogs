## Elasticsearch 不完全入门指北系列（七）：排序

本篇我们来说一下 Elasticsearch 是如何对搜索结果进行排序对。在搜索功能中，除了基本的搜索，排序是使用最多也是最重要的功能了，我们希望越符合搜索结果的越排在前面，当然根据一些规则，还会给结果设置一些权重，并按照这个权重的高低来排序，像是广告、距离、折扣等等。

Elasticsearch 允许在特定字段上添加一种或多种排序。 每种排序也可以颠倒。 排序是在每个字段级别上定义的，`_score` 是一个特殊字段可以按照搜索结果匹配程度的分数排序，`_doc` 则是按索引顺序排序。

假设我们有一个如下的索引 `mapping` ：

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "post_date": { "type": "date" },
      "user": {
        "type": "keyword"
      },
      "name": {
        "type": "keyword"
      },
      "age": { "type": "integer" }
    }
  }
}
```

```
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

*注意：`_doc` 除了是最有效的排序顺序之外，并没有真正的用例。 因此，如果你不关心文档返回的顺序，那么应该按 `_doc` 排序。 这在滚动（scrolling）查询时特别有用。*

### 排序的值

搜索响应包括每个文档的 `sort` 值。 使用 `format` 参数为 `date` 和 `date_nanos` 字段的 `sort` 值指定日期格式。 以下搜索以 `strict_date_optional_time_nanos` 格式返回 `post_date` 字段的排序值。

```
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"format": "strict_date_optional_time_nanos"}}
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

### 排序顺序

`order` 选项可以有以下值：

- `asc` ：生序排序

- `desc` ：降序排序

按 `_score` 排序时顺序默认为 `desc`，按其他任何内容排序时默认为 `asc`。

### 排序模式选项

Elasticsearch 支持按数组或多值字段排序。 `mode` 选项控制选择哪个数组值来对其所属的文档进行排序。 `mode` 选项可以具有以下值：

- `min` ：选择最小值。

- `max` ：选择最大值。

- `sum` ：使用所有值的总和作为排序值。 仅适用于基于数字的数组字段。

- `avg` ：使用所有值的平均值作为排序值。 仅适用于基于数字的数组字段。

- `median` ：使用所有值的中位数作为排序值。 仅适用于基于数字的数组字段。

升序排序中的默认排序模式是 `min` —  选取最小值。 默认的降序排序模式是 `max` — 取最高值。

#### 排序模式的示例

在下面的示例中，每个文档的价格字段有多个价格。 在这种情况下，结果命中将根据每个文档的平均价格按价格升序排序。

```
PUT /my-index-000001/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

### 对数字字段排序

对于数字字段，也可以使用 `numeric_type` 选项将值从一种类型转换为另一种类型。 此选项接受以下值：["double", "long", "date", "date_nanos"] 并且对于跨多个数据流或排序字段映射不同的索引的搜索很有用。

我们可以看下下面两个索引：

```
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "double" }
    }
  }
}
```

```
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "long" }
    }
  }
}
```

由于 `field` 在第一个索引中映射为 `double` ，而在第二个索引中映射为`long`，因此默认情况下无法使用此字段对查询两个索引的请求进行排序。 但是，你可以使用 `numeric_type` 选项将类型强制为一种或另一种，以便为所有索引强制使用特定类型：

```
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "double"
        }
      }
   ]
}
```

在上面的示例中， `index_long` 索引的值被强制转换为 `double`，以便与 `index_double` 索引生成的值兼容。 也可以将浮点字段转换为 `long` ，但请注意，在这种情况下，浮点被替换为小于或等于参数的最大值（如果值为负，则大于或等于）并且等价于一个数学整数。

此选项还可用于将使用毫秒的日期字段转换为具有纳秒的 `date_nanos` 字段。 例如看一下这两个索引：

```
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "date" }
    }
  }
}
```

```
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "date_nanos" }
    }
  }
}
```

这些索引中的值以不同的格式存储，因此对这些字段进行排序将始终将日期排序在 `date_nanos` 之前（升序）。 使用 `numeric_type` 类型选项，可以为排序设置单个单位，设置为 `date` 会将 `date_nanos` 转换为毫秒，而 `date_nanos` 会将 `date` 字段中的值转换为纳秒：

```
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "date_nanos"
        }
      }
   ]
}
```

*注意：为避免溢出，转换为 `date_nanos` 不能应用于 1970 年之前和 2262 年之后的日期，因为纳秒表示为 `long`。*

### 在嵌套对象里的排序

Elasticsearch 还支持按一个或多个嵌套对象内的字段进行排序。 嵌套字段支持的排序具有嵌套排序选项，具有以下属性：

- `path`：定义要排序的嵌套对象。 实际的排序字段必须是此嵌套对象内的直接字段。 按嵌套字段排序时，此字段为必填项。

- `filter`：嵌套路径内的内部对象应匹配的过滤器，以便通过排序考虑其字段值。 常见的情况是在嵌套过滤器或查询中重复查询/过滤器。 默认情况下，没有启用的 `filter` 。

- `max_children`：选择排序值时要考虑的每个根文档的最大子项数。 默认为无限制。

- `nested`：与顶级 `nested` 相同，但适用于当前嵌套对象中的另一个嵌套路径。

如果在没有 `nested` 上下文的排序中定义嵌套字段，Elasticsearch 将抛出错误。

#### 嵌套排序的例子

在下面的示例中，offer 是一个嵌套类型的字段。 需要指定嵌套路径； 否则，Elasticsearch 不知道需要捕获哪些嵌套级别的排序值。

```
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested": {
                "path": "offer",
                "filter": {
                   "term" : { "offer.color" : "blue" }
                }
             }
          }
       }
    ]
}
```

在下面的示例中，`parent` 和 `child` 字段属于嵌套类型。 每一层都需要指定`nested.path` ； 否则，Elasticsearch 不知道需要捕获哪些嵌套级别的排序值。

```
POST /_search
{
   "query": {
      "nested": {
         "path": "parent",
         "query": {
            "bool": {
                "must": {"range": {"parent.age": {"gte": 21}}},
                "filter": {
                    "nested": {
                        "path": "parent.child",
                        "query": {"match": {"parent.child.name": "matt"}}
                    }
                }
            }
         }
      }
   },
   "sort" : [
      {
         "parent.child.age" : {
            "mode" :  "min",
            "order" : "asc",
            "nested": {
               "path": "parent",
               "filter": {
                  "range": {"parent.age": {"gte": 21}}
               },
               "nested": {
                  "path": "parent.child",
                  "filter": {
                     "match": {"parent.child.name": "matt"}
                  }
               }
            }
         }
      }
   ]
}
```

在按脚本排序和按地理距离排序时也支持嵌套排序。

### 没有命中值

`missing` 参数指定应如何处理缺少排序字段的文档：缺失值可以设置为 `_last`、`_first` 或自定义值（将用于缺少文档作为排序值），即当没有匹配到排序字段或者排序字段到值为 `null` 时，将文档排在末尾或者最前。 默认值为 `_last`。

比如：

```
GET /_search
{
  "sort" : [
    { "price" : {"missing" : "_last"} }
  ],
  "query" : {
    "term" : { "product" : "chocolate" }
  }
}
```

*注意：如果嵌套的内部对象与 `nested.filter` 不匹配，则使用缺失值。*

### 忽略未映射的字段

默认情况下，如果没有与字段关联的映射，则搜索请求将失败。 `unmapped_type` 选项允许忽略没有映射的字段并且不按它们排序。 此参数的值用于确定要使用的排序值。 如下示例：

```
GET /_search
{
  "sort" : [
    { "price" : {"unmapped_type" : "long"} }
  ],
  "query" : {
    "term" : { "product" : "chocolate" }
  }
}
```

如果查询的任何一个索引都没有价格映射，则 Elasticsearch 会按照就好像存在 `long` 类型的映射一样处理它，即使该索引中的所有文档都没有该字段的值。

### 基于脚本的排序

Elasticsearch 允许根据自定义脚本进行排序，这是一个示例：

```
GET /_search
{
  "query": {
    "term": { "user": "kimchy" }
  },
  "sort": {
    "_script": {
      "type": "number",
      "script": {
        "lang": "painless",
        "source": "doc['field_name'].value * params.factor",
        "params": {
          "factor": 1.1
        }
      },
      "order": "asc"
    }
  }
}
```

*注意：脚本和正常的字段排序可以同时使用。*

### Track Scores

对字段进行排序时，是不计算分数的。可以通过将 `track_scores` 设置为 `true`，仍然会计算和跟踪分数。

```
GET /_search
{
  "track_scores": true,
  "sort" : [
    { "post_date" : {"order" : "desc"} },
    { "name" : "desc" },
    { "age" : "desc" }
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

### 内存注意事项

排序时，相关的排序字段值会被加载到内存中。 这意味着每个分片应该有足够的内存来容纳它们。 对于基于字符串的类型，排序的字段不应被解析/分词。 如果可能的话，对于数字类型，建议将类型显式设置为更窄的类型（如 `short`、`integer` 和 `float`）。

### Java代码中如何编写排序

我们需要在构建搜索请求的时候将排序参数设置进去，`SearchSourceBuilder` 的 `sort()` 方法有两种实现，第一种是：

```java
/**
 * name - 要参与排序的字段名
 * order - 排序的顺序，生序或者降序，可选 SortOrder.ASC 或者 SortOrder.DESC
 */
SearchSourceBuilder sort(String name, SortOrder order);
```

另外一种是接收 `SortBuilder` 作为参数，`SortBuilder` 又可以被构造为 `FieldSortBuilder`  或者 `ScriptSortBuilder` ：

```java
/**
 * 添加sort builder
 */
SearchSourceBuilder sort(SortBuilder<?> sort);
```

下面是一个例子：

```java
// 构建一个搜索的参数
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
// 设置查询参数
searchSourceBuilder.query(boolQuery);
// 设置排序参数，直接使用field进行设置
searchSourceBuilder.sort("fieldName", SortOrder.DESC);
// 如果有其他参数的设置，也可以使用 SortBuilders 进行构建
 searchSourceBuilder.sort(SortBuilders.fieldSort("fieldName")
                    .order(sortOrder)
                    .sortMode(SortMode.MAX));
// 使用脚本来构建 SortBuilder，这里的 script 传入的是一个脚本的对象
searchSourceBuilder.sort(SortBuilders
                    .scriptSort(script, ScriptSortBuilder.ScriptSortType.NUMBER)
                    .order(sortOrder).sortMode(SortMode.MAX));
```

构建脚本代码示例：

```java
    /**
     * 构建折扣价格排序脚本
     *
     * @return {@link Script}
     */
    private Script buildSortScript() {
        // 这里我们考虑一种场景，最近一周清仓大甩卖，统统打八折
        // params参数必须是key为String类型，value为Object类型的Map
        Map<String, Object> params = new HashMap<>(2);
        params.put("discountFactor", 0.8);
        String scriptStr = "doc['price'].value * params.discountFactor";
        return new Script(Script.DEFAULT_SCRIPT_TYPE, Script.DEFAULT_SCRIPT_LANG, scriptStr, params);
    }
```

### 总结

本文我们简单看了一下排序在 Elasticseach 搜索中的使用，如遇到不同的排序规则，ES 也是支持多个纬度同时排序的，比如说先按照价格倒序，再按照出版时间倒序等。之前的系列文章里也提到了聚合情况下的排序，以及使用 nested 对象中字段的排序，这里不在乎赘述，大家可以自行回顾一下。



### 链接

- Sort search results： https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html
- 仓库地址： https://github.com/lq920320/es-rest-client-demo
