# Spring Boot

Spring Boot项目并不提供能力，其核心功能是自动配置。Spring Boot出现是为了解决基于Spring技术体系的应用配置复杂的问题。使用Spring Boot可以摒弃烦人又过时的XML配置文件，可以从一而终使用注解来配置你的应用，甚至可以做到零配置，开箱即用。

## Spring Boot CLI

Spring Boot CLI是一个命令行工具，可以让开发者快速搭建出一个基于Spring Boot的项目。其作用有点类似于Maven的Aracgetype

## 自定义starter

~~尽管Spring Boot官方提供了许许多多的starter，方便我们使用其他框架时可以开箱即用。但有时候还是需要我们编写自定义starter来集成不太常见的框架。~~

~~在src目录下新建resources目录，在该目录下新建META-INF目录，在该目录下新建spring.factories文件。在该文件中指定自动配置的配置类的全限定名。使Spring Boot能定位，加载，使用用户自定义的自动配置类。~~

~~使用各种Codition编写AutoConfigure类。~~

### 2.x

2.x版本和1.x版本发生了很多变更。

创建自定义的starter之前，需要了解自动配置的相关概念。

1. 定位自动配置类的位置
2. 明白条件注解
3. 如何测试自动配置是否生效

Spring Boot会检查jar文件的==META-INF/spring.factories==文件。我们可以在这个文件中列出所有的需要使用到的自动配置类，例如：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

可以使用@AutoConfigureAfter或@AutoConfigureBefore注解来设置不同自动配置类的生效顺序。比如你需要设定依赖web的配置，你的自动配置类应该在WebMvcAutoConfiguration之后生效。还可以使用@AutoConfigureOrder来更精确生效顺序。

Spring Boot提供了很多很多的条件注解：

- 使用在类级别上的条件
- 使用在创建Bean上的条件
- 使用在属性上的条件
- 使用在资源上的条件
- 使用在Web应用的条件
- 使用了SpEL表达式的条件

之后我们就可以创建自定义starter了。

Spring Boot建议按照acme-spring-boot-starter来命名你的自定义starter。

Spring Boot建议将你的starter分成两个模块，一个为autoconfigure模块，一个是starter模块。autoconfigure模块包含自动配置类，starter模块是一个空的jar，只包含必要的依赖。当然也可以将两个模块合成一个模块。

## Spring Boot开发者工具

启用开发者工具，需要加入依赖：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
</dependency>
```

当将应用构建包时，开发者工具会被禁用，所以不需要手动移除依赖。

### 自动重启

当应用的classpath发生变化时，自定义的classloader重新加载新的类，达到热部署的功能。

以下目录发生变动时，不会触发自动重启

- /META-INF/resources
- /resources
- /static
- /public
- /templates

通过spring.devtools.restart.exclude来修改排除目录。

```yaml
spring:
	devtools:
		restart:
			exclude: /static/**,/templates/**
```

### LiveReload

LiveReload与自动重启功能一致，只不过面向的是静态资源的变动。要启用该功能，浏览器需要安装插件。安装之后，不用刷新浏览器就能看到修改后新的界面样式。

### 远程调试

需要配置一个连接密码：

```yaml
spring:
	devtools:
		remote:
			secret: myappsecret
```

然后应用以`org.springframework.boot.devtools.RemoteSpringApplication`来启动，并需要配置Java参数。

### 其他

使用开发者工具后，会讲一些模版引擎的缓存设置为关闭。

## Actuator

首先需要添加依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 端点

Actuator的端点可以让你监控和影响你的应用程序。Spring Boot内置了很多端点，你可以实现并添加自己的端点。

每个端点都可以设置启用/停用。端点通过JMX或HTTP来暴露，方便远程访问。

#### 启用端点



#### 暴露端点

鉴于一些端点可能包含敏感信息，暴露某些端点之前，需要仔细考虑是否有必要。

可以使用include和exclude来控制是否暴露端点。以下是Spring Boot的默认配置，表示所有的端点默认都通过JMX暴露，通过HTTP暴露的端点只有info和health两个。

| 属性                                      | 默认值       |
| ----------------------------------------- | ------------ |
| managment.endpoints.jmx.exposure.exclude  |              |
| managment.endpoints.jmx.exposure.include  | *            |
| managmenet.endpoints.web.exposure.exclude |              |
| managment.endpoints.web.exposure.include  | info，health |

*用于选择所有的端点。需要注意的是在YAML配置文件中， *有其他的含义，所以需要用双引号修饰起来，“ * “。

如果需要自定义端点暴露的策略，可以自定义注册`EndpointFilter`。

#### 保护基于HTTP暴露的端点

如果使用了Spring Security，端点默认会被Spring Seacurity的内容导航策略所保护。

### 基于HTTP暴露的端点的超链接

Spring Boot提供了一个页面，用于查看基于HTTP暴露出来所有的端点的超链接地址，该页面地址为/actuator。这个地址也是可以配置的。

#### CORS的支持

#### 自定义端点

#### 健康信息

#### 应用信息

