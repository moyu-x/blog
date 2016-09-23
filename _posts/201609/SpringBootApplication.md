---
title: Spring Boot(2):SpringBootApplication注解
date: 2016-09-23 09:24:05
tags: 
  - Spring
  - Spring Boot
---

这篇文章就来简单的介绍出初始化创建Spring Boot程序的时候提供的两个类级别的注解，一个是用于程序如口程序的`@SpringBootAppliaction`注解和用于测试使用的`@SpringBootTest`注解 ，还有就是关注一下SpringRunner类。

# `@SpringBootAppliaction`注解

首先我们先来简单的看一下他的源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class))
public @interface SpringBootApplication {
  
	Class<?>[] exclude() default {};

	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

}
```

这份源代码中有三个比较重要的类级别的注解，一个是`@SpringBootConfiguration`，这个注解表明这个是一个Spring Boot程序。第二个是一个`@EnableAutoConfiguration`注解，表示这个Spring Boot的程序是使用自动配置的，如果需要配置一些其他的配置，则可以在application.properties或application.yml中进行配置，Spring Boot会自动加载这些配置文件；在类中的还可以引入其他配置文件的方式另说。第三个是`@ComponentScan`注解，表明在这个文件的同级别的包之下的所有文件的注解都是可以被发现的。

接口中定义的方法中，有两个方法是用来加载配置类的，这些配置类就是在application.properties中的内容，喜欢直接使用配置类的应该会喜欢这个方法的。剩下的两个方法则是对于扫描类包的了，默认情况下，只会扫描的同级别下所有的类包，但是如果有些类包不想被扫描到的话也可以使用这个方法制定扫描的类包。

# `@SpringBootTest`注解

源代码为：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(SpringBootTestContextBootstrapper.class)
public @interface SpringBootTest {
  // 详细内容请看源码
}
```

这里只有一个类级别的注解需要关注，这就是`@BootstrapWith`注解，这个注解是告诉测试程序测试的入口，一般情况下我们使用Junit进行测试的时候，总是自己加载各种依赖，而使用这个注解则告诉了测试程序配置加载的位置，当然一般是加载默认的配置文件，也可以加载自定义的文件。

而其中默认提供的`SpringBootTestContextBootstrapper`则是一个实现测试文件配置的加载，启动入口以及基本的配置，这我么可以从其继承链上看出来这是对`TestContextBootstrapper`的实现:

![SpringBootTestContextBootstrapper](\img\SpringBootTestContextBootstrapper.png)

# SpringRunner

对于测试文件，如果经常看网上教程的人会发现在测试文件使用的时候使用@RunWith注解会发现使用的不是SpringRuuner.class，而是使用SpringJUnit4ClassRunner.class，这是是不是会认为SpringRunner更加强大的呢。其实不是，SpringRunner集成了SpringJUnit4ClassRunner，其中就多了一个方法，就是：

```java
public SpringRunner(Class<?> clazz) throws InitializationError {
		super(clazz);
}
```

这其实是一个构造器，就是将标准的Junit测试文件加载到TestContextManager中，当然了，既然使用了@RunWith注解了，在这个时候其中的内容SpringRuuner和SpringJUnit4ClassRunner是没有多大区别的，但是在其他情况下还是有区别的。

# SpringApplication

这个类相当的有意思，在目前的情况下我们使用到的其实是他的一个静态类`run`，在源文件中，这个方法是一个重载方法，在以后我们还可以使用这个方法定义在系统加载的时候的参数，默认情况下，这些参数和application.propreties中的没有什么不同，但是我们可以更加自己的需要进行添加，让其加载个性化的参数。