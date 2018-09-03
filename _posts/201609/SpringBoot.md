---
title: Spring Boot(1):参见SpringBoot
date: 2016-09-22 20:22:09
tags: 
  - Spring
  - Spring Boot
---

最近开始看Spring Boot相关方面的文章，主要原因还是看网上说的这家伙开发软件的速度比传统的Spring的开发要快，所以在最近学习Spring 的同时也就顺便就开始看Spring Boot的书籍了。

当然书就看了两本，一本是《Spring Boot In Action》这本书怎么说呢，可以说是一个有实例的官方文档的简单版，其中提到的Grails还是有点意思的。开始的时候这本书还没有中文版的，看的是英文版，这个痛苦啊，那就不说了，看到一半的时候中文版发售了，那个心情就更不好了。另外一本是Apress的《Pro Spring Boot》，这本书讲的比较详细，对其中使用较多的模块还是都有所涉猎的，但是里面有一些技术现在我还是没有遇到过，所有看着还是有点麻烦。马上准备去看一件官方文档，我认为上面应该会讲一些其中内容是怎能实现的。

# 创建项目

创建一个Spring Boot项目有三种方式：

1. 通过官网的[Start Spring Boot](https://start.spring.io/)来进行创建，界面十分简洁，简单的配置一下就可以开始配置了开发自己的项目了

   ![SpringBoot](\images\201609\SpringBoot.png)

2. 使用Spring Boot CLI命令行工具来新建一个项目，简单命令为：

   ```shell
   spring init --build gradle demo
   ```

   这里是使用了Gradle为构建工具，而项目的名称则为demo，我没没有向其中添加其余的Spring 的模块，这就是最基本的Spring Boot的程序。

3. 使用IDE自带的工具来进行配置，其实就是书写一个简单的构建文件，构建文件要么是基于[Maven](http://maven.apache.org/)的，要么是基于[Gradle](https://gradle.org/)的，不过我是推荐使用Gradle作为构建工具，主要是文件的整体结构清晰简单，就是这点就是可以值得去使用的了。

发现了没有，通过以上的方式构建开始项目的文件的方式相比以前开始开发Spring的软件的方法简单了许多，所有在选择自己需要的Spring模块后，其会将这些模块放置到构建文件中，这时候只需要使用Maven和Gradle导入这些文件到IDE中，开始构建项目，其中需要的各种依赖就会自动的引入到系统中了，这就减少了在开发系统的时候需要引入各种jar包，并且还减少了项目中各种依赖关系复杂而到中引入jar包错误的问题了。

# 项目结构

```txt
.
├── .gitignore
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    ├── main
    │   ├── java
    │   │   └── top
    │   │       └── wangjinpeng
    │   │           └── DemoApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── top
                └── wangjinpeng
                    └── DemoApplicationTests.java
```

这是使用[Spring Boot start](https://start.spring.io/)页面上构建的项目页面构建的一个项目，使用了Gradle作为了构建工具，所以其直接就使用的Gradle Wrapper模块，这可以是在系统中没有Gradle的时候还能使用Gradle构建自己需要的文件项目。

剩下就十分的简单了，有一个默认的程序入口，就是DemoApplication.java，因为项目的名称为Demo，所以名称如上，其默认的命名规则为`项目名Application.java`。当然也是可以根据自己需要修改的。

在资源文件中一个配置文件`application.properties`，这个是默认的配置文件。当然，对于喜欢yaml的人，也可创建名为`application.yml`的文件来作为默认的配置文件。

剩下的一个文件为测试文件，这是默认提供的一个测试文件，这个不是程序必须的，所有可以去了也不会有什么问题。但是在测试驱动开发以及对于系统的正确性的要求的前提下，是必须编写测试的，编写测试可为将来的整个开发提供较好的便捷方式，以防止不小心修改代码后没有发现从而造成一个Bug。

# 默认依赖包

 ![依赖文件](\images\201609\依赖文件.png)

这是默认情况提供给我们的依赖包，我们可以发现其提供给我们了三种测试工具，Junit、Mock和hamcrest，还提供了默认的JSON处理包和yaml的处理包，提供了日志文件接口sef4j。

对于Spring的依赖包来说，除了包含Spring的核心容器aop、beans、core、expression外，还给了Spring Boot实现的源泉Spring Boot的依赖文件主要为和兴文件，日志处理，自动配置一加测试。我们最关注的就是Spring Boot的自动配置，这以后再说。

# Gradle文件

Gradle文件

```groovy
buildscript {
	ext {
		springBootVersion = '1.4.1.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'spring-boot'

jar {
	baseName = 'demo'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

从整体上这与Maven的构建文件pom.xml还是有点相似的，都大体上分为三个部分：一个是使用的插件、一个是项目的基本信息，最后一大块则是依赖的各种文件包。

不过对于整个build.gradle文件来说，我们只需要注意三点：

1. Spring Boot的版本是自己选定的，当发现官网的构建页面不能选择自己需要的版本的时候，可以自己修改构建文件中的springBootVersion后的内容。当然这里面的版本号当然也是Spring Boot这个项目曾经出现过的，不然在maven中心仓库中是没有的，会直接报错。当然需要升级的时候也是可以直接修改这个标识，然后从新构建整个项目就可以了。
2. 依赖的文件包可以直接写在dependencies中。不过这里的书写方式还是与以前的有一些区别，这里的依赖文件没有版本号。这样的好处是我们不需要关系我们所使用的依赖是哪一个版本的，每一从新构建的时候都会将相关的文件升级到最新的版本，我们不需要处理相关内容。这也是会带来许多的问题，就是需要一些老的包来进行构建，这时候我们可将这些独立的包的内容直接书写，插件会解决大部分的问题。
3. 如果是直接使用的Spring的组件，则可以没有必要书写使用这个模块的版本号，而对于其余的依赖，还是需要写上版本号，以避免重新构建是整个系统的依赖关发生错误从而导致需要重新编写整个项目的事情。

# 源文件

DemoApplication.java:

```java
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

DemoApplicationTests.java：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

	@Test
	public void contextLoads() {
	}

}
```

看过这两个文件之后我们会发现，在application.properties文件为空的情况下还是可以进行运行部署和对系统的测试，是什么样的技术实现的呢？其实就是两个注解：

1. @SpringBootApplication
2. @SpringBootTest

一个表示了这是一个Spring Boot的入口文件，表明这是一个Spring Boot程序。另一个则表示这个是一个Spring Boot的测试文件。对于这两个注解的特殊用处，我明天会继续书写。