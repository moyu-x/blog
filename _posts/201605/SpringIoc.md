---
title: SpringIoc
date: 2016-05-29 22:11:56
tags:
- Spring
- 笔记
- java
categories:
- 笔记
---
BeanFactory是Spring框架的基础设施，面向Spring本身；而ApplicationContext是面向Spring框架的开发者。所有可以被spring容器实例化并管理的java类都可以成为bean。

## 介绍
先在spring的xml中配置好bean，然后使用。

### ApplicationContext
使用方式：
```Java
// 使用类路径
ApplicationContext context = new ClassPathXmlApplicationContext("xml文件路径");

// 使用系统路径
ApplicationContext context = new FileSystemXmlApplicationContext("xml文件路径");

// 指定一组配置文件
ApplicationContext context = new ClassPathXmlApplicationContext(new String["xml的文件路径"]);

// 带注解的java类提供配置信息
ApplicationContext contex = new AnnotationConfigApplicationContext(Bean类的class);
```
使用注解时需要**cglib**框架
BeanFactory在初始化时并未实例化Bean，直到第一次访问某个Bean时才实例化目标Bean；而ApplicationContext在初始化应用上下文时就实例化所有单实例的Bean。
从WebApplicationContext中获取ServletContext的引用，整个Web应用上下文对象将作为属性放置到ServletContext中，以便Web应用环境可以访问Spring应用上下文。
Spring中提供的用于启动WebApplicationContext的Servlet和监听器为：  
1. org.springframework.web.context.ContextLoaderServlet
2. org.springframework.web.context.ContextLoaderListene

### WebApplicationContext
1. 指定配置文件
2. 申明web容器监听器
3. 指定Log4j配置文件位置（因为WebApplicationContext需要使用日志功能，也可以使用其他的日志框架）
4. 装载Log4j配置文件的自启动Servlet

对于低版本的web来说，则执行：
1. 声明自启动的Servlet
2. 启动顺序

注解方式实现：
1. 通过指定Context参数，让spring使用AnnotationConfigWebApplicationContext而非XmlWebApplicationContext启动容器
2. 指定标注了`@Configuration`的配置类，可以使用逗号或空格分隔
3. ContextLoaderListener监听器将根据上面配置使用AnnotationConfigWebApplicationContext，根据ContextConfiguration指定的配置类启动容器

### 资源加载
Spring支持的资源类型前缀：
![Spring支持的资源类型前缀][1]

## Bean装配
### 基本
bean装配的条件：
1. Spring框架的类包4都已经放到应用程序的类路径之下
2. 应用程序为Spring提供完备的Bean配置信息
3. Bean的类都已经放置到应用程序的类路劲之下

Bean的配置信息：
1. Bean的实现类
2. Bean的属性信息
3. Bean的依赖关系
4. Bean的行为配置

id在Ioc中命名是唯一的，此外id的命名需要满足XML对id的命名规范，在存在多个匿名的bean时，则在类路径之后添加`#编号`进行调用。

### 依赖注入
Spring中配置文件采用和元素标签顺序无关的策略。Spring容器能够顺利实例化以构造参数注入方式配置bean有一个前提：Bean构造函数注入引用的对象必须已经准备完毕。
#### 属性注入
通过set方法注入bean的属性值或者是依赖对象。属性注入要求Bean提供一个默认的构造函数，并且需要注入提供属性对应的Setter方法。Spring先调用Bean的默认构造函数实例化Bean对象，然后通过反射的方式调用Setter注入属性值。Spring只会检查Bean中是否有对应的setter方法，至于bean中是否拥有实现类中拥有相对应的属性变量则不作要求。变量的前两个字母要么全大写，要么全部小写。使用属性注入方式只能人为在配置时提供保证，而无法在语法级提供保证

#### 构造器注入
方式：
1. 按类型匹配注入
```xml
<bean id="test" class="top.wangjinpeng.test">
    <constructor-arg type="java.lang.String">
        测试
    </constructor-arg>
</bean>
```
2. 按索引位置注入
```xml
<bean id="test" class="top.wangjinpeng.test">
    <constructor-arg index="0">
        测试
    </constructor-arg>
</bean>
```
3. 联合使用类型和索引注入
```xml
<bean id="test" class="top.wangjinpeng.test">
    <constructor-arg type="java.lang.String" index="0">
        测试
    </constructor-arg>
</bean>
```
4. 通过自身的反射匹配注入
```java
public Test(String test, Car car){
    this.test = test;
    this.car  = car;
}
```
```xml
<bean id="test" class="top.wangjinpeng.test">
    <constructor-arg type="java.lang.String" index="0">
        测试
    </constructor-arg>
    <constructor-arg>
        <ref bean="car"/>
    </constructor-arg>
</bean>
```
#### 注入参数
##### 字面值
以`<!CDATA[]>`为例
```xml
<bean id="test" class="top.wangjinpeng.test">
   <property name="test">
        <!CDATA[测试]]>
    </property>
</bean>
```
##### 引用其他bean
```xml
<bean id="test" class="top.wangjinpeng.test">
   <property name="test">
        <ref bean="car"/>
    </property>
</bean>
```
`<ref>`元素可以通过以下3种属性引用容器内的其他bean
1. bean：通过该属性可以引用同一容器中的其他bean
2. local：通过该属性只能引用同一配置文件中定义的Bean
3. parent：引用父容器中的bean

##### 集合类型
可以使用如下几种集合类型：Lsit、Set、Map、Properties
以List为例：
```xml
<bean id="test" class="top.wangjinpeng.test">
   <property name="test">
        <list>
            <value>测试1</value>
            <value>测试2</value>
        </list>
    </property>
</bean>
```

##### 通过util命名空间配置集合类型Bean
命名空间为(只是一个例子）：
```xml
xmls:util="http://www.springframwork.org/schema/util
xsi:schemaLocation="http://www.springframwork.org/schema/util
                   "http://www.springframwork.org/util/spring-util-3-1.xsd"
```
使用方法为(以map为例）：
```xml
<util:map id="test">
    <entty key="A" value="测试1"/>
    <entty key="B" value="测试2"/>
<util:map>
```

##### 简化配置方式
使用P命名空间，对于字面值，其格式为：`P:<属性名>="xxx"`，对于引用对象属性，其格式为：`P:<属性名>_ref=""`

##### 自动装配
<bean>元素提供了一个指定自动装配类型的属性:`autowire="<自定装配类型>"`
![自动装配类型][2]

#### Bean的作用域
bean的作用域类型的配置方式为：`scope="<作用域类型>"`
![Bean的作用域类型][3]

### 基于注解的配置
`@Component`注解可以被Spring容器识别，且自动将POJO装换为容器管理的Bean
`@Repository`：用于对DAO实现类进行标注
`@Service`：用于对Service的实现类进行标注
`@Controller`：用于对Controller实现类进行标注

使用注解配置信息启动Spring容器
1. 声明context命名空间
2. 扫描类包以应用注解定义的Bean

如果仅希望扫描特定类而非基类包下的所以类，可以使用`resource-pattern`属性过滤出特定的类
<context:include-filter>表示要包含的目标类，<context:exclude-filter>表示要排除在外的目标类
![过滤表达式][4]

自动装配Bean
spring通过`@Autowired`注解实现Bean的依赖注入，`@Autowired`默认按类型匹配的方式在容器中查找Bean
1. 使用`@Autowired`的required属性——希望spring在找不到Bean时也不会抛出异常，则@`Autowired（required=false）`
2. 使用@Qualifier指定注入Bean的名称
3. 对类方法进行标注——如果一个方法拥有多个参数，默认自动选择匹配类型的Bean进行注入，但spring允许对方法参数标注`@Qualifier`以指定注入Bean的名称
4. 对集合类进行标注

### 基于Java类的配置
使用java类提供的bean定义信息：
普通的POJO只要标注@Configuration注解，每个标注了@Bean的类方法都相当于提供了一个Bean的定义信息
1. 将一个POJO标注为定义Bean的配置类
2. 将一个POJO标注为定义为Bean
Bean的类型由方法返回值类型决定，名称默认和方法名相同，也可以显示指定Bean的名称`@Bean(name="bean的名称")`

任何标注来了@Configuration的类，本身也相当于标注了@Component，既可以像普通的bean一样被注入到其他bean中，可以使用@Scope标注以控制Bean的作用范围，由于会通过反射在运行时调用类，所以使用基于java类的配置必须保证将spring aop类包和CGLIB类包加载到类路径之下

直接通过@Configuration类启动Spring
spring提供了一个AnnotationConfigApplicationContext类，可以直接通过标注@Configuration的java类启动spring容器，且还支持通过编码的方式加载多个@Configuration配置了，语法为
```javva
ApplicationContext contex = new AnnotationConfigApplicationContext();
context.register(类1.class);
context.register(类2.class)
context.refresh();
```
也可通过@Import将多个配置类组装到一个配置类，也可通过xml文件引用@Configuration的配置类：`<context:componet-scan base-package="包位置" resource-pattern="类.class"/>`
，并且@Configuration配置类引用xml中的配置，添加：`@ImportResources("xml文件位置")`

## Bean不同配置方式比较
![Bean不同配置方式比较][5]

## bean不同配置方式适用场合
![bean不同配置方式适用场合][6]


  [1]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-28_210320.png
  [2]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-29_162919.png
  [3]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-29_163409.png
  [4]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-29_164518.png
  [5]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-29_211554.png
  [6]: http://7xokkh.com1.z0.glb.clouddn.com/2016-05-29_211933.png
