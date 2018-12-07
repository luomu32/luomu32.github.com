#Mybatis

**SqlSession**表示单次与数据库交互的会话。当创建一个SqlSession时，保存一份Configuration的引用。根据传入的一个ID找到对应的Mapper，找到并组织出对应的SQL语句。在Web型应用中，其作用范围与Request一致。通常一个请求中或操作多次数据库，就可以共享同一个SqlSession实例。

**SqlSessionFactory**根据合适的策略创建一个Session。**SqlSessionFactoryBuilder**，构造者模式，创建SqlSessionFactory

**Configuration**表示Mybatis全局的配置。内部维护了很多东西。包括维护一个缓存，作为二级缓存，缓存应用级别的数据。

Mapper，并不是一个类，而是一个概念。是一个数据库操作语句的集合。一般一个Domain对应一个Mapper。在早期的Mapper，只有xml文件来表示，后来的版本中支持Java类表示，也可以两者混用。所有可用的Mapper集合维护在Configuration对象中。

**Interceptor**表示一个插件



**SqlSource**

**Executor**负责执行具体的SQL。

- SimpleExecutor
- ReuseExecutor
- BatchExecutor
- CachingExecutor

CachingExecutor顾名思义，在执行SQL语句之前，处理缓存以支持缓存。其具体的SQL执行委托给其他Executor实现负责。

**StatementHandler**负责创建JDBC层的Statement，并设置SQL语句所需的参数

**TypeHandler**负责将返回结果集中的数据转换成指定Java类型。Mybatis内置了很多TypeHandler，可以转成Date、Integer、Enum等等。如果使用Java 8，对于新的类型，需要注册新的TypeHandler。当然也可以自行实现特殊的转换器。

##简单使用流程

1. 根据configuration.xml文件生成Configuration对象，或通过Java API的方式来控制Mybatis的全局配置。如果不指定Configuration，Mybatis会生成一个默认的Configuration配置，所以这一步其实也不是必须的。
2. 根据构造好的Configuration对象，设置好映射文件路径（.xml）后，就可以构造出一个SqlSessionFactory。
3. 每次需要与数据库交互时，再根据SqlSessionFactory获取SqlSession，通过SqlSession执行SQL。

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  Blog blog = session.selectOne(
    "org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}
```

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```

##与Spring的集成

Mybatis官方提供了`mybatis-spring`模块专门负责与Spring容器的集成。提供几个类：

- SqlSessionFactoryBean
- MapperScannerConfigurer
- MapperScan

SqlSessionFactoryBean是采用Spring的FactoryBean机制，负责创建SqlSessionFactory。MapperScannerConfigurer主要负责扫描@Mapper注解的Java类，与注解@MapperScan的作用一致。

##与Spring Boot的集成

Mybatis官方提供了`mybatis-spring-boot-starter`模块专门负责与Spring Boot的集成。

## 执行流程

```sequence
Client->Service:query,insert,update,delete
Service->SqlSessionFactory:create SqlSession
SqlSessionFactory->SqlSessionFactory:create TransactionFactory
SqlSessionFactory->Configuration:create Executor
Configuration->Configuration:create Executor by ExecutorType
Configuration->Configuration:create proxy wapper Executor by plugin
Configuration-->SqlSessionFactory:return Executor
SqlSessionFactory->SqlSessionFactory:create SqlSession by Executor and Configuration
SqlSessionFactory-->Service:return SqlSession
Service->SqlSession:selectOne,SelectList,insert,update...
SqlSession->Configuration:build MappedStatement
Configuration-->SqlSession:return MappedStatement
SqlSession->Executor:update,insert,query,delete
Executor->Configuration:create StatementHandler
Configuration-->Executor:return StatementHandler
Executor->StatementHandler:prepare
Executor->StatementHandler:parameterize
Executor->StatementHandler:query,update
StatementHandler-->Executor:result
Executor-->SqlSession:result
SqlSession-->Service:result
```

## XML Mapper文件解析流程

为SqlSessionFactory指定Mapper文件地址后，交由XMLMapperBuilder解析。

XMLMapperBuilder又委托XMLStatementBuilder去解析语句的部分，`<select>、<insert>、<update>、<delete>`的部分。

XMLStatementBuilder又委托MapperBuilderAssistant将结果保存到MappedStatement

将所有的MappedStatement保存在Configuration中

##缓存

按照范围的不同，可以分为SqlSession级别和SqlSessionFactory级别的。

SqlSession级别的缓存，当SqlSession commit或关闭时，不可用。

##插件

SqlSessionFactory在创建SqlSession时，会从Configuration中获取Executor。而Configuration根据ExecutorType创建合适的Executor，然后根据内部维护的插件链，对生成好的Executor层层生成代理。

##VFS

用于加载文件。用于与Spring的ResourceLoader一致。由于Spring Boot项目打包出的jar文件目录结构比较特俗，如果使用Mybatis的VFS来加载Mapper文件，会出现加载不到的问题，这时候就需要SpringBootVFS。