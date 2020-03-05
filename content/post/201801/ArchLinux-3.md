---
title: ArchLinux的使用(3):日常生活
date: 2018-01-09 00:07:36
tags:
  - ArchLinux
categories:
  - ArchLinux
---

在使用ArchLinux的时候，我一直在想，能不能在ArchLinux之上打造一个简单的娱乐环境。虽然我安装Steam后也能玩游戏，但是我还是建议这个事情还是在Windows上进行比较好。

## Markdown

程序员一般都离不开要写一些`Markdown`文件，对于程序员来说，编`写Markdown`有许多的选择，比如直接在IDE中安装插件编写，也可以在编辑器中安装插件写，也可以使用专门的`Markdown`编辑器进行编写。我尝试过许多的编辑器，但是我推荐的是一款可以跨平台的编辑器[Typora](https://typora.io/)来进行编辑。如果启用了ArchLinux源的话可以之间使用`pacman`下载，不然就要使用`yaourt`在AUR中下载。

```bash
# 直接使用pacman中进行安装
pacman -S typora

# 使用yaourt咱
yaourt -S typora
```

## 下载

### 普通下载

在Linux上，下载工具有许多选择，可以选择使用`curl`或者`wget`这种命令行的，也可以使用uget这种有图形界面的 ，但是我推荐的是aria2这个下载工具，直接下载就行，具体的使用和配置方法我会在以后的博文中介绍。

```bash
pacman -S aria2
```
### bt

在国内这种使用迅雷并且不太乐意分享下载速度的情况下，使用除了迅雷的客户端下载其实效果不太好，但是也没太多的办法，谁叫他们只进不出。所以在ArchLinux上面我推荐两个BT的下载客户端，一个是qBittorrent和Deluge。

```bash
# 安装qBittorrent
pacman -S qbittorrent

# 安装Deluge
pacman -S deluge
```

## 音乐

在以前，在Linux听音乐的话，除了自己下载歌曲来听的话就只能使用网页来听歌。但是自从网易和深度合作后，我们就有了另外一种选择，就是使用网易云音乐。对于网易云音乐的安装，在启用`archlinux`情况下可以之间安装，不然就要在AUR中进行安装。

```bash
# 直接安装
pacman -S netease-cloud-music

# 使用aur安装
yaourt -S netease-cloud-music
```

## 视频

说起视频客户端的话，在ArchLinux上的选择还是挺多的，我推荐的是深度的播放器，但也有其他比较好的客户端，也是值得推荐的

```bash
# 深度电影院
pacman -S deepin-movie

# vlc
pacman -S vlc

# mpv
pacman -S mpv
```

## 聊天

许多人在使用Linux的时候遇到的最大问题不是安装，不是编程环境的使用，而是安装QQ和微信，所以我也来介绍这怎么安装。我们需要的就是安装`wine`环境，这会模拟大部分的Windows的接口，基本能完成Windows上的许多操作，所以这就为我们安装聊天软件成为了可能。我们之间安装其中的包的时候，其会自动解决依赖，假如提示wine包不存在的时候，这个时候就需要启用`muitilib`这个源。

```bash
# 安装Tim
yaourt deepin-wine-tim

# 安装微信
yaourt deepin-wechat
```

这基本上已经包含了绝大部分日常使用的东西了，如果还有其他需要日常使用的功能，我会继续在此博文中添加。

