---
title: Spring实战(1)：基础及配置
date: 2016-12-05 23:02:49
tags:
  - 笔记
  - Spring实战
categories:
  - Spring
---

# 第一章

## 基础

降低Java开发的复杂性，Spring采取的策略：

* 基于POJO的轻量级和最小侵入性编程
* 通过依赖注入和面向接口实现松耦合
* 基于切面和惯例进行声明式编程
* 通过切面和模板减少样板式代码

耦合具有两面性：

1. 紧密的耦合代码难以测试、难以复用、难以理解，并且典型地表兄出“打地鼠”式的bug特性
2. 一定程度的耦合又是必须的——完全没有耦合的代码什么也做不了

### DI

Spring通过Application Context装载bean的定义并把它们组装起来。Spring应用上下文全权负责对象的创建和组装。

### AOP

AOP往往被定义为促使软件系统实现关注点的分离的一项技术。AOP能够使这些服务区模块化，并以声明 的方式将他们的应用到他们需要影响的组件中去。所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来的复杂性。

`<aop:aspect>`元素引用该bean，前置调用为`<aop:before>`，后置调用为`<aop:after>`

## Spring容器

spring自带多个容器的实现，可以分为两种不同的类型bean工厂（由org.springframework. beans.
factory.eanFactory接口定义）是最简单的容器，提供基本的DI支持。应用上下文
（由org.springframework.context.ApplicationContext接口定义）基于BeanFactory构建，并提供应用框架级别的服务

Spring自带了多种类型的应用上下文。下面罗列的几个是你最有可能


* AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。
* AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文。
* ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
* FileSystemXmlapplicationcontext：从文件系统下的一个或多个XML配置文件中加载上下文定义。
* XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。

Spring Bean的生命周期：

![SpringBean的生命周期](\Image\SpringBean的生命周期.png)

![Spring模块](\Image\Spring模块.png)



# 第二章

Spring提供了三种主要的装配机制：

* 在XML中进行显式配置。
* 在Java中进行显式配置。
* 隐式的bean发现机制和自动装配

## 自动化配置

Spring从两个角度来实现自动化装配：

* 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
* 自动装配（autowiring）：Spring自动满足bean之间的依赖

使用了`@Component`注解。这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean

`@ComponentScan`注解启用了组件扫描，这个注解能够在Spring中启用组件扫描,默认会扫描与配置类相同的包

使用XML来启用组件扫描的话，那么可以使用Spring context命名空间的`<context:component-scan>`元素

Spring应用上下文中所有的bean都会给定一个ID，Spring会根据类名为其指定一个ID。具体来讲就是将类名的第一个字母变为小写。如果想为这个bean设置不同的ID，你所要做的就是将期望的ID作为值传递给@Component注解。

Spring支持将`@Named`作为`@Component`注解的替代方案。两者之间有一些细微的差异，但是在大多数场景中，它们是可以互相替换的。

有一个原因会促使我们明确地设置基础包，那就是我们想要将配置类放在单独的包中，使其与其他的应用代码区分开来，为了指定不同的基础包，需要做的就是在`@ComponentScan`的value属性中指明包的名称.

除了将包设置为简单的String类型之外，`@ComponentScan`还提供了另外一种方法，那就是将其指定为包中所包含的类或接口

为了声明要进行自动装配，可以借助Spring的`@Autowired`注解。如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出一个异常。为了避免异常的出现，你可以将`@Autowired`的required属性设置为`false`

将required属性设置为false时，Spring会尝试执行自动装配，但是如果没有匹配的bean的话，Spring将会让这个bean处于未装配的状态。但是，把required属性设置为false时，你需要谨慎对待。如果在你的代码中没有进null检查的话，这个处于未装配状态的属性有可能会出现NullPointerException

`@Autowired`是Spring的特有注解，可以替换为`@Inject`

## 通过java代码转配bean

JavaConfig是配置代码，这意味着它不应该包含任何业务逻辑，JavaConfig也不应该侵入到业务逻辑代码之中。尽管不是必须的，但通常会将JavaConfig放到单独的包中，使它与其他的应用程序逻辑分离开来，这样对于它的意图就不会产生困惑了。

创建JavaConfig类的关键在于为其添加`@Configuration`注解，`@Configuration`注解表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节

要在JavaConfig中声明bean，我们需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加`@Bean`注解.默认情况下，bean的ID与带有@Bean注解的方法名是一样的，但是可以使用name属性指定一个不同的名字。

在JavaConfig中装配bean的最简单方式就是引用创建bean的方法。

但是，在软件领域中，我们完全可以将同一个实例注入到任意数量的其他bean之中。默认情况下，Spring中的bean都是单例的。

**带有@Bean注解的方法可以采用任何必要的Java功能来产生bean实例**

## 通过XML装配bean

最简单的Spring XML配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <!--其余代码-->
  
</beans>
```

要在基于XML的Spring配置中声明一个bean，我们要使用spring-beans模式中的另外一个素：`<bean>`。`<bean>`元素类似于JavaConfig中的`@Bean`注解

作为替代的方案，你也可以使用Spring的c-命名空间。c-命名空间是在Spring 3.0中引入的，它是在XML中更为简洁地描述构造器参数的方式。要使用它的话，必须要在XML的顶部声明其模式

XML不允许某个元素的多个属性具有相同的名字。

`<list>`元素是`<constructor-arg>`的子元素，这表明一个包含值的列表将会传递到构造器中。其中，`<value>`元素用来指定列表中的每个元素。可以使用`<ref>`元素替代`<value>`，实现bean引用列表的装配。

Spring为`<constructor-arg>`元素提供了c-命名空间作为替代方案，与之类似，Spring提供了更加简洁的p-命名空间，作为`<property>`元素的替代方案

util-命名空间所提供的功能之一就是`<util:list>`元素，它会创建一个列表的bean。![Spring util-命名空间中的元素](\Image\Spring util-命名空间中的元素.png)

在使用了XML的配置之后，如果要让spring同时加载它和其他基于Java的配置可以使用`@ImportResource`注解

在XML中，我们可以使用import元素来拆分XML配置

Spring中装配bean的三种主要方式：自动化配置、基于Java的显式配置以及基于XML的显式配置

