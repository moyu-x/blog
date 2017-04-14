---
title: ElementaryOS使用及美化
date: 2017-04-14 08:01:55
tags:
  - Linux
categories:
  - Linux
---

前几天微软的创造版更新了，导致我以前安装的双系统中的引导被覆盖了，所以我在将数据拷贝出来之后，就准备找一个发行版玩玩了，这一折腾就是一个周的时间，其中大部分的时候都在折腾最近一个周都在折腾Linux的安装和各种的配置。在折腾好几个发行版之后，从Archlinux到openSUSE，还有就是一直使用的Ubuntu都使用过，最后我使用了ElementaryOS这样一个发行版。

# ElementaryOS简介

ElementaryOS是一个基于Ubuntu LTS的一个发行版本，选择这个发行版主要是因为他的界面设计非常的好看，如果在加上一些自己的配置之后，甚至可以从外观上构建成一个高仿的Mac，现在的最新版本是基于Ubuntu 16.04的Loki，每个发行版都是使用一个神的名字进行命名的。

在其中，整体的一个设计风格偏向与Mac的风格，但是也有自己的一些设计的风格在其中有所展现。在开发上面，使用的基于Gnome桌面环境的开发，并且使用了一个Vala的语言来开发整个的的系统界面，整个的设计风格我是十分的喜欢的。

这个是我在对界面和图标进行优化后的效果图：

![ElementaryOS](/Image/ElementaryOS/ElementaryOS.png)

# 安装

按照一般的情况来说，应该不用介绍安装的过程了吧，但是如果你是拥有一个NVIDIA双显卡的人，这个按照步骤对你来说是有相当大的帮助的，因为你可能进不去安装界面，也有可能高高兴兴的安装好了之后发现打不开桌面环境，这是不是一个相当悲剧的信息，所以我下面的这个方法可以很简单的解决这个问题：

```shell
# 在Linux这一行有个quite后面加上这么一段
nouveau.modeset=0
```

这个主要是因为内核模块自己附带了开源的英伟达的驱动，这个会造成双显卡工作时候的冲突，如果发生这样的事情，造成的结果就是显示界面进不去，在进去之后安装英伟达的闭源驱动就行。

# 美化

## 安装twaks-tool

可以使用这个工具来安装自己需要的主题，具体的安装步骤如下：

```shell
# 安装PPA工具，来使用PPA源
sudo apt install software-properties-gtk

# 安装teaks-tool
sudo add-apt-repository ppa:philip.scott/elementary-tweaks
sudo apt-get update
sudo apt-get install elementary-tweaks
```

## 安装Mac-arc主题

这个主题是一个高仿的Mac主题，可以是你的整个桌面像Mac一样，逼格是不是有点高了：

```shell
# 下载安装包之后运行
sudo dpkg -i osx-arc-collection_1.3.9_amd64.deb
```

## 安装la-capitaine-icon-theme

这个图标包设计的相当的不错，如果不喜欢的话，还有另外一个图标包可以推荐，那就是numix这个，安装方法如下：

```shell
git clone https://github.com/keeferrourke/la-capitaine-icon-theme.git
sudo cp -r la-capitaine-icon-theme /usr/share/icons
```

## 安装plank的主题

```shell
git clone https://github.com/fsvh/plank-themes.git
cd plank-themes
./install.sh
```

# 驱动

## NVIDIA的闭源驱动

有一个团队在维护这么一个NVIDIA的驱动源，官方现在的版本是375，但是使用这个源的话可以使用到378的的显卡驱动：

```shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install nvidia-378
```

## Intel的驱动

对于使用基于Ubuntu系列的发行版来说，可以使用Intel自己出的一个驱动管理工具来管理intel的驱动，这个驱动管理软件就是[Intel Graphics for Linux](https://01.org/zh/linuxgraphics)，安装过程如下：

```shell
# 下载工具
wget https://download.01.org/gfx/ubuntu/16.04/main/pool/main/i/intel-graphics-update-tool/intel-graphics-update-tool_2.0.2_amd64.deb

# 导入key
wget --no-check-certificate https://download.01.org/gfx/RPM-GPG-KEY-ilg-4 -O - | sudo apt-key add -
sudo apt update
sudo apt upgrade

# 安装
sudo dpkg -i intel-graphics-update-tool_2.0.2_amd64.deb
```

# 软件

## Typora

这是一个markdown的编辑器，对我来说，使用了一年左右，感觉上相当的不错，推荐一下，这个是一个全平台的软件，在Ubuntu下的安装方式如下（我直接复制官网的了）：

```shell
# optional, but recommended
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io ./linux/'
sudo apt-get update
# install typora
sudo apt-get install typora
```

#  在Ubuntu上使用

当然，如果不想使用这么一个发行版，也还是可以在Ubuntu上使用这么一个桌面环境的，安装步骤也十分的简单，直接使用PPA源就可以了：

```shell
sudo add-apt-repository ppa:elementary-os/stable
sudo apt update
sudo apt install elementary-desktop
```

# 写在最后

在使用Linux前，可以先看看这个[百度Linux吧炸道贴](https://github.com/910JQK/ZhaDao)，然后评估一下自己是否适合使用这么一个系统，如果要使用的话，也还要许多的替代方案可以进行使用，例如：vagrant、虚拟机以及docker容器等。



