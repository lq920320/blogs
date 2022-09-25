## Elasticsearch不完全入门指北系列（四）：组合搜索



本篇内容我们来演示一下 Elasticsearch 复杂搜索语句写法。拿 sql 语句来说，当我们的查询需求需要很多条件进行查询时，就得用 `and` 或者 `or` 拼接多个条件进行查询，有时候还需要带上排序条件以及分页参数。Elasticsearch 的搜索也是很类似的，当要使用多个条件进行搜索时，就也得拼接多个条件组成一个搜索语句（请求），我们可以称之为组合搜索。其中用来连接多个查询条件的便是 `BoolQueryBuilder` 类。

### 1 来认识一下各种查询（Query）

#### 全文搜索

| DSL Search Query                                             | Java 类名                                                    | 使用方法                                                     | 解释说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Match](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query.html) | [ MatchQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/MatchQueryBuilder.html) | [ QueryBuilders.matchQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#matchQuery-java.lang.String-java.lang.Object-) | **匹配查询** 是执行全文搜索的标准查询，包括用于模糊匹配的选项。返回与提供的文本、数字、日期或布尔值匹配的文档。 在匹配之前会分析提供的查询条件。 |
| [Match Phrase](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query-phrase.html) | [MatchPhraseQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/MatchPhraseQueryBuilder.html) | [QueryBuilders.matchPhraseQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#matchPhraseQuery-java.lang.String-java.lang.Object-) | match_phrase 查询分析文本并根据分析的文本创建**短语**查询。  |
| [Match Phrase Prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query-phrase-prefix.html) | [MatchPhrasePrefixQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/MatchPhrasePrefixQueryBuilder.html) | [QueryBuilders.matchPhrasePrefixQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#matchPhrasePrefixQuery-java.lang.String-java.lang.Object-) | 以与提供的相同的顺序返回包含提供的文本的单词的文档。 提供的文本的最后一个 term 被视为前缀，匹配以该术语开头的任何单词。 |
| [Query String](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-query-string-query.html) | [ QueryStringQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryStringQueryBuilder.html) | [ QueryBuilders.queryStringQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#queryStringQuery-java.lang.String-) | 可以使用 query_string 查询来创建包含通配符、跨多个字段的搜索等的复杂搜索。 虽然用途广泛，但查询是严格的，如果查询字符串包含任何无效语法，则返回错误。 |

#### 词语级别的搜索

| DSL Search Query                                             | Java 类名                                                    | 使用方法                                                     | 解释说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Term](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-term-query.html) | [TermQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/TermQueryBuilder.html) | [QueryBuilders.termQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#termQuery-java.lang.String-java.lang.String-) | 返回在提供的字段中包含确切词语的文档。可以使用词语查询根据价格、产品 ID 或用户名等精确值查找文档。 |
| [Terms](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-terms-query.html) | [TermsQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/TermsQueryBuilder.html) | [QueryBuilders.termsQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#termsQuery-java.lang.String-java.util.Collection-) | 多个词语查询。                                               |
| [Range](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-range-query.html) | [RangeQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/RangeQueryBuilder.html) | [QueryBuilders.rangeQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#rangeQuery-java.lang.String-) | 返回包含提供范围内的词语的文档。`gte`、`lte`。               |
| [Exists](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-exists-query.html) | [ExistsQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/ExistsQueryBuilder.html) | [QueryBuilders.existsQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#existsQuery-java.lang.String-) | 返回包含字段索引值的文档。即文档中是否存在该 field。         |
| [Wildcard](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-wildcard-query.html) | [WildcardQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/WildcardQueryBuilder.html) | [QueryBuilders.wildcardQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#wildcardQuery-java.lang.String-java.lang.String-) | 返回包含匹配通配符模式词语的文档。通配符运算符是匹配一个或多个字符的占位符。 例如，* 通配符运算符匹配零个或多个字符。 您可以将通配符运算符与其他字符组合以创建通配符模式。 |
| [Ids](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-ids-query.html) | [IdsQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/IdsQueryBuilder.html) | [QueryBuilders.idsQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#idsQuery--) | 根据文档的 ID 返回文档。 此查询使用存储在 _id 字段中的文档 ID。 |

#### 组合查询

| DSL Search Query                                             | Java 类名                                                    | 使用方法                                                     | 解释说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Bool](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-bool-query.html) | [BoolQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/BoolQueryBuilder.html) | [QueryBuilders.boolQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#boolQuery--) | 匹配与其他查询的布尔组合匹配的文档的查询。 bool 查询映射到 Lucene BooleanQuery。 它是使用一个或多个布尔子句构建的，每个子句都有一个查询类型的出现 |

| 关键字     | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `must`     | 子句（查询）必须出现在匹配的文档中，并将有助于得分。         |
| `filter`   | 子句（查询）必须出现在匹配的文档中。 然而，与 must 不同的是，查询的分数将被忽略。 过滤器子句在过滤器上下文中执行，这意味着忽略评分并考虑缓存子句。 |
| `should`   | 子句（查询）应该出现在匹配的文档中。                         |
| `must_not` | 子句（查询）不得出现在匹配的文档中。 子句在过滤器上下文中执行，这意味着忽略评分并考虑将子句用于缓存。 由于评分被忽略，所有文档的评分为 0。 |

#### 连接查询

| DSL Search Query                                             | Java 类名                                                    | 使用方法                                                     | 解释说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Nested](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-nested-query.html) | [NestedQueryBuilder](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/NestedQueryBuilder.html) | [QueryBuilders.nestedQuery()](https://artifacts.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.15.1/org/elasticsearch/index/query/QueryBuilders.html#nestedQuery-java.lang.String-org.elasticsearch.index.query.QueryBuilder-org.apache.lucene.search.join.ScoreMode-) | 包装另一个查询以搜索嵌套字段。嵌套查询搜索嵌套字段对象，就好像它们被索引为单独的文档一样。 如果对象与搜索匹配，则嵌套查询将返回根父文档。要使用嵌套查询，索引必须包含嵌套字段映射（mapping）。 |

常规的多条件搜索应该都会包含在上面表格中所列取的 query 中，具体的使用方法大家可以自行查阅[文档]([Building Queries | Java REST Client [7.15\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-query-builders.html))。

### 2. 组合查询

当我们要搜索数据时，一般我们需要构建搜索请求，然后通过 `ES` 的 Java 客户端调用搜索 API 进行搜索数据。再对搜索得到的结果进行解析和处理。

官方文档方法：

```java
 // 创建一个搜索请求
SearchRequest searchRequest = new SearchRequest();
// 大多数搜索参数都添加到 SearchSourceBuilder。 它为进入搜索请求正文的所有内容提供设置。  
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder(); 
// 把匹配所有文档查询到 SearchSourceBuilder
searchSourceBuilder.query(QueryBuilders.matchAllQuery()); 
// 把 SearchSourceBuilder 添加到 SearchRequest.
searchRequest.source(searchSourceBuilder); 

// 同步返回查询结果
SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

// 异步返回查询结果
ActionListener<SearchResponse> listener = new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        // 请求成功返回的结果
    }

    @Override
    public void onFailure(Exception e) {
        // 请求失败
    }
};
client.searchAsync(searchRequest, RequestOptions.DEFAULT, listener);

```

当我们使用多个条件进行搜索时，就需要使用 `BoolQueryBuilder` 将各个条件组合起来，其中有 `must`、`mustNot`、`should` 几个关键的链接词，分别表示，文档中必须出现匹配的内容，必须不出现匹配的内容，应该出现匹配的内容，对比我们常用的 sql 语法，可以简单类比为 `and`、`and !=` 、`or` 这几个关键字。

比如，我们创建了一个 `/book` 索引，然后往索引里添加几个文档，接着，我们要开始搜索一部分图书，比如，名称中包含“编程”的，我们可以通过下面的方式构建一个DSL查询的方式：

```
POST /book/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "bool": {
                        "should": [
                            {
                                "match": {
                                    "bookName": {
                                        "query": "编程"
                                    }
                                }
                            },
                            {
                                "match_phrase": {
                                    "bookName": {
                                        "query": "编程"
                                    }
                                }
                            }
                        ]
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "id": {
                "order": "asc"
            }
        }
    ]
}
```

意思就是说，我们查询的时候可以匹配出含有“编程”、“编”、“程”或者含有“编程”短语的文档，最后的搜索结果按照 id 生序排列。那么我们就可以按照这个逻辑构建一个Java的“DSL”语句：

```java
// 创建一个 BoolQueryBuilder
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
// 模糊搜索，使用should连接两个匹配
BoolQueryBuilder bookNameQuery = QueryBuilders.boolQuery();
bookNameQuery.should(QueryBuilders.matchQuery("bookName", searchReq.getBookName()));
bookNameQuery.should(QueryBuilders.matchPhraseQuery("bookName", searchReq.getBookName()));
// 使用 must 组合这两个
boolQuery.must(bookNameQuery);

// 创建一个搜索请求，设置索引名称
SearchRequest searchRequest = new SearchRequest("book");
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
// 设置组合索引到 SearchSourceBuilder
searchSourceBuilder.query(boolQuery);
// 默认按 id 正序排序
searchSourceBuilder.sort("id", SortOrder.ASC);
// 将 SearchSourceBuilder 添加到 SearchRquest
searchRequest.source(searchSourceBuilder);
// 执行搜索
SearchResponse searchResponse = client.search(searchRequest, COMMON_OPTIONS);

```

其他查询条件大家也可以自行进行设置和试验，像是 `TermQueryBuilder` 、`WildcardQueryBuilder`、`NestedQueryBuilder`等等。除此之外，Search API还有一些其他的设置，比如高亮，搜索等等，后面的文章我们会逐一介绍到。

### 总结

本文我们先简单介绍了一下 Java Rest Client 的 Search API，当我们需要根据多个条件来搜索的时候，应该怎么把各个条件组合起来，以及哪些文档字段是需要使用精确匹配还是模糊匹配，还是相关度匹配。对于排序，我们是可以设置多个排序条件的，`SearchSourceBuilder` 的 `sort` 方法是可以设置多次的。后面的文章会继续对于各种搜索做介绍和实例演示，大家可以先参考一下链接中的项目，如有问题，欢迎叨扰。



### 链接

- [Building Queries | Java REST Client [7.15\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-query-builders.html)
- [Search API | Java REST Client [7.15\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html)
- [项目地址](https://github.com/lq920320/es-rest-client-demo)