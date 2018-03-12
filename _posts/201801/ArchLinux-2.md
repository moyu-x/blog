---
title: ArchLinux的使用(2):开发环境的搭建
date: 2018-01-07 20:43:05
tags:
  - ArchLinux
categories:
  - ArchLinux
---

对于我用过的几个发行版本来说，ArchLinux算是天生对程序员亲和的，主要是有这几个原因，首先是官方源中维护了许多的编程环境的包，尤其是以Python维护的最多。其次是有AUR源，有许多人在共同的维护这个源，可以让开箱即用的包越来越多。最后的原因才是他是一个Linux的发行版本。

## 安装之前

在配置安装环境之前，我们得先对我们镜像源改造一下，这样才能继续我们之后的工作。我们得启用用`multilib`和`archlinuxcn`两个源。

### multilib

对于`multilib`这个源，我们只需要简单的将`/etc/pacman.conf`中的`multilib`的注释取消了就行

### arclinuxcn源

对于这个我们可以在`/etc/pacman.conf`加入如下配置：

```bash
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

然后再安装`archlinuxcn-keyring`这个包导入秘钥就行。对于这其中可能存在的两个问题，也就是由使用硬件时钟造成秘钥导入不成功的问题（这个一般是时间的问题，硬件时间不是UTC时间的情况下造成的）
，可以使用以下方法进行解决：

1. 立即同步时间，不用修改系统的时间设置
2. 删除`/etc/pacman.d/gnupg`文件夹，然后运行`pacman-key --init`，`pacman-key --poplaute archlinux`和`pacman-key --refresh-keys`就可以解决这一个问题

## Python

对于Python编程环境来说，ArchLinux默认的Python环境是最新版本的Python3版本，所以在使用的时候需要注意这个问题，对于常见Python环境的安装方法如下：

```bash
# 安装Anaconda
pacman -S anaconda

# 安装pyenv
pacman -S pyenv

# 安装Pytcharm
yaourt -S pycharm-professional
```

## Java

 在ArchLinux中使用Java，可以选择两种JDK的版本，一种是使用openjdk，另外一个是使用Oracle jdk版本，并且在ArchLinux中，可以使用`archlinux-java`来切换不同的版本。对于Java环境的一些工具集，可以使用如下的命令进行安装。

```bash
# 安装oracle jdk
pacman -S jdk

# 安装openjdk
pacman -S jdk9-openjdk

# 安装maven
pacman -S maven

# 安装gradle
pacman -S gradle

# 安装eclipse
pacman -S eclipse

# 安装Idea
pacman -S intellij-idea-ultimate-edition
```

## Node环境

和大部分的平台的安装配置一样，就是有可能需要配置以下全局的`npm`包的安装位置和一些目录，可以使用`pacman -S nodejs npm`来进行安装，可以在`~/.npmrc`中写入如下配置：

```bash
# $HOME为你home目录路径的全写
cache=$HOME/.node_modules
prefix=$HOME/.node_modules
```
对于其他的一些编程环境来说，基本上也是大同小异。对于IDE的配置和选择我会在以后的博文中写出给大家作为参考。
