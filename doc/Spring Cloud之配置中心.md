# Spring Cloud之配置中心

spring-cloud-config是配置中心的项目，支持Git、SVN、文件、[Vault](https://www.vaultproject.io/)、JDBC作为后端存储。

如果使用ZooKeeper作为配置中心，采用spring-cloud-starter-zookeeper-config项目。

如果使用Consul作为配置中心，采用spring-cloud-starter-consul-config项目。

## 配置在配置中心的结构

### Git

全局配置在application.properties或application.yml中，应用的配置在{application-name}-{profile}.properties或{application-name}-{profile}.yml中。

### ZooKeeper

可以为配置设置一个root，不设置为/根目录。假如root设置为/config，那么全局配置在/config/application目录下，应用的配置在/config/application/{application-name}目录下。

如果指定了profile，那么先是/config/{application-name},{profile}，再是/config/{application-name}，再是/config/application,dev，再是/config/application

### Consul

默认的路径是/config/{application-name}。如果使用了profile，/config/{application-name},{profile}/。除了可以配置k-v之外，还可以配置k-yml|properties。

## 配置的使用

- @Value
- Environment
- @ConfigurationProperties

@Value使用方式历史最为悠久，可以注解在字段上。Environment除了提供简单的获取配置外，还支持默认值，类型转换等高级功能。@ConfigurationProperties是Spring Boot中提供的功能，可以将同业务的配置放在一个类中，方便维护管理。

另外一点不同的是，如果在配置中心中没有对应的配置，使用@Value会报错，容器启动失败，Environment和@ConfigurationProperties不会报错，容器正常启动。获取时，Environment返回null（Environment还支持默认值等高级用法），而@ConfigurationProperties，会返回类型的默认值，比如是int，返回int的默认值0。

客户端还可以配置，覆盖优先级。比如使用客户端所在系统环境变量覆盖配置中心的配置。

###配置的刷新

当配置中心的配置发生变更时，我们会希望自动的去刷新各个应用中的对应配置。使用不同的方式作为配置中心，刷新的策略有所不同。

####ZooKeeper作为配置中心的刷新

如果使用@Value，需要在类上标注@RefreshScope注解。而Environment和@ConfigurationProperties无需额外的配置，默认就会被自动刷新。

默认情况下，`ZookeeperConfigAutoConfiguration`会将`ConfigWatcher`注册到容器中。`ConfigWatcher`根据CuratorFramework提供了监听能力，当ZooKeeper中的节点发生变动时，`ConfigWatcher`发布RefreshEvent事件。

#### Consul作为配置中心的刷新

`ConfigWatcher`默认每秒执行一次，去Consul获取配置。Consul返回时会带上consulIndex，类似版本号。

####使用Git、SVN或文件作为配置中心的刷新

如果加入了spring-boot-starter-actuator依赖，会暴露一个/refresh接口，用于手动刷新。对于有很多应用实例的情况下，这种手动刷新的方式显然不现实。那么需要使用spring-cloud-bus来自动刷新。

应用加入bus依赖之后，比如使用RabbitMQ：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

并开启端点：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

此时，会暴露一个/actuator/bus-refresh的接口。然后在Git配置Hook，当提交修改到Git后触发Hook去调用/actuator/bus-refresh接口。

## 配置的加密

spring-cloud-config作为配置中心，server端加入Actuator后提供几个端点用于加密解密等操作。要使用先需先配置密钥。

1. 在bootstrap.yml或bootstrap.properties中配置`encrypt`节点，设置加密密码或对称加密的密钥，或非对称加密
2. 在配置中心的配置项使用`{cipher}`作为前缀并以单引号包裹，表明这个配置是加密的

key的管理。对称加密可以通过`encrypt.key`配置文本格式的密钥，非对称加密也可以使用PEM格式的文本内容在`encrypt.key`中配置，也可以使用`encrypt.keystore`指定keystore。

还支持多密钥。

解密这个动作既可以放在配置中心做，也可以在客户端-应用中做。

对于ZooKeeper、Consul等方式实现的配置中心，加密需要自行实现。

## 配置中心的高可用

spring-cloud-config作为配置中心，要保证server高可用，其背后的Git、SVN或DB也需要保证高可用。通过将server注册到服务中心，可以实现配置中心的高可用。

客户端也可以做高可用，将配置缓存到应用本地，当配置中心不可用时，可以回退使用本地的配置。

