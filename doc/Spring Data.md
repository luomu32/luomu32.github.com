# Spring Data

- Spring Data JPA
- Spring Data Redis
- Spring Data Mongodb
- Spring Data Elasticsearch

## JPA

`EntityManager`。

### 查询

Spring Data JPA支持多种风格的查询方式：

- NamedQuery、
- Example、
- Querydsl。通过其流畅的API来构建强类型的类SQL查询语言。`QuerydslPredicateExecutor`。
- DDD风格的Specification

#### 查询方法

根据一定的规则的方法名称，Spring Data可以组织成对应的查询方法。比如find开头的方法表明为查询，方法名中查询条件参数之间按照驼峰命名规则区分，方法参数与方法名中的字段一致就可以自动匹配上。

除了可以在方法名上指定查询条件之外，还可以使用`distinct`、忽略大小写、`order by`、`limit`等特性

参数除了支持查询条件，还支持`Pageable`和`Sort`	

还可以使用@Query注解，在注解中使用更为灵活的类原生SQL语法，只是表名用实体类名替换，表字段名用实体类字段替换。

@Query注解用于资源库的方法上。

配合@Param，可以使用命名参数，而不是靠顺序。

@Query内还支持SpEL表达式

@Query配合@Modifying，用于update、insert、delete

##### 投影

基于接口的投影。定义一个接口，接口内定义需要的查询结果字段的访问方法，比如username这个属性，那么在接口中定义`String getUsername()`这个方法。这个接口可以直接用于资源库的查询方法的返回值，投影自动完成。接口除了支持基本数据类型，还支持复合对象。

封闭投影与开放投影。接口中可以使用@Value注解，可以使用SpEL表达式，可以将多个字段合并成一个字段，这种使用了@Value注解的称之为开放投影，Spring Data无法做优化。其他的称之为封闭投影，Spring Data可以针对的优化。

@Value中的表达式最好不要太复杂，一种方式使用Java 8的接口默认方法，另一种是将这部分逻辑放到其他Bean中，表达式中调用这个Bean的方法。

@Vaule表达式还支持调用方法的参数。也就是接口中的方法可以带上参数，然后被表达式使用。

基于类的投影（DTO），Date Transfer Object，数据传输对象。与接口的投影不同的是，没有代理，也不支持嵌套的投影。使用方法与接口投影差不多，定义与目标对象一致的字段名称，并暴露geter方法。

为了避免样板代码，可以使用Lombok的@Value注解定义DTO，自动将所有字段定义为final，暴露一个包含所有字段的构造方法，暴露所有字段的setter方法，自动生成equals和hashcode方法。

动态投影。在调用的时候动态确定投影。可以为资源库的查询方法多添加一个`Class<T> type`参数，根据不同的参数返回不同的投影。

#### NamedQuery（命名查询）

这个是JPA的特性。可以将命名查询定义在一个特定名称的XML文件中，orm.xml，或使用@NamedQuery。语法类SQL，只不过用实体类名和实体字段名。@NamedNativeQuery可以使用原生的SQL。@NamedQuery或@NamedNativeQuery用于实体类上

#### Example

Spring Data提供的一种简易的查询方式。特点是简单直白，能够根据查询条件对象根据规则动态的创建查询条件，非常类似Mybatis的if-else动态查询机制。

缺点是：

- 不支持嵌套和分组
- 对于字符串类型支持模糊匹配和精确匹配，对于其他类型只支持精确匹配。

分为两个部分，查询条件对象，匹配类型，根据这两个类型就可以构造一个Example查询对象。

能不能扩展Example？以支持Date（LocalDate）类型的区间匹配？

#### QueryDSL

需要额外添加依赖。

pom.xml中添加对应的maven plugin，会自动将每个@Entity注解的实体类生成对应的查询对象。

另外资源库需要继承`QuerydslPredicateExecutor`。

####DDD

DDD-领域驱动设计中，定义了一些实体持久化的方法论和概念。Spring Data中有部分提供了支持

####聚合根发布事件。

聚合根的概念：一个实体是一个聚合根。实体内的嵌套对象是聚合。聚合没有表，没有唯一ID，脱离了聚合根失去了意义。数据层面上，聚合没有表，与聚合根共用一张表，也就是说聚合无法单独获取，必须通过聚合根获取。

#### 规格（Specifications）

JPA2定义了criteria API，可以编程式动态构建查询。criteria可以作为实体类的predicate（谓语？断言？）。

Spring Data接受了《领域驱动设计》一书中Specifications的概念。资源库通过继承`JpaSpecificationExecutor`接口来支持Specifications。方法接受`Specification`参数。

### Web支持

使用@EnableSpringDataWebSupport注解来开启。会注册一些组件：

- DomainClassConverter类，可以使Spring MVC通过请求参数或请求路径中的参数构造并实例化受实体容器管理的实体类。

```java
@Controller
@RequestMapping("/users")
class UserController {

  @RequestMapping("/{id}")
  String showUserForm(@PathVariable("id") User user, Model model) {

    model.addAttribute("user", user);//user对象将通过id从数据库中获取
    return "userForm";
  }
}
```

- HandlerMethodArgumentResolver类，可以从请求参数中构造并实例化`Pageable`和`Sort`对象。
- QuerydslPredicateArgumentResolver类，

Pageable，参数：page和size，page默认从0开始，size默认是20。

Sort，参数：`property,property(,ASC|DESC)`，比如`sort=firstname&sort=lastname,asc`

事务、锁、

###审计

提供@CreatedBy、@LastModifiedBy、@CreatedDate、@LastModifiedDate四个注解，用于记录实体的创建人，创建时间，最近一次修改人及最近一次修改时间。

除了使用注解，还可以通过实现一个接口的形式来实现：`Auditable`。

@EntityListeners

@EnableJpaAuditing