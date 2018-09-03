---
title: 在Windows上玩Linux
date: 2017-03-27 09:25:30
tags:
  - Linux
  - Windows
categories:
  - Linux
---

最近几天在Windows上折腾Linux了，主要是发现现在我装双系统没有意义，准备换回以前使用Linux的模式了，主要也是用来进行测试和一些Linux的学习，所有来回切换系统十分的麻烦。当然了我也体验了一些新的玩法，所以在这里总结一下，在Windows上玩Linux主要有这么几种：

1. 虚拟机
2. Docker
3. Bash on Windows
4. Cygwin

前两种折腾的方式我就不再说了，应该是现在的主流玩法了，所以我还是介绍一下后面两种玩法吧O(∩_∩)O哈哈~

# Bash on Windows

Windows的Linux子系统其实一直存在，Windows xp和Windows 7中都可以使用，在Windows 8中被微软删除了，以前这个子系统是微软为美国军方提供的，所以用的比较少，功能也少，其也不是为了开发者进行设计的，所以社区对其功能的挖掘较少。在Windows 10发布一周年更新的时候，微软联合了Canonical 联合发布了Bash on Linux这个子系统，其完全使用净室开发，在Windows上完全实现了Bash的接口和操作，其实现的机构图如下，其实就是利用bash对Windows底层的内核态进行了一个抽象和，其基本和win32 api这些是位于同一个层次的：

![BashOnWindows结构](/images/201703/BashOnWindows结构.jpg)

使用的方法也十分的简单，在`设置 -> 更新 -> 针对开发人员`中设置成开发人员模式就行，然后在`启用或关闭Windows功能`中将Linux子系统打开基本就可以了。在重启后，在cmd或power shell中输入bash，就可以使用了。

当然也有一种玩法，就是基于这个项目中使用ArchLinux的组件和更新，因为是bash对Windows的底层操作做了抽象，所以这是一种十分好玩的方式，可以试试[alwsl](https://github.com/alwsl/alwsl)这个项目，具体操作如下：

```shell
alwsl install
```

安装完成后就可以使用ArchLinux那个丰富的软件源了。

# Cygwin

许多人知道Cygwin的话应该是初学C语言或者C++语言的时候，但是Cygwin还可以干的事情有很多，Cygwin是自己在Windows上实现了POSIX的模型，利用Cygwin1.dll这个动态库实现各种操作，性能比较低，相当于在一个应用软件中运行一个应用。淡然他的可玩性也是特别高的，比如可以在Windows上体验xfce桌面环境，这几天我折腾的及时[StarLight](https://github.com/starlight)这个项目，其基于Cygwin，利用xserver实现了在Windows上的体验，安装的话和Cygwin安装差不多，不再说了，我写了我的一些玩法：

### 安装swan-desktop：

```shell
spm -i swan-desktop
```

安装完成后使用的效果如下：

![Windows上xfce效果](/images/201703/Windows上xfce效果.jpg)

### 使用zsh

starlight这个项目提供了一个zsh的实现，我们可以使用这个包，并且配合[oh-my-zsh](http://ohmyz.sh/)的强大插件，可以体验在Linux上使用各种指令的快感了：

```shell
# 安装zsh
spm -i swan-zsh

# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

