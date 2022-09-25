## Elasticsearch不完全入门指北系列（六）：管道聚合（Pipeline Aggregations）



在本系列的上一篇文章：[Elasticsearch 不完全入门指北系列（五）：聚合搜索](https://github.com/lq920320/blogs/blob/master/articles/elasticsearch/Elasticsearch%E4%B8%8D%E5%AE%8C%E5%85%A8%E5%85%A5%E9%97%A8%E6%8C%87%E5%8C%97%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%94%EF%BC%89%EF%BC%9A%E8%81%9A%E5%90%88%E6%90%9C%E7%B4%A2.md) 中提到的聚合搜索便包含桶聚合（`Bucket Aggregations`）与管道聚合（`Pipeline Aggregations`），本文我们主要先来看一下管道聚合，着重来了解一下什么是管道聚合，以及我们要通过一个例子来详细介绍 `buckets_path` 的语法。

### Pipeline Aggregations（管道聚合）

管道聚合处理从其他聚合而不是文档集产生的输出，将信息添加到输出树。 有许多不同类型的管道聚合，每种聚合计算与其他聚合不同的信息，但这些类型可以分为两个系列：

- ***Parent（父聚合）***
  
  一系列管道聚合，提供其父聚合的输出，并且能够计算新桶（`bucket`）或新聚合以添加到现有桶（`bucket`）。

- ***Sibling（兄弟聚合）***
  
  与兄弟聚合的输出一起提供的管道聚合，并且能够计算与兄弟聚合处于同一级别的新聚合。

管道聚合可以通过使用 `buckets_path` 参数来引用它们执行计算所需的聚合，以指示所需指标的路径。 定义这些路径的语法可以在下面的 `buckets_path` 语法部分中找到。

管道聚合不能有子聚合，但根据类型，它可以引用 `buckets_path` 中的另一个管道，从而允许链接管道聚合。 例如，可以将两个导数链接在一起以计算二阶导数（即导数的导数）。

**注意：** 因为管道聚合只添加到输出中，所以当链接管道聚合时，每个管道聚合的输出将包含在最终输出中。

### `buckets_path` 语法

大多数管道聚合需要另一个聚合作为输入。 输入聚合通过 `buckets_path` 参数定义，该参数遵循特定格式：

```
AGG_SEPARATOR       =  `>` ;
METRIC_SEPARATOR    =  `.` ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
MULTIBUCKET_KEY     =  `[<KEY_NAME>]`
PATH                =  <AGG_NAME><MULTIBUCKET_KEY>? (<AGG_SEPARATOR>, <AGG_NAME> )* ( <METRIC_SEPARATOR>, <METRIC> ) ;
```

比如，路径 `my_bucket>my_stats.avg` 将会指向 `my_stats` 指标中的平均值，该指标包含在 `my_bucket` 桶聚合中。

还有一些更多的例子：

- `multi_bucket["foo"]>single_bucket>multi_metric.avg` 将转到`multi_bucket` 多桶聚合的 `foo` 桶中的单个桶 `single_bucket` 下的`multi_metric` 聚合中的 `avg` 指标。

- `agg1["foo"]._count` 将获取多桶聚合 `multi_bucket` 中 `foo` 桶的 `_count` 指标。

### 一个需求的例子

比如，我们有一个需求，ES 的文档里有一个 `nested` 的 `markets` 列表对象，我们对整个ES 文档分组之后，要根据不同 `maket.marketCode` 条件，来按照对应的 `market.marketSort` 进行排序。

首先，我们先创建一个 `test` 索引：

```
PUT /test

{
    "mappings": {
        "properties": {
            "coId": {
                "type": "keyword"
            },
            "id": {
                "type": "keyword"
            },
            "markets": {
                "type": "nested",
                "properties": {
                    "coId": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "marketCode": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "marketSort": {
                        "type": "long"
                    },
                    "stallCode": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    }
                }
            }
        }
    }
}
```

然后，添加一些数据：

```
POST /test/_bulk?pretty


{"index": { "_index": "test", "_id": "1" }}
{ "coId": "1", "markets": [ { "coId": "1", "marketCode": "A", "marketSort": 66.0, "stallCode": "A-001" } ] }
{"index":{"_index":"test","_id":"2"}}
{"coId":"2","markets":[{"coId":"2","marketCode":"A","marketSort":77,"stallCode":"A-002"},{"coId":"2","marketCode":"B","marketSort":33,"stallCode":"B-001"}]}
{"index":{"_index":"test","_id":"3"}}
{"coId":"3","markets":[{"coId":"3","marketCode":"A","marketSort":99,"stallCode":"A-003"},{"coId":"3","marketCode":"B","marketSort":22,"stallCode":"B-003"}]}
{"index":{"_index":"test","_id":"4"}}
{"coId":"4","markets":[{"coId":"4","marketCode":"B","marketSort":0,"stallCode":"B-004"}]}
```

查询语句如下：

```
POST /test/_search


{
    "from": 0,
    "size": 0,
    "aggregations": {
        "group": {
            "terms": {
                "field": "coId"
            },
            "aggregations": {
                "marketsAgg": {
                    "nested": {
                        "path": "markets"
                    },
                    "aggregations": {
                        "filterMarket": {
                            "filter": {
                                "term": {
                                    "markets.marketCode": "a"
                                }
                            },
                            "aggregations": {
                                "marketSorted": {
                                    "max": {
                                        "field": "markets.marketSort"
                                    }
                                }
                            }
                        }
                    }
                },
                "bucket_sort": {
                    "bucket_sort": {
                        "sort": [
                            {
                                "marketsAgg>filterMarket>marketSorted": {
                                    "order": "desc",
                                    "missing": "_last",
                                    "unmapped_type": "double"
                                }
                            }
                        ],
                        "from": 0,
                        "size": 100,
                        "gap_policy": "SKIP"
                    }
                }
            }
        },
        "total": {
            "cardinality": {
                "field": "coId"
            }
        }
    }
}
```

我们可以看到在排序的聚合 `bucket_sort` 中，我们排序的条件是按照路径 `marketsAgg>filterMarket>marketSorted` 来获取的，同时 `marketsAgg>filterMarket.marketSorted` 也可以实现同样的效果，我们得到了如下的期望结果：

```json
{
    "aggregations": {
        "total": {
            "value": 4
        },
        "group": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "3",
                    "doc_count": 1,
                    "marketsAgg": {
                        "doc_count": 2,
                        "filterMarket": {
                            "doc_count": 1,
                            "marketSorted": {
                                "value": 99.0
                            }
                        }
                    }
                },
                {
                    "key": "2",
                    "doc_count": 1,
                    "marketsAgg": {
                        "doc_count": 2,
                        "filterMarket": {
                            "doc_count": 1,
                            "marketSorted": {
                                "value": 77.0
                            }
                        }
                    }
                },
                {
                    "key": "1",
                    "doc_count": 1,
                    "marketsAgg": {
                        "doc_count": 1,
                        "filterMarket": {
                            "doc_count": 1,
                            "marketSorted": {
                                "value": 66.0
                            }
                        }
                    }
                }
            ]
        }
    }
}
```



### 总结

本篇文章带大家认识一下什么是管道聚合，以及主要是为了让大家知道 `buckets_path` 的语法。我在开始使用的时候以为和对象一样直接用 `.` 操作符去访问路径 `marketsAgg.filterMarket.marketSorted`，结果报错了，找不到路径，ES 聚合的路径不支持这种写法，为此我还提了个 issue，感兴趣的可以去看一下 issue 详情。







### 链接：

- Pipeline aggregations | Elasticsearch Guide： https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline.html#buckets-path-syntax

- Elasticsearh issue#87880： https://github.com/elastic/elasticsearch/issues/87880
