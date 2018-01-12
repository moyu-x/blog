---
title: 下载怎么办，试试Aria2
date: 2018-01-12 23:35:22
tags:
  - aria2
  - Linux
  - Archlinux
categories:
  - ArchLinux
  - Linux
---

刚使用Linux的时候，对于下载东西会有种无力感，没了迅雷，没了常见的下载工具，那怎么办呢？`wine`环境下面搞一个，还是搞下其他的工具？后来在我几经尝试之下，我发现了`Aria2`这个下载工具，所以这篇博客就是来介绍这个下载工具的使用的。

那我们的目标是什么呢，那就是我们将其伪装成了一个BT客户端，还和百度云盘和Chrome进行了集成，还是做成了一个Systemd的服务，并且有个桌面客户端，想想是不是有点激动，那就开始吧。

## 安装

对于`Aria2`这个工具来说，绝大部分的发现版已经内置在官方维护的镜像中，最大的区别就是可能在不同发行版本之下的默认版本不同，但是一般情况下也没有什么影响，所以可以一行命令就搞定这个事情。

```bash
# 以ArchLinux作为示例
pacman -S aria2
```

## 配置

安装完成了，那我们就要说一下配置了，如果只是简单的使用，配置还是很简单的。但是我们的目标是伪装成一个BT客户端，能在Chrome中使用，还能使用百度云并且还能支持开机启动，这个在配置上来说就有点麻烦，所以我们得一步一步的来。

### 基本配置

对于基本的配置来说，最重要的几点就是下载的位置，下载任务进度的保存位置以及远程访问的密码等这些配置。 这里有个示例配置的[网站](https://aria2c.com/usage.html)，我们可以在这个配置的基础上进行修改后得到我们的配置。

我个人的建议是把下载任务的回话保存到`/etc/aria2`这个文件夹下面，并且把这个文件夹的权限调高，等之后的配置会使用到。

```bash
# 创建文件夹
mkdir /etc/aria2

# 更改文件夹权限
chmod 777 /etc/aria2

# 然后在次文件夹下面创建配置文件并保存
vim /etc/aria2/aria2.conf

# 创建一个空的回话文件，不然启动的时候可能会报错
touch /etc/aria2/aria2.conf
```

### 变身服务

当写好配置文件之后，我们就可以用`aria2c`这个指令来进行开启和使用了，但是这样不是很麻烦么，每次都要进入命令行进行操作，所以我们在`/lib/systemd/system`这个文件夹下面创建一个`aria2.service`的文件，并在其中写入如下内容(注意将其中的`User`一栏换成你保存位置用户的名称)：

```
[Unit]
 Description=Aria2c download manager
 After=network.target

 [Service]
 Type=forking
 User=user
 RemainAfterExit=yes
 ExecStart=/usr/bin/aria2c --conf-path=/etc/aria2/aria2.conf -D
 ExecReload=/usr/bin/kill -HUP $MAINPID
 RestartSec=1min
 Restart=on-failure

 [Install]
 WantedBy=multi-user.target
```

在配置完成之后，我们就可以使用`systemctl start aria2.service`来启动任务了，如果需要开机启动，可以使用如下命令`systemctl enable aria2.service`

## 使用

### 百度云

百度云盘的离线下载是一个十分好的工具，我们要好好的利用。在使用百度云的时候，我们得使用一个Chrome的扩展：[BaiduExporter](https://github.com/acgotaku/BaiduExporter)，在安装完成之后，在百度云中进入简单的配置，然后可以使用他的RPC的导出方式了

### Chrome集成

此时我们的下载还有一个十分不舒服的地方那就是没法右键导出下载，并且不能简单的对下载的 任务进行管理，这个时候[yaaw](https://github.com/binux/yaaw)这个扩展就十分的好用了，直接在Chrome商店中安装后就可以使用这个服务集成了。


### webui-aria2

如果感觉这个还是有点简单了，有没有更加复杂点的了，有，那就是[webui-aria2](https://github.com/ziahamza)。这个网站可以在网页中进行aria2的控制，也给出了如何使用docker进行部署的方式，但是我还有一个更好的办法，那就是将其变成一个桌面应用，这个时候就要用到[nativefier](https://github.com/jiahaog/nativefier)这个工具了。

```bash
# 安装nativefier
npm install -g nativefie

# 生成桌面客户端
nativefier --name 'aria2' 'https://ziahamza.github.io/webui-aria2/'

# 配置桌面图标，如果是使用Gnome就使用如下指令，否则就需要根据不同版本进行设置
gnome-desktop-item-edit ~/.local/share/applications --create-new
```

现在回过头一看，是不是发现aria2这个工具的强大之处呢，我们将其伪装成了一个BT客户端，还和百度云盘和Chrome进行了集成，还是做成了一个Systemd的服务，并且有个桌面客户端，是不是特别爽，那就尽情使用吧。
