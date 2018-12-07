# Redis

Redis是一款高性能缓存中间件，使用场景广泛。

Redis支持以下特性：

- 订阅/发布
- Lua脚本
- 管道
- 事务

## 使用场景

- 分布式锁
- 高速缓存
- 全局计数器
- 消息队列

## 常见配置

### 管道

Redis的管道（Pipelining）允许一次发送多个命令，节省传输时间。

Redis是基于TCP的服务，使用客户端-服务器模式，称之为请求-响应模型。

一般来说，一个请求完成需要以下两个步骤：

1. 客户端发送请求，通常采用阻塞的形式从socket中等待读取来自服务端相应的数据
2. 服务端处理命令然后发送响应到客户端

客户端和服务端之间通过网络进行连接，这可以非常快，也可以非常慢。无论网络的延迟如何，数据包从客户端到服务端再到客户端，都需要时间。

这一次的往返时间称之为RTT。当客户端需要连续执行多个请求时，对性能的影响将会被放大。

#### 脚本和管道

### 缓存过期策略

当将Redis用作缓存时，通常会涉及到过期策略。常见的有LRU：按照创建时间删除旧的数据。Redis 4.0之后还支持了LFU。

可以在redis.conf，或通过`config set`命令为redis设置内存大小。比如在redis.conf中设置100mb大小：

```
maxmemory 100mb
```

如果`maxmemory`设置成0，表示无限制。

当达到指定容量时，可以选择不同的行为，称之为策略。可以配置成返回错误，或者删除一些旧数据。

`maxmemory-prolicy`可以配置不同的过期策略：

- noeviction：当达到内存限制时，对新增等一些命令返回错误
- allkeys-lru：尝试删除最近较少使用的key
- volatile-lru：尝试删除最近较少使用并设置了过期时间的key
- allkeys-random：
- volatile-random
- volatile-ttl

### 事务

### 发布/订阅

### 事件通知

### 分布式锁

### 二级索引

## 管理

## 嵌入式和物联网
