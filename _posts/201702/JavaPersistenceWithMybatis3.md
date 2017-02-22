---
title: Java Persistence with Mybatis3
date: 2017-02-22 21:11:18
tags:
  - MyBatis
  - 笔记
categories:
  - 笔记
---

# Mybatis的配置

## 通过xml方式配置

### 环境（Environment）

MyBatis支持配置多数据源环境：配置多个数据源，然后在SqlSessionFactory中给出不同的环境。对于每一个环境，我们需要配置dataSource和transcationManager元素

 ### 数据源（DataSource）

DataSource的类型可以是UNPOOLED，POOLED，JNDI：

1. UNPOOLED：Mybatis将会在每次数据库操作时打开和关闭连接
2. POOLED：Mybatis将会使用数据库连接池来进行每一次数据库操作
3. JNDI：Mybatis将会通过JNDI数据源进行连接到每一个配置的数据库服务器

### TransctionManager

Mybatis支持两种类型的事务管理：JDBC和MANAGED

1. JDBC事务管理被用来管理应用的声明周期，如commit、rollback等。
2. MANADGED事务管理器被用来管理应用服务器连接的生命周期。

### Properties

属性配置元素可用于外部化配置值添加到属性文件中，并将属性的键名称用作占位符。

当在properties中配置的时候，会覆盖注入的数据。

### typeAliases

在SQL Mapper配置文件中，我们需要哎`resultType`和`parameterType`中给出JavaBean的完全限定类名。

也可以在JavaBean中使用注解`@Alias`，这会覆盖`<typeAliases`的配置

### Setttings

默认的MyBatis全局配置可以根据应用程序的特别需求进行覆盖

```xml
<settings>
	<setting name="cacheEnabled" value="true"/>
	<setting name="lazyLoadingEnabled" value="true"/>
	<setting name="multipleResultSetsEnabled" value="true"/>
	<setting name="useColumnLabel" value="true"/>
	<setting name="useGeneratedKeys" value="false"/>
	<setting name="autoMappingBehavior" value="PARTIAL"/>
	<setting name="defaultExecutorType" value="SIMPLE"/>
	<setting name="defaultStatementTimeout" value="25000"/>
	<setting name="safeRowBoundsEnabled" value="false"/>
	<setting name="mapUnderscoreToCamelCase" value="false"/>
	<setting name="localCacheScope" value="SESSION"/>
	<setting name="jdbcTypeForNull" value="OTHER"/>
	<setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```



### TypeHandlers

MyBatis通过实现抽象的JDBC来简化了持久层逻辑的实现。MyBatis在底层提供了更加简单的方法来实现数据库操作。

MyBatis自带了所有基本类型的基本类型包装类型，所有MyBatis找到其中的类型属性，它使用相应类型的类型处理程序来设置PreparedStatement的值，并将其填充到JavaBeans SQL的结果集中。

MyBatis提供了一个抽象类`BaseTypeHandler<T>`来创建自定义的类型处理程序

### Mappers

Mapper XML文件将通过应用程序的`statement id`来执行映射的SQL文件。我们需要在`mybatis-config.xml`文件中配置本地的 SQL Mappper文件。

```xml
<mappers>
	<mapper resource="com/mybatis3/mappers/StudentMapper.xml"/>
	<mapper url="file:///D:/mybatisdemo/app/mappers/TutorMapper.xml"/>
	<mapper class="com.mybatis3.mappers.TutorMapper"/>
	<package name="com.mybatis3.mappers"/>
</mappers>
```

每一个`<mapper>`标签可以通过不同的方式读取资源

## 使用Java API来配置MyBatis

### Environment

我们需要对每一个数据库创建一个`Environment`对象来通过MyBatis进行连接。对于多数据源的环境，我们需需要对每个环境配置`SqlSessionFactory`对象。

### DataSource

基本和xml配置的DataSource类似

### TransactionFactory

MyBatis支持两种`TranscationFactory`的实现：

* JdbcTransactionFactory
* ManagedTransactionFactory

如果应用程序在非托管环境中运行，则应使用JdbcTransactionFactory。

如果应用程序正在受管环境中运行并使用容器支持的事务管理服务，你应该使用ManagedTransactionFactory

### typeAliases

MyBatis提供了几种在`Configuration`对象中注册Type Aliases的方法：

* 通过一个类的类名来注册类的别名，不合规则的类名通过默认的别名规则
* 通过给定一个单类的别名进行注册一个类的别名
* 通过给一个全限定类名来注册一个单类别名
* 通过给定一个包名来注册包下所有类的别名
* 通过在包中继承`Identifiable`类型来注册一个类的别名

### typeHandlers

MyBatis提供了几种不同的方法在`Configuration`对象中注册类型处理器：

* 通过特别的类来注册类型管理器
* 直接注册类型管理器
* 通过给定报名来注册包下的所有类型处理器

### Settings

MyBatis提供了一组适合大多数应用程序的默认全局设置。当然我们也可进行更加适合的设置

```java
configuration.setCacheEnabled(true);
configuration.setLazyLoadingEnabled(false);
configuration.setMultipleResultSetsEnabled(true);
configuration.setUseColumnLabel(true);
configuration.setUseGeneratedKeys(false);
configuration.setAutoMappingBehavior(AutoMappingBehavior.PARTIAL);
configuration.setDefaultExecutorType(ExecutorType.SIMPLE);
configuration.setDefaultStatementTimeout(25);
configuration.setSafeRowBoundsEnabled(false);
configuration.setMapUnderscoreToCamelCase(false);
configuration.setLocalCacheScope(LocalCacheScope.SESSION);
configuration.setAggressiveLazyLoading(true);
configuration.setJdbcTypeForNull(JdbcType.OTHER);
Set<String> lazyLoadTriggerMethods = new HashSet<String>();
lazyLoadTriggerMethods.add("equals");
lazyLoadTriggerMethods.add("clone");
lazyLoadTriggerMethods.add("hashCode");
lazyLoadTriggerMethods.add("toString");
configuration.setLazyLoadTriggerMethods(lazyLoadTriggerMethods );
```

### Mappers

MyBatis提供了几种不同的方法在`Configuration`对象中注册Mapper XML文件和Mapper接口：

* 增加一个Mapper接口类
* 通过给定Mapper XML文件或者接口的包名来注册其下的所有Mapper
* 将所有的Mapper接口继承于一个`BaseMapper`接口

### logging

支持：

* SLF4J
* Apache Commons Logging
* Log4j 2
* Log4j
* JDK logging

当应用程序运行在过个日志库的环境下，并且需要使用特别的日志实现，可以调用以下方法之一

```java
org.apache.ibatis.logging.LogFactory.useSlf4jLogging();
org.apache.ibatis.logging.LogFactory.useLog4JLogging();
org.apache.ibatis.logging.LogFactory.useLog4J2Logging();
org.apache.ibatis.logging.LogFactory.useJdkLogging();
org.apache.ibatis.logging.LogFactory.useCommonsLogging();
org.apache.ibatis.logging.LogFactory.useStdOutLogging();
```

# 使用XML配置SQL Mapper

MyBatis提供了一个通过Mapper接口创建映射语句的好方法。一旦我们通过Mapper XML配置映射语句，我们可以创建一个与全限定名称相同的Mapper接口并添加具有匹配语ID的方法签名，输入参数和返回类型。

在XML映射文件中的命名空间是Mapper的全限定类名，而id，parameterType和returnType则和Mapper类的方法名称，参数类型和返回类型相同。

## 映射语句

MyBatis提供了各种不同元素来配置不同类型语句，例如SELECT，INSERT，UPDATE和DELETE。

对于像数据库中进行插入的语句来说，我们可以用useGeneratedKeys和keyProperty属性让数据库生成auto_increment列值并将生成的值设置为其中的一个输入对象属性。

对于SELECT返回的数据，我们可以使用集合来进行管理：

1. 对于List， Collection，或者Iterable类型，返回值是java.util.ArrayList
2. 对于Map类型，返回值是java.util.HashMap
3. 对于Set类型，返回值是java.util.HashSet
4. 对于SortedSet，返回值为java.util.TreeSet

## ResultMaps

我们可以定义ResultMaps并引用此resultMap查询几个SELECT语句。

当resultMap属性配置为`<select>`语句，MyBatis使用属性列映射以便填充JavaBeans属性。

## 一对一映射

对于一对一的映射，有三种方式可以进行处理：

1. 普通方式
2. 使用ResultMap：通过在xml文件的ResultMap中加入`<association>`元素来连接其他文件中的ResultMap文件
3. 使用Select：现在xml文件中加入其他select语句，然后在ResultMap中用`<association>`进行连接

## 一对多映射

我们可以将一对多类型的结果映射到使用对象的集合`<collection>`元素。当然也是可以使用ResultMap，也可以使用Select

## 动态SQL

MyBatis提供了对于动态SQL查询的一流支持，例如：`<if>`、`<choose>`、`<where>`、`<foreach>`和`<trim>`

`<if>`元素可用于有条件地嵌入SQL代码段。

MyBatis评估`<choose>`测试条件并使用带第一个子句条件，计算结果为TRUE。如果没有条件为真，则`<otherwise>`子句将被使用。

`<where>`元素只有在内部返回任何内容时才插入WHERE条件标签。此外，如果WHERE子句开始，它将删除AND或OR前缀与AND或OR.

`<trim>`元素的工作方式类似于`<where>`，但提供了额外的灵活性的去处前缀/后缀的需要。

`<set>`元素类似于`<where>`元素，任何内容都由内部条件返回并且将插入SET。

### 处理枚举类型

默认情况下，MyBatis使用EnumTypeHandler来处理枚举类型Java属性和存储枚举值的名称。您不需要任何额外的配置来做到这一点。

使用RowBounds来进行分页

使用ResultSet自定义使用ResultSetHandler

## 缓存

MyBatis默认使用一级缓存

当您添加`<cache />`元素时，将发生以下情况：

* 使用`<select>`语句查询出来的数据将被缓存
* 所有`<insert>`，`<update>`和`<delete>`操作查询出来的数据将会刷新缓存
* 缓存将会使用最近最少使用算法进行更新
* 缓存将不会对任何基于时间的计划刷新
* 缓存将会存储1024个对象或引用列表

# 使用注解配置Mapper

使用@Insert注解进行插入

使用@Options注解配置自动生成键

使用@Update注解配置更新语句

使用@Delete进行配置删除

使用@Select进行查询配置

使用@Results配置ResultMap

我们也可以在xml文件中配置好ResultMap，然后在Mapper文件中使用@ResultMap进行引入

动态SQL提供程序方法可以具有以下参数之一：

* 没有参数
* 单个参数具有相同类型的Mapper接口方法
* java.util.Map

然后使用@SelectProvider来调用这些方法

@InsertProvider

@UpdateProvider

@DeleteProvider

# Spring集成MyBatis

Spring可以通过以下配置要创建SQLSessionFactory对象：

* dataSource 
* typeAliases
* typeAliasesPackage
* typeHandlers
* typeHandlersPackage
* mapperLocations
* configLocation

当SqlSessionFactory配置完成后，我们需要配置SQLSessionTemplate，这是一个线程安全的对象，我们可以从中获取安全的SqlSession对象

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

可以使用基于XML配置文件的Spring Bean或者使用基本注解的配置的Spring Bean，将SqlSession bean注入到需要的bean中

`<mybatis：scan>`元素将在指定的搜索Mapper接口以逗号分隔的包名称列表，当然通过注解@MapperScan也是可以完成这样一个过程的，当然如果不使用的话就要使用XML文件为每一个Mapper进行相关的配置了

也可以在Spring中接合MyBatis来完成事务管理