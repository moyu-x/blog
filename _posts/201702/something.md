---
title: 编译和配置
date: 2017-02-05 22:33:44
tags:
  - 配置
  - 编译
categories:
  - 配置
  - 编译
---

# 前言

记录我编译一些软件的方法和经常使用到的工程配置。

# 编译Git

首先得到Github上面下载最新的git源代码，下面是解压后的编译命令过程：

```bash
# 安装编译环境
sudo apt install build-essential libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext

# 编译源代码
make prefix=/usr/local all
sudo make prefix=/usr/local install
```

# 编译Mercurial

下载最新版本的代码后执执行如下代码：

```bash
# 安装基本环境
sudo apt install python-dev python-docutils

# 编译源代码
make all
sudo make install
```

# 在maven中指定jdk的版本

### 全局配置

在settings.xml中进行配置

```xml
<profile>  
    <id>jdk18</id>  
    <activation>  
        <activeByDefault>true</activeByDefault>  
        <jdk>1.8</jdk>  
    </activation>  
    <properties>  
        <maven.compiler.source>1.8</maven.compiler.source>  
        <maven.compiler.target>1.8</maven.compiler.target>  
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
    </properties>   
</profile>  
```

### 局部配置

在每个项目的pom.xml文件中进行配置：

```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>  
            <configuration>  
                <source>1.8</source>  
                <target>1.8</target>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>  
```



# 在maven中使用阿里镜像

配置setting.xml文件:

```xml
<mirrors>
  <mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
</mirrors>
```



# 在gradle中使用阿里镜像

在每个工程的文件build.gradle中配置：

```groovy
maven { url 'http://maven.aliyun.com/nexus/content/groups/public/'}
```



