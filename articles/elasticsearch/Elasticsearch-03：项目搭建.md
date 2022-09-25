## Elasticsearch 不完全入门指北系列（三）：项目搭建

今天我们来实际搭建一个简单的 spring-boot web 项目来操作一下 Elasticsearch，通过 ES 的 `elasticsearch-rest-high-level-client` 来调用ES接口。

### 1. 项目初始化

首先我们先创建一个spring-boot web项目，这一步大家可以自行解决。然后引入 `elasticsearch` 相关依赖，引入版本可以根据自己需要调整，一般和安装的 Elasticsearch 一致即可：

maven：

```
<!-- elasticsearch & rest-high-level-client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.12.0</version>
</dependency>
```

gradle：

```
implementation group: 'org.elasticsearch.client', name: 'elasticsearch-rest-high-level-client', version: '7.12.0'
```

增加如下配置：

```yaml
# elasticsearch 相关配置
elasticsearch:
  cluster-name: elasticsearch
  cluster-nodes: 127.0.0.1:9200
  index:
    number-of-replicas: 2
    number-of-shards: 3
  account:
    username:
    password:
```

 此外，还需要创建一个配置类连接 Elasticsearch 客户端：

```java
@Data
@Builder
@Component
@NoArgsConstructor
@AllArgsConstructor
@ConfigurationProperties(prefix = "elasticsearch")
public class ElasticsearchProperties {
    /**
     * 请求协议
     */
    private String schema = "http";
    /**
     * 集群名称
     */
    private String clusterName = "elasticsearch";
    /**
     * 集群节点
     */
    @NotNull(message = "集群节点不允许为空")
    private List<String> clusterNodes = new ArrayList<>();
    /**
     * 连接超时时间(毫秒)
     */
    private Integer connectTimeout = 1000;
    /**
     * socket 超时时间
     */
    private Integer socketTimeout = 30000;
    /**
     * 连接请求超时时间
     */
    private Integer connectionRequestTimeout = 500;

    /**
     * 每个路由的最大连接数量
     */
    private Integer maxConnectPerRoute = 10;
    /**
     * 最大连接总数量
     */
    private Integer maxConnectTotal = 30;
    /**
     * 索引配置信息
     */
    private Index index = new Index();
    /**
     * 认证账户
     */
    private Account account = new Account();

    /**
     * 索引配置信息
     */
    @Data
    public static class Index {
        /**
         * 分片数量
         */
        private Integer numberOfShards = 3;
        /**
         * 副本数量
         */
        private Integer numberOfReplicas = 2;
    }
    /**
     * 认证账户
     */
    @Data
    public static class Account {
        /**
         * 认证用户
         */
        private String username;
        /**
         * 认证密码
         */
        private String password;
    }
}
```

```java
@Configuration
@RequiredArgsConstructor(onConstructor_ = @Autowired)
@EnableConfigurationProperties(ElasticsearchProperties.class)
public class ElasticsearchConfiguration {

    private final ElasticsearchProperties elasticsearchProperties;

    private final List<HttpHost> httpHosts = new ArrayList<>();

    @Bean(name = "restHighLevelClient")
    @ConditionalOnMissingBean
    public RestHighLevelClient restHighLevelClient() {

        List<String> clusterNodes = elasticsearchProperties.getClusterNodes();
        clusterNodes.forEach(node -> {
            try {
                String[] parts = StringUtils.split(node, ":");
                Assert.notNull(parts, "Must defined");
                Assert.state(parts.length == 2, "Must be defined as 'host:port'");
                httpHosts.add(new HttpHost(parts[0], Integer.parseInt(parts[1]), elasticsearchProperties.getSchema()));
            } catch (Exception e) {
                throw new IllegalStateException("Invalid ES nodes " + "property '" + node + "'", e);
            }
        });
        RestClientBuilder builder = RestClient.builder(httpHosts.toArray(new HttpHost[0]));

        return getRestHighLevelClient(builder, elasticsearchProperties);
    }


    /**
     * get restHistLevelClient
     *
     * @param builder                 RestClientBuilder
     * @param elasticsearchProperties elasticsearch default properties
     * @return {@link RestHighLevelClient}
     * @author fxbin
     */
    private static RestHighLevelClient getRestHighLevelClient(RestClientBuilder builder, ElasticsearchProperties elasticsearchProperties) {

        // Callback used the default {@link RequestConfig} being set to the {@link CloseableHttpClient}
        builder.setRequestConfigCallback(requestConfigBuilder -> {
            requestConfigBuilder.setConnectTimeout(elasticsearchProperties.getConnectTimeout());
            requestConfigBuilder.setSocketTimeout(elasticsearchProperties.getSocketTimeout());
            requestConfigBuilder.setConnectionRequestTimeout(elasticsearchProperties.getConnectionRequestTimeout());

            requestConfigBuilder.setAuthenticationEnabled(true);

            return requestConfigBuilder;
        });

        // Callback used the basic credential auth
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        ElasticsearchProperties.Account account = elasticsearchProperties.getAccount();
        if (!StringUtils.isEmpty(account.getUsername()) && !StringUtils.isEmpty(account.getUsername())) {
            credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(account.getUsername(), account.getPassword()));
        }

        // Callback used to customize the {@link CloseableHttpClient} instance used by a {@link RestClient} instance.
        builder.setHttpClientConfigCallback(httpClientBuilder -> {
            httpClientBuilder.setMaxConnTotal(elasticsearchProperties.getMaxConnectTotal());
            httpClientBuilder.setMaxConnPerRoute(elasticsearchProperties.getMaxConnectPerRoute());
            httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            return httpClientBuilder;
        });

        return new RestHighLevelClient(builder);
    }
}
```

然后，先启动 Elasticsearch 客户端，再启动项目即可。访问健康检查接口 `/actuator/health`，可以看到 elasticsearch 的连接情况：

```json
{
    "status":"UP",
    "components":{
        "elasticsearchRest":{
            "status":"UP",
            "details":{
                "cluster_name":"elasticsearch",
                "status":"yellow",
                "timed_out":false,
                "number_of_nodes":1,
                "number_of_data_nodes":1,
                "active_primary_shards":40,
                "active_shards":40,
                "relocating_shards":0,
                "initializing_shards":0,
                "unassigned_shards":79,
                "delayed_unassigned_shards":0,
                "number_of_pending_tasks":0,
                "number_of_in_flight_fetch":0,
                "task_max_waiting_in_queue_millis":0,
                "active_shards_percent_as_number":33.61344537815126
            }
        },
        "ping":{
            "status":"UP"
        }
    }
}
```

### 2. 创建/删除索引

关于 Elasticsearch 的索引，我们常用的几个操作便是检查索引是否存在、创建索引、以及删除索引。对于索引的操作，我们需要使用 `client.indices()` 方法来获取 `IndicesClient` 实例，进而对索引进行相关操作。

**检查索引是否存在。** 首先我们需要构建一个检查索引是否存在的请求 `GetIndexRequest` ，然后使用 `client` 发送请求即可，其响应结果为 `boolean` 类型，即存在与否：

```java
    
    @Autowired
    @Qualifier(value = "restHighLevelClient")
    protected RestHighLevelClient client;

    protected static final RequestOptions COMMON_OPTIONS;

    private void checkIndexExist(String index) {
        try {
            GetIndexRequest indexRequest = new GetIndexRequest(index);
            boolean exists = client.indices().exists(indexRequest, COMMON_OPTIONS);
            if (!exists) {
                createIndexRequest(index);
            }
        } catch (IOException e) {
            log.error("Failed to check index. {}", index, e);
        }
    }
```

**创建索引。** 同理创建索引亦然：

```java
    /**
     * create elasticsearch index (asyc)
     *
     * @param index elasticsearch index
     */
    protected void createIndexRequest(String index) {
        try {
            CreateIndexRequest request = new CreateIndexRequest(index);
            // Settings for this index
            request.settings(Settings.builder()
                    .put("index.number_of_shards", elasticsearchProperties.getIndex().getNumberOfShards())
                    .put("index.number_of_replicas", elasticsearchProperties.getIndex().getNumberOfReplicas()));

            CreateIndexResponse createIndexResponse = client.indices().create(request, COMMON_OPTIONS);

            log.info(" whether all of the nodes have acknowledged the request : {}", createIndexResponse.isAcknowledged());
            log.info(" Indicates whether the requisite number of shard copies were started for each shard in the index before timing out :{}", createIndexResponse.isShardsAcknowledged());
        } catch (IOException e) {
            log.error("failed to create index. ", e);
            throw new ElasticsearchException("创建索引 {" + index + "} 失败");
        }
    }
```

我们可以看到，创建索引时可以进行一些设置，当然复杂一些的还可以定制索引的一些 `mapping` ，可以精确到字段属性维度，后续我们会进行演示。

**删除索引。** 相对来说，删除索引就比较简单了（注：删除索引意味着该索引下的数据也会被删除，再次进行搜索时便会抛出异常）：

```java
    /**
     * delete elasticsearch index
     *
     * @param index elasticsearch index name
     */
    protected void deleteIndexRequest(String index) {
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest(index);
        try {
            client.indices().delete(deleteIndexRequest, COMMON_OPTIONS);
        } catch (IOException e) {
            log.error("failed to delete index. ", e);
            throw new ElasticsearchException("删除索引 {" + index + "} 失败");
        }
    }
```

### 3.  操作 Elasticsearch 文档（`_doc`）

说完索引，就是对文档的操作，同样的，对于 Elasticsearch 的文档（`_doc`），我们也有 CRUD 基础的四种操作。分别对应四个Request 对象：

```java
// 新增/创建一个文档
IndexRequest request = new IndexRequest(index)
  // 文档ID
  .id(id)
  // 文档源数据
  .source(JSONUtil.toJsonStr(object), XContentType.JSON);
// client 创建文档
client.index(request, COMMON_OPTIONS);
```

```java
// 读取某个ID的文档
GetRequest getRequest = new GetRequest(EsConstant.INDEX_NAME) // 索引名称
  // id
  .id(String.valueOf(id));
GetResponse response = client.get(getRequest, COMMON_OPTIONS);
```

```java
// 更新一个文档
UpdateRequest updateRequest = new UpdateRequest(index, id)
  .doc(BeanUtil.beanToMap(object), XContentType.JSON);
// client 更新文档
client.update(updateRequest, COMMON_OPTIONS);
```

```java
// 删除某个ID的文档
// 参数为 索引名称以及ID
DeleteRequest deleteRequest = new DeleteRequest(index, id);
// client 删除文档
client.delete(deleteRequest, COMMON_OPTIONS);
```

还有一种是对文档进行搜索，这一点我会放在后面的系列文章里细讲，对于初始化项目来说，我们先完成这些已经足够了。

### 总结

本篇文章主要给大家简单介绍并演示了一下如何初始化我们的测试项目，大家可以自己试着搭建一个出来。后续我们会详细介绍 Elasticsearch 搜索场景的各种实现。

### 链接

- 项目地址：https://github.com/lq920320/es-rest-client-demo
- 参考项目：https://github.com/xkcoding/spring-boot-demo/tree/master/demo-elasticsearch-rest-high-level-client
