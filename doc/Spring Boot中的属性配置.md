# Spring Boot中的属性配置

Spring Boot允许你外部化你的配置，可以让你的应用在不同的环境中运行。可以使用properties文件，YAML文件，环境变量和命令行参数来外部化配置。属性值可以通过@Value注解，通过Spring的Environment抽象，或通过@ConfigurationProperties注解绑定到结构化对象来访问。

Spring Boot对PropertySource使用严格的顺序，以便合理的覆盖相同的属性值。

1. Devtool全局配置，在home目录下(~/.spring-boot-devtool.properties)
2. 测试类上的@TestPropertySource
3. 测试上的proerpties属性，@SpringBootTest
4. 命令行参数
5. 来自SPRING_APPLICATION_JSON
6. ServletConfig初始化属性
7. ServletContext初始化属性
8. 来自java:comp/env的JNDI属性
9. Java系统属性，System.getProperties()
10. 系统环境变量
11. 来自random.*的RandomValuePropertySource
12. 指定剖面应用配置，jar包外部的application-{profile}-properties或YAML变种
13. 指定剖面应用配置，jar包内部的application-{profile}-properties或YAML变种
14. jar包外部的应用配置，application.properties或YAML变种
15. jar包内部的应用配置，application.properties或YAML变种
16. @Configuration注解的类上的@PropertySource注解指定的属性
17. 默认配置，通过SpringApplication.setDefaultProperties指定

如果出现相同的属性，顺序在前的将会覆盖在后的属性。

## 配置文件中的随机数

RandomValuePropertySource对在机密或测试场景注入随机数非常有用。提供integer、long、uuid、string类型的随机数

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

## 命令行中的参数

默认情况下，Spring Boot会将命令行中的参数转换成property，并加入到Environment中（以--开始，比如--server.port=9090）。

如果不想接受来自命令行的属性，可以通过SpringApplication.setAddCommandLineProperties(false)来禁用。

## 应用属性文件

SpringApplication按照以下路径寻找application.properties文件，并获取其中的属性，并加入到Environment当中。

1. /config二级目录中
2. 当前目录中
3. classpath下/config包
4. classpath根目录

application.yml和application.properties是等价的

spring.config.name可以更改这个文件的名称，spring.config.location可以更改这个文件的路径

## 剖面指定的属性

application-{profile}.properties，来指定不同剖面的配置属性值。Environment有一个默认的属性值列表，默认叫default，如果没有激活任何一个剖面，application-default.properties将会被加载和使用其中的属性值。

如果有相同的属性，剖面指定的配置总会覆盖没有剖面指定中的配置，无论剖面指定的配置文件是在jar包外还是在jar包内。

## 属性中的占位符

application.properties文件中属性值可以使用占位符，来注入先前指定的属性值（比如System.properties），因为这个文件会被已存在的Environment过滤。

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

## 使用YAML替代Properties

YAML是JSON的超集，因此对于有上下层级的配置属性来说是一种很方便的格式。当你有Snake YAML依赖在classpath，SpringApplication会自动支持YAML。

SpringFramework提供两种方式加载YAML文档，YamlPropertiesFactoryBean加载YAML转换成Properties，YamlMapFactoryBean加载YAML转换成Map。

YAML列表通过以下方式表示：

```yaml
my:
servers:
- dev.example.com
- another.example.com
```

等价于：

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

如果使用Spring Boot的Binder工具集（@ConfigurationProperties）来绑定这个属性，Java类需要用list或set属性来接收。

### 将YAML的属性值暴露到Environment中

YamlPropertySourceLoader

### 多剖面的YAML文档

使用spring.profiles，在单个YAML文档中指定多个剖面。

```yml
server:
address: 192.168.1.100
---
spring:
profiles: development
server:
address: 127.0.0.1
---
spring:
profiles: production & eu-central
server:
address: 192.168.1.120
```

### YAML的缺点

YAML不能通过PropertySource来加载。在这种场景下，如果你需要使用属性值，那么你就只能用properties文件了。

## 类型安全的配置

使用@Value("${property}")来注入配置属性值，有时候不方便，当你的属性值带有上下层级的情况时。Spring Boot提供了替代方法，允许使用强类型的Java对象来管理和验证应用程序的配置属性值。

Java Bean需要有getter和setter方法。只考虑标准的Java Bean属性，静态属性不支持。

@Value和@ConfigurationProperties的区别：

| 特性       | @ConfigurationProperties | @Value |
| ---------- | ------------------------ | ------ |
| 松散绑定   | 支持                     | 不支持 |
| 元数据支持 | 支持                     | 不支持 |
| SpEL表达式 | 不支持                   | 支持   |

还需要通过@EnableConfigurationProperties注解来注册属性类。

推荐@ConfiguratoinProperties只处理属性，特别是不要讲context中的其他bean注入进来。

@EnableConfigurationProperties会自动的将那些通过Environment配置好的被@ConfigurationProperties注解的配置类应用到你的项目当中。这些配置类也可以注入到其他的被context上下文管理起来的bean当中。

通过在应用中加入的configuration-process注解，可以将@ConfigurationProperties注解的配置类生成一个元数据文件，该文件可以用于帮助IDE，当编写application.properties或application.yml文件时，可以更好的提供自动补全的功能。

### 流程

@EnableConfigurationProperties来启用

当Spring处理到该注解时，Import其中的EnableConfigurationPropertiesImportSelector

如果@EnableConfigurationProperties没有指定配置类，向Spring容器注册ConfigurationPropertiesBindingPostProcessorRegistrar

如果指定了配置类，除此之外，还会注册一个ConfigurationPropertiesBeanRegistrar。

1. ConfigurationPropertiesBeanRegistrar类中，会将EnableConfigurationProperties中的配置类拿到，然后生成一个名称：prefix+“-”+类的全限定名。然后生成BeanDefinition，以BeanDefinition的方式注册到Spring容器中
2. ConfigurationPropertiesBindingPostProcessorRegistrar类中，注册另外两个类到Spring容器中：一个是ConfigurationBeanFactoryMetaData，另一个是ConfigurationPropertiesBindingPostProcessor

ConfigurationBeanFactoryMetaData是BeanFactoryPostProcessor，会在BeanFactory做一层拦截，处理容器里的BeanDefinition。

ConfigurationPropertiesBindingPostProcessor是BeanPostProcessor，会在容器内的Bean实例化之后初始化之前做一些处理。

ConfigurationBeanFactoryMetaData只是将容器中的Factory Bean保存到一个map中，以便后续ConfigurationPropertiesBindingPostProcessor处理的时候能够更快速。

ConfigurationPropertiesBindingPostProcessor过滤每一个容器中的bean，当Bean有被@ConfigurationProperties注解时，创建一个PropertiesConfigurationFactory，并设置好参数，然后委托PropertiesConfigurationFactory来负责将PropertySources中的属性值注入到该Bean中。

## 松散绑定

RelaxedDataBinder继承自DataBinder，负责将应用中的配置信息，附加到配置文件类上（@ConfigurationProperties注解的类）。RelaxedDataBinder除了能将不同命名风格的配置映射到对应的配置文件类的属性中，还能进行类型转换。比如可以将String类型的配置，转换成Spring中的Resource对象，或Class对象。

## 构建时扩展配置属性

可以将构建的信息注入到配置当中，通过@占位符，比如：

```properties
app.encoding=@project.build.sourceEncoding@
app.java.version=@java.version@
```

src/test/resources的将不会生效

前提是使用了spring-boot-starter-parent，如果没有，那么需要做一些额外的配置

如果使用Gradle作为构建工具，\${..}占位符给Gradle使用，所以Spring的占位符变成了\\\${...}。

## SpringApplication的配置

SpringApplication提供了很多的配置，可以通过Java API的方式来控制，也可以在application.properties中控制。比如

```properties
spring.main.web-application-type=none
spring.main.banner-mode=off
```

如果通过Java API也配置相同的属性，那么会被这里的覆盖。