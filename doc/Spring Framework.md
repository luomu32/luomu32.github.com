# Spring Framework

##核心

###Resource接口

Resource接口表示一个资源的抽象，是在InputStream之上的抽象。有以下几种：

- UrlResource
- ClasspathResource
- FileSystemResource
- ServletContextResource
- InputStreamResource
- ByteArrayResource

ResourceLoader则是用于获取Resource的接口。ResourceLoaderAware接口可以获取到当前容器中的ResourceLoader。

PathMatchingResourcePatternResolver是常用的一个ResourceLoader实现。

## MessageSource接口

用于资源国际化，基于Java的ResourceBundle和MessageFormat，进行了进一步封装。

`ResourceBundleMessageSource`和`ReloadableResourceBundleMessageSource`两个实现。在配置MessageSource时，Bean的名称必须为messageSource，不然其他Bean无法注入有效的MessageSource。

## MessageCoudeResolver

用于JSR 303 Bean Validation，验证结果的错误代码生成规则。比如按照错误类型+被校验类名+验证失败字段名。

###应用启动流程

不同类型的应用完整的启动顺序略有不同。比如基于Spring Boot的应用，由DispatchServlet启动的基于Spring MVC的应用，测试环境启动的应用等等。

ApplicationContext是Spring容器的最基础的接口。实现该接口的子类也有很多，不同的实现，启动顺序也是有所不同。首先分为ConfigurableApplicationContext和WebApplicationContext两类，一个代表普通的容器，一个代表Web容器

WebApplicationContext下有分为ConfigurableWebApplicationContext和StubWebApplicationContext，前者表示可配置的容器，后者是测试用的容器。

ConfigurableWebApplicationContext下又分为GenericWebApplicationContext和AbstractRefreshableWebApplicationContext两类。前者表示固定的容器，后者可以动态注册新的Bean及一些Processor等，通过refresh方法刷新的容器。

根据配置的方式的不同，有可以分为Xml、Annotation等类型容器，比如AbstractRefreshableWebApplicationContext这个类别下就有：

- XmlWebApplicationContext
- GroovyWebApplicationContext
- AnnotationConfigWebApplicationContext

####Spring Boot应用

这里的应用指的是Application，容器指的是BeanFactory

1. 选择一个合适的应用类型，如果是Web类型的应用，采用AnnotationConfigEmbeddedWebApplicationContext，如果是非Web类型的应用，采用AnnotationConfigApplicationContext。当然可以手动指定应用类型，但必须是ConfigurableApplicationContext的子类。选择好之后，调用构造方法实例化。此时，容器也已经实例化好了。
2. 开始准备应用：
   1. 设置Environment
   2. 向容器注册BeanNameGenerator和ResourceLoader
   3. 遍历注册的ApplicationContextInitializer扩展，对应用进行处理
   4. 遍历注册SpringApplicationRunListener扩展的contextPrepared方法，广播应用准备事件
   5. 向容器注册ApplicationArguments
   6. 向容器注册Banner
   7. 创建BeanDefinitionLoader，设置Source，然后设置Environment、BeanNameGenerator和ResourceLoader。然后加载Source
   8. 遍历注册的SpringApplicationRunListener扩展的contextLoaded方法，广播应用被加载事件
3. 刷新应用
4. 刷新应用后：
   1. 遍历应用中注册ApplicationRunner和CommandLineRunner，回掉其中的run方法
5. 遍历注册SpringApplicationRunListener扩展的finished方法，广播应用完成事件

一共大体五个步骤，其中第三步是最核心的，也是最复杂的。第三步调用的是应用的refresh方法来刷新应用，这一步里的流程就跟Spring MVC应用，或直接通过new实例化应用来设置应用的步骤和过程就没有区别了。

####应用的refresh方法

refresh方法在AbstractApplicationContext中，提供了主要的应用和容器准备工作，也提供了一些方法，用于子类去扩展。

1. 准备刷新

2. 获取容器

3. 准备容器，为容器设置ClassLoader，设置SpEL表达式解析器，设置ApplicationContextAwareProcessor用来处理实现了Aware接口的类，并过滤几个Aware接口注入

4. 容器后置处理，该方法AbstractApplicationContext并没有提供实现，主要为子类扩展使用。

   AnnotationConfigEmbeddedWebApplicationContext在这个方法中，调用内部的scaner扫描指定包中的Bean，并生成BeanBeanDefinition，并注册到BeanFactory中。

5. 处理BeanFactoryPostProcessor

6. 处理BeanPostProcessor

7. 初始化MessageSource

8. 初始化ApplicationEventMulticaster

9. 重刷新，该方法AbstractApplicationContext并没有提供实现，主要为了子类扩展使用

10. 注册ApplicationListener

11. 准备完成容器初始化，设置容器的冻结标识为true，开始实例化容器中未标识lazy-init的单例对象

12. 完成刷新，初始化LifecycleProcessor，调用LifecycleProcessor的onRefresh，发布ContextRefreshedEvent事件

####Spring内置的BeanFactoryPostProcessor

BeanFactoryPostProcessor除了直接子类之外，还有一个扩展接口：BeanDefinitionRegistryPostProcessor。

BeanFactoryPostProcessor一般用于修改已经注册到容器中的BeanDefinition。而BeanDefinitionRegistryPostProcessor可用于向容器注册新的BeanDefinition。

#####ConfigurationClassPostProcessor

用于处理被@Configuration注解的配置类。

####Spring内置的BeanPostProcessor

BeanPostProcessor除了直接子类之外，还有扩展接口：InstantiationAwareBeanPostProcessor

InstantiationAwareBeanPostProcessor这个接口还有扩展接口：SmartInstantiationAwareBeanPostProcessor

SmartInstantiationAwareBeanPostProcessor这个接口有一个重要的方法：determineCandidateConstructors。它是根据已知的一个Bean的名称和类型，选择一个合适的构造方法，用于实例化该Bean。

###Bean的实例化

获取一个Bean的带参数的构造方法

- 如果有，
- 如果没有，通过无参数的构造方法实例化对象

####Bean中属性值的注入

我们可以使用@Value注解对Bean的属性进行注入。

## Web

###WebFlux

自从Spring 5之后，Spring提供了响应式的方式来开发Web应用。基于响应式的方式，底层通信可以是Netty、Undertow或基于Servlet 3.1+的容器。响应式的在webflux项目中，与webmvc区分开。两者也可以混合使用。

据说要想理解响应式编程，需要先理解Java 8中的Lambda，Stream流编程，再理解函数式编程，再理解Java 9中的响应式流Flow

![](https://spring.io/img/homepage/diagram-boot-reactor.svg)



基于响应式的是非阻塞的，而之前的是异步阻塞的。响应式的应该吞吐量优于非响应式的。

####为什么创建响应式的编程模型

一个原因是相对基于Servlet的Web而言，使用较少的线程来处理相同的并发。虽然Servlet 3.1+提供了非阻塞的方式，但是不是完全是非阻塞的，有些是同步的，有些是阻塞的（getParameter或getPart）。

另一个原因是提供了函数式编程方式。通过Java 8中提供的Lambda表达式，在Java中创建函数式的API，通过CompletableFuture和ReactiveX，实现非阻塞应用和continuation风格（函数式编程中较为常见的风格）的API。

####何为响应式

在同步命令式调用中，阻塞调用是一种自然而然的背压。在非阻塞代码中，控制发布者生成事件的速率就很关键，防止事件的目的地（订阅者）被大量事件淹没（跟不上处理速度）。

Reactive Streams是一个小规范（Java 9中有部分支持，`java.util.concurrent.Flow`）。用于定义具有背压（back pressure）的异步组件之间的交互，让订阅者控制发布者生成数据的速度。

> 如果发布者无法慢下来怎么办？
>
> Reactive Streams只是建立机制和边界。如果发布者无法慢下来，只能选择缓冲，丢弃还是失败。

####响应式API

Reactor是Spring WebFlux采用的响应式库。提供Mono和Flux。

####编程模型

web模块提供基础功能，webmvc和webflux两个模块共用。

在此基础上，Spring WebFlux提供了两种编程模型：

- 注解式的控制器：使用方式与WebMVC一致。
- 函数式端点：基于Lambda轻量级函数式编程模型，与上面最大的不同是负责从头到尾的请求处理，而不是等待回调。

####适用性

如何选择使用Spring Web MVC还是Spring WebFlux。

- 如果应用运行正常，就无须更改。命令式编程是编写、理解和调试代码最简单的方法。可选择的库也更多，大多数都是阻塞的。
- 如果选择响应式，Spring WebFlux支持多种容器，两种编程模型，和多种响应式库来实现
- 如果对基于Java 8 Lambda或Kotlin实现的轻量级，函数式web框架感兴趣，可以使用函数式端点编程模型。
- 在微服务框架中，Web MVC和WebFlux可以混合使用。Web MVC与WebFlux很多使用方式一致，减少使用学习成本。
- 检查依赖，如果使用到了阻塞的API，那么选择Web MVC。虽然在阻塞线程上可以新开线程使用响应式编程，但是这样响应式所带来的好处大大减少。
- 如果需要调用远程服务，推荐使用非阻塞的WebClient。WebClient也可以用在Web MVC中。
- 如果在一个大型项目中，需要注意响应式带来的学习成本，可以先只使用响应式的WebClient。也可以从小的项目做起。对于大项目，转换没有必要。最好之前能理解非阻塞IO的原理和影响（比如单线程的Node.js中的并发）。

####性能

响应式和非阻塞带来的好处是能够使用少量固定数量的线程和更少的内存。使用应用程序在负载下更具有弹性，因为可以以更可预测的方式扩展。

####并发模型

###WebMVC

`DispatcherServlet->HandlerMapping->HandlerAdatper->Interceptor->ArgumentResolver->ViewResolver`

DispatchserServet负责处理所有的HTTP请求，并根据一定规则进行分发到不同的控制器处理。其父类负责拉起、初始化Spring的ApplicationContext。

Spring提供一些Servlet的Filter，比如处理请求传参乱码的Filter，处理PUT类型请求支持application/x-www-form-urlencoded格式的参数等。

####控制器

支持的参数类型：

- Request、Response，比如ServletRequest或HttpServletRequest
- HttpSession，永不为null。Session可能不是线程安全的，需要将`RequestMappingHandlerAdapter`的synchronizeOnSession标识设置为true来保证线程安全。
- WebRequest或NativeWebRequest。也可以像request/session一样访问到请求中的参数
- Locale，当前线程级别的Locale
- TimeZone：Java6+或ZoneId：Java8+，访问到时区，根据LocaleContextResolver决定哪个时区。
- InputStream或Reader，访问请求中的内容
- OutputStream或Writer，用于向Response输出响应内容
- HttpMethod，请求的HTTP Method信息
- Principal，当前被认证的用户
- @PathVariable注解的值，可以从URL路径中获取到参数
- @MatrixVariable注解的值，URL中键值对参数
- @RequestParam注解的值，获取请求中的参数，会自动将String类型转换成对应的类型
- @RequestHeader注解的值，获取请求头的参数，类型也会自动转换
- @RequestBody注解的值或对象，获取请求体中的参数，会通过HttpMessageConvert将参数转换组织成对应的对象
- @RequestPart注解的，用于获取上传文件
- @SessionAttribute注解，访问Session中的值
- @RequestAttribute注解，访问请求属性中的值
- HttpEntity<?>，方便访问请求头和请求内容，请求流会被HttpMessgaeConverter转换成对应的实体类
- Map，Model，ModelMap。用于将控制器处理好的结果内容输出到页面中
- RedirectAttributes，用于重定向后的穿参
- 普通对象，带有setter方法。请求中的参数会绑定到对象中对应的属性上。具体参考`RequestMappingHandlerAdapter`中的webBindingInitializer属性的配置。
- Errors或BindingResult，用于存放对象验证结果
- SessionStatus，通过调用该对象对应方法，来表明处理是否结束，来触发session中属性值的清空
- UriComponentsBuilder，用于构造URL，基于当前请求的主机、端口、scheme，上下文路径等

Java 8的Optioanl支持，如果使用Optional包裹的值，表明参数是可选的，等同于@RequestParam(required=false)。

支持的返回值类型：

请求映射：`RequestMappingHandlerMapping`负责查找@ReuqestMapping（@GetMapping、@PutMapping、@PostMapping、@DeleteMapping），来维护一组请求URL与处理器（控制器）的映射关系。当请求过来时，`RequestMappingHandlerMapping`就能找到的合适的处理器（控制器）来处理请求。

#### CORS

默认情况下，浏览器出于安全的考量不允许AJAX进行跨域名请求的。跨源资源共享是被大多数浏览器支持的W3C的规范。可以灵活的配置以支持哪种跨域请求，而不是使用安全性较低的且功能较弱的hack，比如iframe或jsonp。

SpringFramework 4.2之后，支持CORS开箱即用。接口`CrosProcessor`，默认实现`DefaultCrosProcessor`，会将`Access-Control-Allow-Origin`加入到响应头中。

@CrossOrigin注解可以加到方法上，方法就支持所有来源和所有HTTP Method。@CrossOrigin可以可以加在类上。

也可以全局配置启用：

```java
 	@Override
    protected void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**");
    }
```

提供`CorsFilter`，支持使用Spring Security的场景，确保`CorsFilter`在Filter最前。

####Formatter和Converter

两者都用与转换对象。Formatter只能将String转换成Java对象，适用于Web。Converter更灵活，能将任意对象转换成其他任意对象，适合应用的任何层。

####HttpMessageConvert

####异常处理

DefaultHandlerExceptionResolver，处理Sprin定义的一些关于请求参数等错误的异常处理。