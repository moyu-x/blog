title: Docker的使用
date: 2015-11-22 15:07:45
tags:
- docker
categories:
- 虚拟化
---

## 简介
在2013年还是2014年年初的时候，我当时在闲暇时刻去CSDN上面看资料时，发现了关于docker的介绍，后来就再也没有关心过这样一个技术。并且在当时还看到了对于go语言的简单介绍，说是这样一门语言是由Google开发的要用来代替C预言的，我当时还在我的计算机上搭建了一个Go语言的开发环境，并且还看了一些书籍，但是现在都完全的忘记了，只知道当的论坛中对这门语言的评价不是太好，那时还是1.3版本，编译器还是用C来写的。没有实现自举编译，现在1.5已经实现自举编译了，这不说了，因为现在我也不会这一门语言，除了一些基本的语法之外。虽然网络上已经有许多的安装方法了，但是我还是写一下作为分享好了。

## 网站
[Docker官网](www.docker.com)  
[The way to go中文版](https://github.com/Unknwon/the-way-to-go_ZH_CN)  
[Daocloud官网](www.daocloud.io):docker的一个镜像，挺好使得

## 安装docker
鉴于国内的网速，还说使用快一点的镜像比较好，如果是企业的话就不要考虑了，尽量使用商业的吧:  
Linux  
```bash
# daocloud的下载指令
# curl -sSL https://get.daocloud.io/docker | sh
# docker下载指令
# wget -qO- https://get.docker.com/ | sh
```
Windows和Mac上就直接使用官方的镜像了，不用考虑其他的了，因为迅雷下这个还是挺快的

## 安装daocloud的工具（不适用可直接跳过）
在Daocloud的Dashboard中的我的集群中根据步骤进行部署  
![Dashboard](http://7xokkh.com1.z0.glb.clouddn.com/dashboard.png)  
然后下载镜像的使用如下命令：
```bash
# dao pull youimages
```

## 直接下载命令
直接使用docker命令，如果使用了daocloud话会直接使用daocloud的的加速模式：
```bash
# docker pull yourimages
```

## 一个简单的docker实例，来自官方
输入如下指令；
```bash
# docker run hello-world
```
这时会下载一个hello-world的镜像，如果显示了如下的东西，则说明你已经成功安装Docker了：
```
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/

```
然后这时你就可以根据你手中的书籍去学习docker了。

## 后话
现在docker的中文书籍比较少，我认为还是使用英文的官方文档比较好，这是你需要加入一些社区去学习新的知识才能在以后的路上走的更加远。在和Docker同一时间出现的还有一个coreos的基于linux的操作系统，也是应用于虚拟化方面的，我曾经想下了试过，但是很可惜我没有成功，但这也是现在容器技术的另外的一种形式。