---
title: SpringAOP
date: 2016-06-03 22:37:44
tags:
- Spring
- 笔记
- java
categories:
- 笔记
---

# Spring AOP
AOP——Aspect Oriented Programing

## AOP术语
**连接点（Joinpoint）**：一个类或一段程序代码拥有一些具有边界性质的特定点，这些代码中的特定点就称为“连接点”。Spring仅支持方法的连接点。由用方法表示的程序执行点和用相对点表示的方位确定
**切点（Pointcut）**：连接点相当于数据库中的记录，而切点相当于查询条件。切点和连接点不是一对一的关系，一个切点可以匹配多个连接点，切点只定位到某个方法上，所以需定位到具体的连接点还需要提供方位信息。
**增强（Advice）**：增强是织入目标连接点上的一段代码。
**目标对象（Target）**：增强逻辑的织入目标类。
**引介（Introduction）**：一种特殊的增强，它为类添加了一些属性和方法
**织入（Weaving）**：是将增强添加大目标类具体连接点上的过程，方式：
1. 编译时织入，需要使用特殊的Java编译器
2. 类加载时织入，使用特殊的类加载器
3. 动态代理织入，在运行期为目标类添加增强生成子类的方式
spring采用动态代理织入，而AspectJ采用编译期织入和类加载期织入
**代理（Proxy）**
**切面（Aspect）**：切面有切点和增强（引介）组成，既包括横切逻辑定义，也包括连接点的定义，AOP的重点在于如何将增强应用于目标对象的连接点上，包括：如何通过切点和增强定位到连接点上，如何在增强中编写切面代码

## 创建增强类型
Spring支持的5种增强：
1. **前置增强**：
`public void before(Method method, Object[] args, Object target) throws Throwable `
method为目标类提供方法，args为目标类提供参数，target提供目标类实例，强制执行则为使用spring提供的代理工厂，设置目标代理，为代理目标添加增强。在spring的xml中的配置则为：创建bean；指定代理的接口，如果是多个接口，则使用<list>元素；指定使用的增强；指定对哪个bean进行代理
![ProxyFactoryBean几个常用属性][7]
2. **后置增强**
`public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable`
returnValue为目标实例返回的结果，method为目标类提供方法，args为目标类提供参数，target提供目标类实例
3. **环境增强**：
`public Object invoke(MethodInvocation invocation) throws Throwable`
MethodInvocation不但封装了目标方法及注入参数，还封装了目标方法所在的实例对象，通过MethodInvocation的getArgument()可以获取目标方法的参数数组，通过proceed()反射调用目标实例相应对象方法
4. **异常抛出增强**：最适合事务管理，当参与事务的某个Dao抛出异常时，事务管理器就必须回滚事务，ThrowAdvice异常抛出增强接口没有定义任何方法，是一个标示接口，在运行期间使用反射的机制自行判断，必须采用以下标签形式定义异常抛出的增强方法：
`public void afterThrowing(Method method, Object[] args, Object target,Throwable);`
前三个参数为可选参数，要么提供三个参数，要么不提供，而最后一个参数为Throwable或是其子类
在继承树上，两个类的距离越近，就说明两个类的相似度越高，目标抛出异常后，优先选取有异常参数和抛出异常相似度最高的方法
5. 引介增强

## 创建切面
spring支持两种方法匹配器：静态方法匹配和动态方法匹配器。静态方法匹配器仅对方法名（包括方法名和参数类型及顺序）进行匹配；而动态方法匹配器会在运行期检查方法参数的值

### 切点类型
1. 静态方法切点
2. 动态方法切点
3. 注解切点
4. 表达式切点
5. 流程切点
6. 复合切点

切面可以分为3类：一般切面（Advisor）、切点切面（PointcutAdvisor）和引介切面(IntroductionAdvisor)

静态普通方法名匹配切面：
1. 切点方法匹配规则:为所匹配的方法名
2. 切点类的匹配规则：为匹配的类或是其子类

静态方法匹配切面：
1. 向切面注入一个前置增强
2. 通过一个父<bean>定义公共的配置信息
3. 设置bean的代理

静态正则表达式方法匹配切面：
1. 用正则表达式定义目标类全限定方法名的匹配模式
2. 匹配模式串

## 自动创建代理
代理创建器的分类：
1. 基于bean配置名规则的自动代理创建器
2. 基于Advisor匹配机制的自动代理创建器
3. 基于bean中的AspectJ注解标签的自动代理创建

自动代理创建器实现类的类继承图：
![继承图][8]

BeanNameAutoProxyCreator有一个beanNames属性，允许用户指定一组需要自动代理的bean，bean的名称可以使用*通配符，一般情况下，不会为FactorBean的备案创建代理，如果需要，则在beanNames中指定添加&的bean名称

DefaultAdvisorAutoProxyCreator能够自动扫描容器中的Advisor并将Advisor自动织入匹配的目标bean中，即为匹配的目标bean自动创建代理

## 基于@AspectJ配置切面
spring支持9个@AspectJ切点表达式函数，他们用不同的方式描述目标类的连接点，根据描述对象的不同，可分为四类：
1. 方法切点函数
2. 方法参数切点函数
3. 目标类切点函数
4. 代理类切点函数

@AspectJ支持的3中通配符
1. `*`：匹配任意字符，但它只能匹配上下文中的一个元素
2. `..`：匹配任意字符，可以匹配上下文中的多个元素，在表示类时，必须和*联合使用，在表示参数时则单独使用
3. `+`：表示按类型匹配指定类的所以类，必须跟在类名的后面

@ApectJ的其他增强注解符：
![图1][9]
![图2][10]
![图3][11]

## 基于Schema配置切面
前置增强
1. 声明切点引用
2. 声明切点表达式
3. 调用增强的bean的方法

其余方式都是差不多

引介增强：
1. 要引介实现的接口
2. 默认的实现类
3. 哪些类要引介接口实现

基于Schema和@AspectJ绑定连接点想信息：
1. 所以增强类型对应方法的第一个参数都可以声明为JionPoint（环绕增强可以声明为ProceedingJoinPoint）访问连接点信息
2. <aop:after-returning>可以通过returning属性绑定连接点方法的返回值，<aop:after-throwing>可以通过throwing属性绑定连接点方法
3. 所有增强类型都可以通过可绑定参数的切点函数绑定连接点方法的参数

## 比较
![比较][12]



[8]: http://7xokkh.com1.z0.glb.clouddn.com/2016-06-01_154514.png
[9]: http://7xokkh.com1.z0.glb.clouddn.com/2016-06-01_175822.png
[10]: http://7xokkh.com1.z0.glb.clouddn.com/2016-06-01_175847.png
[11]: http://7xokkh.com1.z0.glb.clouddn.com/2016-06-01_175904.png
[12]: http://7xokkh.com1.z0.glb.clouddn.com/2016-06-02_112318.png
