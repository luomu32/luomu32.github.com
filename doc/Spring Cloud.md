# Spring Cloud

Spring Cloud为开发人员快速构建分布式系统提供一些常见的模式工具（例如配置管理，服务发现，熔断器，智能路由，微代理，控制总线，一次性令牌，全局锁，领导选举，分布式会话，集群状态）。

基于Spring Cloud的分布式系统架构：

![](https://spring.io/img/homepage/diagram-distributed-systems.svg)

- 服务发现
- 熔断器
- 配置中心
- API网关
- 分布式跟踪
- OAuth2
- 消费驱动合约

## 术语

治理：简单讲就是服务管理。包括服务的发现，变更，监控，调用等等。

容错：当系统在运行时有错误被激活的情况下仍能保证不间断提供服务的方法和技术

冗余：提供系统的可用性

## 版本

由于是一个综合项目，包含了很多子项目，所以版本命名以“英文单词+SR+数字”形式。英文单词一般采用伦敦地铁名。SR表示Service Release用于bug修复。

## 子项目

- Spring Cloud Config：配置管理。配置刷新，加解密配置
- Spring Cloud Netfix
  - Eureka：服务治理
  - Hystrix：容错管理
  - Ribbon：负载均衡
  - Feign：基于Ribbon和Hystrix的声明式服务调用组件
  - Zuul：网关。提供智能路由、访问过滤
  - Archaius：外部化配置
- Spring Cloud Bus：事件总线。用于传播集群中的状态变化或事件。
- Spring Cloud Cluster：针对ZooKeeper、Redis、Hazelcast、Consul的选举算法和通用状态模式的实现
- Spring Cloud Cloudfoundry
- Spring Cloud Consul：服务发现和配置管理
- Spring Cloud Stream：通过简单的声明式来收发消息
- Spring Cloud AWS
- Spring Cloud Security：安全。提供Zuul代理对OAuth2客户端请求的中继器
- Spring Cloud Sleuth：分布式链路跟踪
- Spring Cloud Zookeeper：基于ZooKeeper的服务发现和配置管理
- Spring Cloud Starters
- Spring Cloud CLI

## 配置中心

Spring Cloud提供四种配置中心服务端。

- 基于Git或Svn
- 基于Zookeeper
- 基于Consul
- 基于Vault

### spring-cloud-config

分为server和client两个部分。

作为server

1. 添加`spring-cloud-config-server`依赖。
2. 在application.properties等配置文件加上`spring.cloud.config.server.git.uri`配置指向git目录
3. 在启动类上添加注解：`EnableConfigServer`

作为client

1. 添加`spring-cloud-config-client`依赖
2. 在bootstrap.properties/bootstrap.yml配置文件上添加`spring.cloud.config.uri`指向server

当在服务端更新配置后，客户端读取的还是原来的值，需要手动刷新一下。但是在生产环境中，应用不止一个实例，这时候需要搭配Spring Cloud Bus来统一刷新。

### spring-cloud-config-zookeeper

基于zookeeper的配置中心就没有server端，数据库落在zookeeper上，client直接与zookeeper交互。

1. 添加`spring-cloud-starter-zookeeper-config`依赖
2. 在bootstrap.properties/bootstrap.yml添加`spring.cloud.zookeeper.connect-string`

配置示例：

```yaml
spring:
  application:
    name: zc
  cloud:
    zookeeper:
      enabled: true  # true:开启zookeeper外部化配置, false:读取本地配置;
      connect-string: 127.0.0.1:2181
      config:
        root: /config
        enabled: true
        watcher:
          enabled: false
```

这种方案的配置中心，一个缺点是配置不好维护。没有一个可用的图形化界面方便的维护配置，需要借助第三方工具。

配置刷新

ConfigWatcher类继承自TreeCacheListener，依靠CuratorFramework的变更通知，当Zookeeper中有节点发生新增、变更、删除时ConfigWatcher发布一个RefreshEvent事件。

### 高可用

需要按照不同的实现来讨论。如果采用Eureka，首先是Git仓库的高可用，然后是配置中心的高可用。

如果是采用ZooKeeper，只需要做到ZooKeeper的高可用。

## 服务中心

Spring Cloud支持多种框架实现服务中心。某一个版本之后，Eureka闭源，Spring不再推荐。

- Eureka
- Consul
- Zookeeper
- Cloud Foundry

其中如果Eureka作为服务中心的实现，还需要一个Server端。server端spring cloud也有提供，具有图形化界面，方便展示和维护。



## API网关

Spring Boot 1.x使用的Zuul作为API网关，而到了2.x之后，Spring自己写了一个网关的实现。

Spring Cloud Gateway旨在提供一种简单而有效的方式来路由到API，并为他们提供横切关注点，例如：安全性，监控/指标和弹性。

API网关可以说是外观模式的一种实现，内部服务的统一出口。方便外部调用，所以肯定具有服务编排的功能，还会有协议转换。API网关提供的API，对外部用户更友好。

Spring Cloud Gateway提供了以下特性：

- 构建在Spring Framework 5，Reactor和Spring Boot 2.0之上
- 路由可以指定谓词和过滤器
- 集成Hystrix熔断器
- 集成Spring Cloud服务发现
- 容易编写的谓词和过滤器
- 请求限速
- 路径重写

### 核心概念

- Route路由
- Filter过滤器
- Predicate断言

##全链路跟踪

Trace表示一次完整请求，Span位于Trace之下，表示一次微服务请求。一个请求会调用到多个微服务接口，每个微服务用一个Span表示。Span可以手动创建，也可以自定义开启，但是不要忘记关闭。

ELK、ZipKin

抽样率。

## Stream消息流

## 与gRPC集成

## 版本控制与灰度发布

系统发布的几种方式：

- 蓝绿发布。绿表示旧版本，蓝表示新版本。测试通过后，将流量全部从绿切到蓝。
- 滚动发布。集群少量节点更新为新版本，逐步更新，直至集群全部更新完成。当一个节点要更新时，需要切断流量，当更新完成之后，这个节点重新被分配到流量。
- 灰度发布。在线上部署，切入小部分流量，然后线上测试，没有问题之后放大流量。
  - A/B测试。
  - 金丝雀部署

## 分布式事务

## 领域驱动

领域模型通过聚合（Aggregate）组织在一起。聚合间有边界，边界将领域划分成一个个界限上下文（Bounded Context）。一般一个微服务代表一个界限上下文。微服务设计首先识别出聚合根（Aggregate Root）。微服务之间集成应采用DDD的防腐层（Anti-Corruption Layer，ACL）

- 实体。带有唯一标识。具有属性和行为。具有状态。
- 值对象。没有唯一标识。不可变。易于创建，随手丢弃。
- 领域服务。有些操作或动作，不适合归类到实体或值对象，就放到服务中。服务会涉及到多个实体。
- 聚合和聚合根。聚合定义一组有内聚关系的对象集合。对象之间密不可分，缺一不可。一个聚合包含多个实体和值对象，聚合也称为根实体。一个实体也可以作为聚合根。只有根实体对外暴露引用。
  - 根实体有全局唯一标识，负责检查规定规则
  - 聚合内的实体有本地标识，在聚合范围才是唯一的
  - 外部对象只能引用根实体
  - 只有根实体才能直接通过数据库查询获得，其他对象必须通过遍历关联来发现
  - 聚合内部对象可以保持其他聚合根的引用
  - 对聚合内任何对象进行修改，整个聚合所有固定规则（invariant）都必须满足
- 界限上下文。对于一个模型，在不同界限上下文中可能会有不同的含义，概念，规则等等。每个界限上下文应该有自己的模型。跨界限上下文的通信，不要简单的使用RPC服务调用，而是加入一个专门的防腐层做转化。防腐层对外部依赖解耦，避免外部领域概念污染内部实体语义。
- 工厂。对于一些创建起来比较复杂的对象，需要由一个工厂来负责创建。工厂需要知道满足业务规则，需要知道如何实例化对象，如何初始化等。而被创建的对象，如果参数无效应当抛出异常，而不是创建出一个错误的对象
- 仓储/资源库。负责将实体持久化。资源库里的对象一定是聚合的。

