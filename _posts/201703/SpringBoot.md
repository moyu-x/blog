---
title: Spring Boot学习笔记——基于程序员DD的博客
date: 2017-03-03 22:34:48
tags:
  - Spring boot
  - 笔记
categories:
  - 笔记
---

# 前言

对于Spring boot的学习，我推荐的是看Spring boot的官方文档，这是最好的选择，因为相关方面的书不怎么好。关于实战的话，《Spring boot实战》作为一个实战类的书还行，对其他方面的解释还不行，得看文档。

我这个笔记是看的是[程序猿DD](http://blog.didispace.com/)的博客来进行写的，主要是他的博客中讲了，数据源、日志及缓存方面的东西，所以我的这个笔记可以说是我看程序猿DD博客的缩减版本。

# Spring boot

## 基础

spring-boot-starter：核心模块，包括自动化配置支持，日志和YAML

spring-boot-test：测试模块

配置文件位于application.properties或application.yml中

配置文件的用处：

1. 自定义的属性与加载——在配置文件定义完成之后使用`@Value(${属性名})`进行使用
2. 参数间的引用
3. 使用随机数——可以使用${random}来产生int值，long值或值String字符串
4. 可以通过命令行设置属性值
5. 多环境配置

在配置文件中通过spring.profile.active属性来设置对应的profile值

@RestController：Spring4新加注解，取代之前的@Controller和@RequestBody配合返回JSON的写法

## 文档注释

使用Swagger2构建API文档的步骤：

1. 添加Swagger2依赖
2. 创建Swagger2的配置类——在Application.java同级别下创建配置类Swagger2，，通过`@Configuration`注解，让Spring来加载该类配置。再通过`@EnableSwagger2`注解来启用Swagger2。再通过`createRestApi`函数创建`Docket`的Bean之后，`apiInfo()`用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。`select()`函数返回一个`ApiSelectorBuilder`实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被`@ApiIgnore`指定的请求）。
3. 添加上文档
   1. 通过@ApiOption注解来个API增加说明
   2. 通过`@ApiImplicitParams`、`@ApiImplicitParam`注解来给参数增加说明。

## 统一异常处理

1. 创建全局异常处理类：通过使用`@ControllerAdvice`定义统一的异常处理类，而不是在每个Controller中逐个定义。`@ExceptionHandler`用来定义函数针对的异常类型，最后将Exception对象和请求URL映射到`error.html`中
2. 实现error.html页面

我们只需要在Controller中抛出Exception，当然我们可能会有多种不同的Exception。然后在@ControllerAdvice类中，根据抛出的具体Exception类型匹配@ExceptionHandler中配置的异常类型来匹配错误映射和处理。

返回JSON格式：

本质上，只需在`@ExceptionHandler`之后加入`@ResponseBody`，就能让处理函数return的内容转换为JSON格式。

## 使用JdbcTemplate访问数据库

数据源的配置：

1. 连接数据库
2. 在application.properties中配置数据源信息
3. 使用JdbcTemplate操作数据库


### 数据访问

通过整合Hibernate之类的ORM框架，我们以操作Java实体的方式最终将数据改变映射到数据库表中。

`spring.jpa.properties.hibernate.hbm2ddl.auto`是hibernate的配置属性，其主要作用是：自动创建、更新、验证数据库表结构。该参数的几种配置如下：

- `create`：每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。
- `create-drop`：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
- `update`：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
- `validate`：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

创建一个Spring配置类，定义两个DataSource用来读取`application.properties`中的不同配置。如下例子中，主数据源配置为`spring.datasource.primary`开头的配置，第二数据源配置为`spring.datasource.secondary`开头的配置。

对JdbcTemplate的支持比较简单，只需要为其注入对应的datasource即可

新增对第一数据源的JPA配置，注意两处注释的地方，用于指定数据源对应的`Entity`实体和`Repository`定义位置，用`@Primary`区分主数据源。

操作数据的方式：

1. 使用默认接口
2. 通通过解析方法名创建查询
3. 使用`@Query`注解来创建查询

在JdbcTemplate的多数据源的支持可以使用`@Qualifier`来标识就行

Spring JPA中对于多数据源的支持可以使用DataSource Cofing来完成

### 整合MyBatis

1. 引入整合MyBatis的核心依赖mybatis-spring-boot-starter，这里不引入spring-boot-starter-jdbc依赖，是由于mybatis-spring-boot-starter中已经包含了此依赖
2. 配置数据库的连接

#### MyBatis注解配置：

传递参方式：

1. 使用`@Param`
2. 使用Map
3. 使用对象

增删改查：`@Insert`、`@Delete`、`@Update`、`@Select`

对于增、删、改操作相对变化较小。而对于“查”操作，我们往往需要进行多表关联，汇总计算等操作，那么对于查询的结果往往就不再是简单的实体对象了，往往需要返回一个与数据库实体不同的包装类，那么对于这类情况，就可以通过`@Results`和`@Result`注解来进行绑定

## 事物管理

事务的作用就是为了保证用户的每一个操作都是可靠的，事务中的每一步操作都必须成功执行，只要有发生异常就回退到事务开始未进行操作的状态。

通过单元测试演示了如何使用`@Transactional`注解来声明一个函数需要被事务管理，通常我们单元测试为了保证每个测试之间的数据独立，会使用`@Rollback`注解让每个单元测试都能在结束时回滚。而真正在开发业务逻辑时，我们通常在service层接口中使用`@Transactional`来对各个业务逻辑进行事务管理的配置

### 隔离级别

隔离级别是指若干个并发的事务之间的隔离程度，与我们开发时候主要相关的场景包括：脏读取、重复读、幻读

我们可以看`org.springframework.transaction.annotation.Isolation`枚举类中定义了五个表示隔离级别的值：

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);
}
```

- `DEFAULT`：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是：`READ_COMMITTED`。
- `READ_UNCOMMITTED`：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
- `READ_COMMITTED`：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- `REPEATABLE_READ`：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- `SERIALIZABLE`：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

指定方法：通过使用`isolation`属性设置，例如：

```java
@Transactional(isolation = Isolation.DEFAULT)
```

### 传播行为

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。

我们可以看`org.springframework.transaction.annotation.Propagation`枚举类中定义了6个表示传播行为的枚举值：

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```

- `REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- `SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- `REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- `NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。
- `NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`REQUIRED`。

指定方法：通过使用`propagation`属性设置，例如：

```java
@Transactional(propagation = Propagation.REQUIRED)
```

## Spring Security

- 通过`@EnableWebSecurity`注解开启Spring Security的功能
- 继承`WebSecurityConfigurerAdapter`，并重写它的方法来设置一些web安全的细节
- `configure(HttpSecurity http)`方法通过`authorizeRequests()`定义哪些URL需要被保护、哪些不需要被保护。例如以上代码指定了`/`和`/home`不需要任何认证就可以访问，其他的路径都必须通过身份验证。通过`formLogin()`定义当需要用户登录时候，转到的登录页面。
- `configureGlobal(AuthenticationManagerBuilder auth)`方法，在内存中创建了一个用户

## 缓存

### 缓存注解

- `@CacheConfig`：主要用于配置该类中会用到的一些共用的缓存配置。在这里`@CacheConfig(cacheNames = "users")`：配置了该数据访问对象中返回的内容将存储于名为users的缓存对象中，我们也可以不使用该注解，直接通过`@Cacheable`自己配置缓存集的名字来定义。
- `@Cacheable`：配置了findByName函数的返回值将被加入缓存。同时在查询时，会先从缓存中获取，若不存在才再发起对数据库的访问。该注解主要有下面几个参数：
  - `value`、`cacheNames`：两个等同的参数（`cacheNames`为Spring 4新增，作为`value`的别名），用于指定缓存存储的集合名。由于Spring 4中新增了`@CacheConfig`，因此在Spring 3中原本必须有的`value`属性，也成为非必需项了
  - `key`：缓存对象存储在Map集合中的key值，非必需，缺省按照函数的所有参数组合作为key值，若自己配置需使用SpEL表达式，比如：`@Cacheable(key = "#p0")`：使用函数第一个参数作为缓存的key值，更多关于SpEL表达式的详细内容可参考[官方文档](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html#cache-spel-context)
  - `condition`：缓存对象的条件，非必需，也需使用SpEL表达式，只有满足表达式条件的内容才会被缓存，比如：`@Cacheable(key = "#p0", condition = "#p0.length() < 3")`，表示只有当第一个参数的长度小于3的时候才会被缓存，若做此配置上面的AAA用户就不会被缓存，读者可自行实验尝试。
  - `unless`：另外一个缓存条件参数，非必需，需使用SpEL表达式。它不同于`condition`参数的地方在于它的判断时机，该条件是在函数被调用之后才做判断的，所以它可以通过对result进行判断。
  - `keyGenerator`：用于指定key生成器，非必需。若需要指定一个自定义的key生成器，我们需要去实现`org.springframework.cache.interceptor.KeyGenerator`接口，并使用该参数来指定。需要注意的是：**该参数与key是互斥的**
  - `cacheManager`：用于指定使用哪个缓存管理器，非必需。只有当有多个时才需要使用
  - `cacheResolver`：用于指定使用那个缓存解析器，非必需。需通过`org.springframework.cache.interceptor.CacheResolver`接口来实现自己的缓存解析器，并用该参数指定。

除了这里用到的两个注解之外，还有下面几个核心注解：

- `@CachePut`：配置于函数上，能够根据参数定义条件来进行缓存，它与`@Cacheable`不同的是，它每次都会真是调用函数，所以主要用于数据新增和修改操作上。它的参数与`@Cacheable`类似，具体功能可参考上面对`@Cacheable`参数的解析
- `@CacheEvict`：配置于函数上，通常用在删除方法上，用来从缓存中移除相应数据。除了同`@Cacheable`一样的参数之外，它还有下面两个参数：`allEntries`：非必需，默认为false。当为true时，会移除所有数据`beforeInvocation`：非必需，默认为false，会在调用方法之后移除数据。当为true时，会在调用方法之前移除数据。

### 缓存配置

在Spring Boot中通过`@EnableCaching`注解自动化配置合适的缓存管理器

除了按顺序侦测外，我们也可以通过配置属性`spring.cache.type`来强制指定。我们可以通过debug调试查看cacheManager对象的实例来判断当前使用了什么缓存

## Spring boot的工程结构

- root package结构：`com.example.myproject`
- 应用主类`Application.java`置于root package下，通常我们会在应用主类中做一些框架配置扫描等配置，我们放在root package下可以帮助程序减少手工配置来加载到我们希望被Spring加载的内容
- 实体（Entity）与数据访问层（Repository）置于`com.example.myproject.domain`包下
- 逻辑层（Service）置于`com.example.myproject.service`包下
- Web层（web）置于`com.example.myproject.web`包下

```txt
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- web
      |  +- CustomerController.java
      |
```


