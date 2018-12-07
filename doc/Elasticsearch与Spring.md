# Elasticsearch与Spring

ElasticSearch是一款非常流行的搜索框架，基于Lucene构建，屏蔽了Lucene的复杂性。而Spring Data Elasticsearch方便应用以Sprinig提供的一致的模型和方式来访问控制Elasticsearch。

## 配置

由于Elasticsearch暴露HTTP接口。所以可以使用Spring Boot支持的HTTP Client来与之交互：

- Java官方提供的底层或高层的REST Client
- Jest

还可以使用由Spring Data Elasticsearch项目提供的Transport Client来与Elasticsearch交互。

### REST Client连接Elasticsearch

Elasticsearch提供两种不同REST Client。

如果你将` org.elasticsearch.client:elasticsearch-rest-client`依赖加入到你的classpath当中，Spring Boot会自动配置和注册`RestClient`，默认采用localhost:9200。你可以进一步调整`RestClient`的配置，如下：

```properties
spring.elasticsearch.rest.uris=http://search.example.com:9200
spring.elasticsearch.rest.username=user
spring.elasticsearch.rest.password=secret
```

如果你将` org.elasticsearch.client:elasticsearch-rest-high-level-client`依赖加入当你的classpath当中，Spring Boot会自动配置一个`RestHighLevelClient`，该Bean会聚合一个任意的`RestClient`，并重用该HTTP配置。

### Jest连接Elasticsearch

如果你将Jest的依赖加入到classpath中，Spring Boot会自动配置`JestClient`，默认采用localhost:9200。你可以进一步调整配置，如下：

```properties
spring.elasticsearch.jest.uris=http://search.example.com:9200
spring.elasticsearch.jest.read-timeout=10000
spring.elasticsearch.jest.username=user
spring.elasticsearch.jest.password=secret
```

### Spring Data连接Elasticsearch

通过`spring.data.elasticsearch.cluster-nodes`配置来设置一个或多个需要连接的Elasticsearch的节点地址。比如：`spring.data.elasticsearch.cluster-nodes=localhost:9300`。`TransportClient`和`ElasticsearchTemplate`就会被自动配置，然后你可以在你的Bean中注入这两个Bean开始使用。

通过`spring.data.elasticsearch.properties.*`，可以自定义配置低层的TransportClient的一些配置。比如：

```properties
spring.data.elasticsearch.properties.client.transport.ignore_cluster_name=true
```

也可以自定义配置`TrasnsportClientFactoryBean`来覆盖starter中的默认注册的Bean，来配置一些常用的基础配置。同样可以直接自定义配置`TransportClient`来实现更多的自定义配置。

`TransportClient`是连接Elasticsearch服务的关键类。Spring提供`TransportClientFactoryBean`类来构建`TransportClient`。Spring Boot提供的`ElasticsearchAutoConfiguration`可以自动创建`TransportClientFactoryBean`。`ElasticsearchDataAutoConfiguration`类可以自动创建`ElasticsearchTemplate`。

## Repository

Repository是Spring Data框架提供的统一访问各种数据源的接口。在Spring Data Elasticsearch中，主要是ElasticsearchRepository类。ElasticsearchRepository继承自PagingAndSortingRepository，除了提供常见的增删改查功能外，还提供了额外的搜索功能。

Spring Data Elasticsearch还提供了ElasticsearchTemplate类。ElasticsearchTemplate负责所有于Elasticsearch交互的功能。



## Entity

由@Document标注。indexName属性必填，对应Elasticsearch中的Index，Type对应Elasticsearch中的Type，可为空。

