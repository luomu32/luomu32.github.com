# Spring Cloud之服务中心

spring-cloud-common中封装了一些服务注册的抽象，不限于技术，可以选择Eureka、Consul、ZooKeeper作为我们的服务注册与发现中心。

通过服务注册中心访问服务有三种方式：

- RestTemplate
- OpenFeign
- DiscoveryClient

RestTemplate需要添加@LoadBalanced。一方面开启了负载均和，另一方面也会从注册中心寻找服务。不然服务会找不到。

## Eureka

一些常用的Eureka REST API

| 操作                      | 地址                                                         | HTTP方法 | 返回                               |
| ------------------------- | ------------------------------------------------------------ | -------- | ---------------------------------- |
| 注册新的实例              | /eureka/apps/{appId}                                         | POST     | 成功返回204                        |
| 注销                      | /eureka/apps/{appId}/{instanceId}                            | DELETE   | 成功返回200                        |
| 发送心跳                  | /eureka/apps/{appId}/{instanceId}                            | PUT      | 成功返回200，instance不存在返回404 |
| 查询所有实例              | /eureka/apps                                                 | GET      |                                    |
| 查询指定appId的实例       | /eureka/apps/{appId}                                         | GET      |                                    |
| 根据appId和instanceId查询 | /eureka/apps/{appId}/{instanceId}                            | GET      |                                    |
| 根据指定instanceId查询    | /eureka/instances/{instanceId}                               | GET      |                                    |
| 暂停应用实例              | /eureka/apps/{appId}/{instanceId}/status?value=OUT_OF_SERVICE | PUT      |                                    |
| 恢复应用实例              | /eureak/apps/{appId}/{instanceId}/status?value=UP            | DELETE   |                                    |
| 更新元数据                | /eureka/apps/{appId}/{instanceId}/metadata?key=value         | PUT      |                                    |
| 根据vip地址查询           | /eureka/vips/{vipAddress}                                    | GET      |                                    |
| 根据svip查询              | /eureka/svips/{svipAddress}                                  | GET      |                                    |

- InstanceInfo表示一个实例
- LeaseInfo表示实例的租约信息
- InstanceStatus表示服务的状态，其中OUT_OF_SERVICE表示不会被路由到，常用于升级部署
- LeaseManager接口，定义了几个方法用于服务注册、下线、心跳等
- LoopupService接口，定义了几个方法用于从服务注册中心查询服务

Eureka是一个AP类型的，允许集群各节点的注册信息不是一致的，需要客户端支持负载均衡及失败重试。

一般分布式系统，数据在节点之间复制或同步，有两种方式：

- 主从复制。所有对数据的写操作都提交到主节点，再由主节点更新其他从节点。写操作压力都在主节点上，容易成为系统的瓶颈。从节点负担读操作
- 点对点复制（Peer to Peer）。每个节点都可以写，各节点之间相互更新数据。数据冲突是一个问题。

Eureka的各节点的复制采用点对点。为了防止每个Client按照配置文件顺序请求Eureka Server造成请求分布不均衡的情况，Client端会有定时任务每隔五分钟刷新并随机化服务列表。

Eureka服务端也同样是其他Eureka服务端的客户端。

数据复制冲突的问题：

- lastDirtyTimestamp标识。相当于版本号
- heartbeat

Eureka服务端与客户端通过心跳，并通过访问客户端暴露的自定义健康检查API来进行存活确认。因网络抖动等原因会出现一定的误判。另外如果出现网络分区问题，将出现该分区内的服务大规模下线，这时Eureka会进入自我保护模式，将不对服务下线。

### 常见问题

- 为什么服务下线了，服务端还有服务
- 为什么服务下线了，其他客户端不能及时获取到
- 为什么有时候会出现：

EMERCENCY！EUREKA MAY BE INCORRECTLY.....

第一个问题，Eureka并不是强一致，所以会有过期数据。一个原因是应用当掉之后没有通知Eureka服务端，需要服务端的任务去删除，通过设置eureka.server.eviction-interval-timer-in-ms来调快频率。另一个原因是Eureka的REST API接口有缓存，可以通过eureka.server.use-read-only-response-cahce=false来关闭缓存。在测试环境中可以关闭自我保护模式，eureka.server.enable-self-preservation=false。对于Client获取其他新服务上线获取不及时的问题，可以通过eureka.client.registry-fetch-interval-seconds=5，来提高Client拉取服务端注册信息的频率。对于生产环境，可以调小renewalPercentThreshold和leaseRenewalIntervalInSeconds来提高进入自我保护模式的门槛。

Eureke的监控，Spring Boot提供Micrometer，支持Netflix的Spectator。

Eureka的在线扩容，由于需要修改配置，所以最好将Eureka服务端接入配置中心。

###Eureka不可用的场景

可以为应用配置eureka.client.backup-registry-impl属性，当出现应用在启动的时候，Eureka服务端不可用，应用可以从这个back registry获取服务注册信息，作为fallback。比较适合服务端有负载均衡或服务IP地址固定的场景。

如果应用启动之后，Eureka服务端不可用。那么Client端的定时同步服务端注册信息的任务会抛异常，无法更新Client本地的服务列表。

如果部分Eureka服务端不可用，

```yaml
eureka: 
  client:
    serviceUrl:
      defaultZone: http://locahost:8091/eureka,http://localhost:8092/eureka,http://localhost:8093/eureka
```

第一次8091，8092，8093这个顺序访问Eureka服务端，定时任务会随机化一次，然后可能会是8092，8093，8091这个顺序。Client会有重试，默认3次，如果不可用，会被加入到Client维护的不可用列表中。

由于服务端之间会同步数据，如果有服务端不可用，那么会影响数据同步，会抛出异常，需要人工修改配置剔除可不用的服务端。这个依赖具体实现，后续版本的Eureka可能不需要人工操作，服务端之间也有健康检查，可以自动去除。

###使用

需要一个server端。

1. 添加依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

2. application.properties或application.yml添加配置

```yaml
eureka:
	instance:
		hostname: localhost
		
```

然后是客户端

### 流程

## Consul

CP

Consul与ZooKeeper有点类似。是Go语言编写的一个独立的中间件。即可作为服务注册使用，也可以作为配置中心使用。

ConsulServiceRegistry负责向注册中心注册、下线服务。

如果使用Ribbon（Feign或@LoadBalance），背后采用ConsulServerList。也可以直接使用DiscoveryClient。

## 服务调用之Feign

Feign是一个声明式的HTTP Client。Spring在启动时，会扫描容器内被@FeignClient的接口，然后对其生成代理。当接口的方法被调用时，代理会生成一个RequestTemplate对象。然后由RequestTemplate生成Request对象交由Client处理。Client可以是JDK的URLConnection、Apache的Http Client也可以是Okhttp。最后Client会被封装到LoadBalanceClient，支持负载均衡

FeignClient：

- name，指定名称。如果使用Ribbon，通过name作为服务发现来调用
- url。一般用于调试，手动指定调用地址
- decode404，true时并出现404时会解码，否则抛出异常
- configuration：Feign配置类，可以配置Encoder、Decoder、LogLevel、Contract
- fallback：定义容错处理类。fallback指定类必须被@FeignClient注解标记
- fallbackFactory：用于生成容错处理类的工厂。
- path：统一前缀

Feign支持对请求和响应进行GZIP压缩。开启压缩时候，传输会采用二进制，所以返回值需要修改为ResponseEntity<byte[]>。

在application.yml或application.properties中，既可以对所有FeignClient进行配置，也可以对单个指定name的FeignClient进行配置。在@EnableFeignClient中也可以对所有Client进行配置。

Feign默认采用JDK自带的URLConnection发送HTTP请求。没有连接池，可以替换成Apache Http Client或其他Client。

多参数传递，Spring MVC支持GET方法将参数绑定到一个POJO中，但是Feign不支持。所以有一下几种方式：

- POJO拆分成一个个参数
- Map作为参数
- @RequestBody
- 实现Feign的RequestInterceptor来拦截请求，自定义实现

Feign有个子项目，feign-form来支持文件上传。

Feign首次失败问题，首次请求会比较慢。Hystrix默认1秒超时进入fallback。可以将Hystrix超时时间调大或禁用超时时间。

##Ribbon负载均衡

### 超时与重试



### 饥饿加载

## Hystrix

一般情况下，Ribbon的超时时间应短于Hystrix超时时间。

以下情况会被Hystrix的fallback截获

- 方法失败抛出异常
- 执行超时
- 断路器打开
- 线程池拒绝
- 信号量拒绝

线程池的大小配置。有一个公式：每秒请求峰值*99%的延迟百分比（请求的响应时间）+预留缓冲的线程数。比如，峰值30个请求，0.2s响应时间，预留4个线程：30\*0.2+4=6+4=10。

### Dashboard

当加入Hsytrix，Actuator会暴露一个端点，展示Hystrix的统计信息。集群下Turbine。

