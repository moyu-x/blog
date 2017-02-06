---
title: 开始使用Linux
update: 2017-2-5 15:30:50
date: 2017-01-30 23:03:58
tags:
  - Linux
categories:
  - Linux
---

使用Linux已经几年了，当然从技术上来说，我应该还只能算上一个新手。玩过了许多的发行版本，装了多次的双系统和虚拟机，其中最为佩服的三个发行版为ArchLinux、Ubuntu和CentOS。现在我还是想写一篇文章来介绍一下我的日常配置，当然也是为了我不知道什么时候手痒又把虚拟机中重新装一遍（现在我已经不再实体机上玩了，感觉浪费时间）。

先说安装，Ubuntu的安装时最简单，基本不用进行多少处理，和windows的安装差不多。其次是centos，使用centos主要是学习[《鸟哥的Linux的私房菜》](http://linux.vbird.org/)，当然我的私人服务器上的Linux版本也是选择的CentOS，主要看中的是他的稳定性（虽然现在只是跑了个Minecraft）。最麻烦的算是ArchLinux了，其实看[官方维基](https://wiki.archlinux.org/index.php/Installation_guide)就能安装成功，顺便还把Linux的一些基础知识了解了，主要分为以下几个大步骤：

1. 基本配置
2. 分区
3. 安装引导
4. 安装基本软件包
5. 安装驱动和桌面环境

在安装完成之后，就是需要进行各种配置了，一般我主要使用Linux来进行开发或者学习，所以接下来的配置就是开发环境的配置了。在ArchLinux中的包管理器为pacman，但是有个前端yaourt比较好用，Ubuntu的包管理器为apt，CentOS的包管理器为yum，我日常主要使用Ubuntu，因为滚蹦过几次：

1. 安装Ubuntu-make

   ```bash
   sudo add-apt-repository ppa:ubuntu-desktop/ubuntu-make
   sudo apt update
   sudo apt install ubunt-make
   ```

2. 安装jdk，这里使用WebUpd8团队提供的安装方式，会自动配置环境变量，并且调整默认使用的Java环境：

   ```bash
   sudo add-apt-repository ppa:webupd8team/java
   sudo apt update
   sudo apt install oracle-java8-installer
   ```

3. 安装nodejs并配置

   ```bash
   # 安装，默认安装在node文件件下
   umake nodejs nodejs-lang

   # 配置，在.bashrc或者.zshrc中配置环境变量，并将npm包也配置到path中
   export PATH="$PATH:${HOME}/node/bin"
   export PATH="$PATH:${HOME}/node_modules/bin"

   # 使用淘宝的镜像，在.npmrc文件中写入
   registry=https://registry.npm.taobao.org
   ```

4. 安装MySQL数据库，虽然大部分时候推荐使用Mariadb，但是还是得安装MySQL数据库进行使用和学习

   ```bash
   # 在Ubuntu 16.04和Ubuntu 16.10中使用的是5.7版本
   sudo apt install mysql-server mysql-client msyql-workbench
   ```

5. 安装Clang和llvm，虽然gcc是个的好编译器，但是其错位提示不好，所有错误提示好一点的clang了：

   ```bash
   sudo apt install clang llvm
   ```

6. 安装Visual Studio Code

   ```bash
   umake ide visual-studio-code
   ```

7. 使用oh-my-zsh

   ```bash
   # 安装
   sudo install git zsh
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

   # 配置，使用默认的主题，简洁，跳转alias
   alias zshconfig="vim ~/.zshrc"
   alias ohmyzsh="vim ~/.oh-my-zsh"
   ```

8. 使用vim，安装我不说了，对于插件有两个选择，一个[spf13](http://vim.spf13.com/)，另一个是国人写的[spacevim](http://spacevim.org/)

9. 关闭Ubuntu的访客模式：

   ```bash
   # 编辑以下文件，没有就创建一个
   sudo vim /etc/lightdm/lightdm.conf

   # 在配置文件中写入以下内容
   [SeatDefaults]
   greeter-session=unity-greeter
   allow-guest=false
   ```

到现在，基本的配置已经完成了，现在就可以进入写代码的环节了。当然我写的这个博客不适合所有的人，只是我记录我日常使用时的安装和配置过程，简单一些，明白一些，方便我下一次手痒是将系统重新装了。