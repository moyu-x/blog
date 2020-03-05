---
title: ArchLinux的使用(1):安装
date: 2018-01-04 20:36:53
tags:
  - ArchLinux
categories:
  - ArchLinux
---

在折腾了许多的Linux的发行版本之后，我最终选择的ArchLinux作为我的日常使用版本，具体的心路历程那就多了，反正上了Arch的这条船之后我就没想在下去了。在这之间我还玩了一段时间的Manjaro，但是9月份发生的一次系统更新mongo的版本发生问题导致我本地的测试数据发生错误之后我就没有再使用了，凭自己良心说，这其实是最好上手的Arch的发行版本了。

我使用的是Intel的CPU和NVIDIA的GPU，所以其他的我没折腾过，如果发现问题的话还是请参考官方的文档进行解决（其实我想说的是，官方的文档已经写的很好了，这个只是我的总结而已）。

## 安装准备

### 空间

在安装ArchLinux之前需要有一个没有被分过区的剩余空间，如果安装的时候发现没有的话，可以在安装的过程中删除没有文件的分区来解决这一个问题

### 安装盘

去[官方网站](www.archlinux.org)或者[清华源](https://mirrors.tuna.tsinghua.edu.cn)上面下载最新的镜像进行刻录。理论上来说使用老的镜像进行安装也是可以的，但是安装的过程中会把安装的软件更新到最新的版本，所以没有意义。

然后使用[rufus](https://rufus.akeo.ie/)这个软件将下下来的镜像刻录成USB的启动盘，这是就需要选择UEFI还是BIOS启动了，这两个根据硬件来自行进行选择吧。如果是要装双系统的，应该优先安装Windows，并且需要关闭Windows的快速启动和安全启动功能，这样才能安全的安装ArchLinux。

## 安装

对于此安装过程，我默认是你用的是默认的键盘，默认的字体还有就是使用网线连接而不是WiFi进行网络连接进行安装，这会在安装过程中少去很多的麻烦，如果需要修改可以到安装完成之后才进行。

### 时间

首先要将本地的时间和网络的时间进行同步，时间同步在操作系统内部是十分重要的

```bash
timedatectl set-ntp true
```

### 设置安装源

由于众所周知的原因，我们得先设置安装的镜像源，这样才能不会花太多的时间咋安装的过程中，并且这个安装的配置还会应用到安装后的系统中。

```bash
sed -i '/China/!{n;/Server/s/^/#/};t;n' /etc/pacman.d/mirrorlist
```

### 分区

在Linux系统中，至少需要一个根分区，如果使用交换文件的话是不需要使用交换分区的，不适用交换问价的花需要单独的配置一个交换分区。如果配置双系统或者是UEFI的话，需要单独配置一个EFI分区，分区的类型为FAT32。使用`lsblk`来查看文件分区，我建议是使用cfdisk来进行分区，这个命令行软件比较人性化一点。在分区完成之后就需要将这些分区进行格式化。

```bash
# 格式化EFI分区
mkfs.vfat -F32 /dev/sdax

# 格式化普通分区
mkfs.ext4 /dev/sdax

# 格式化交换分区，为了防止交换分区不能挂载，还是进行格式化比较好一点
mkswap /dev/sdax
```

在格式化之后，就需要想各个分区进行挂载，使用UEFI的话需要将EFI分区挂载到`/mnt/boot/EFI`之下，这里的统一挂载点就为`/mnt`

### 安装基本系统

对于基本系统的安装，我建议安装完整的`base`和`base-devel`这两个安装集合

```bash
pacstrap /mnt base base-devel
```

### 生成fstab

如果这一部出了问题，如果只分了一个分区的话应该还能启动，如果不是的话启动的可能性就变低了。在生产之后，我认为还是使用`cat`命令看看比较合适

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## 配置基础系统

### 进入基本系统

使用`arch-root`进入到安装的安装的系统进行配置

```bash
arch-root /mnt
```

### 设置时间

配置Locale，将需要的行前面的注释符号去掉，我建议只使用`en_US`,`zh_CN`,'zh_TW'的UTF-8的字符集

```bash
vim /etc/locale.gen
```

然后使用`locale-gen`生产locale，然后使用以下命令设置默认的Locale

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### 时区

国内可以使用上海的时区来进行时间的设置

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```


### 硬件时间

如果不是双系统的话，强烈建议使用UTC时间，如果是双系统的话，我建议还是使用硬件时间，这主要是Linux和Windows对时间的计算方式不同造成的，不然会出现一些问题，最常见的就是GPG签名验证的问题。

```bash
# 使用硬件时间
hwclock --systohc

# 使用硬件时间
hwclock --systohc --localtime
```

### 主机名

```bash
echo <主机名> /etc/hostname
```

### 设置root密码

```bash
passwd
```

### 安装引导程序

我建议使用GRUB作为引导程序，如果是用其他的请查看官方文档

#### BIOS

```bash
pacman -S grub os-prober
grub-install --target=i386-pc /dev/sdx
grub-mkconfig -o /boot/grub/grub.cfg
```

#### UEFI

```bash
pacman -S dosfstools grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/mnt/boot/EFI --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## 安装完成

对于网络，如果是只使用命令行的话，那就要开启dhcpcd来进行IP分配`systemctl enable dhcpcd`，如果使用图形界面的话，还是先启用这个，之后再进行配置。

使用`exit`退出安装环境，然后使用`umount -R /mnt`取消挂载，使用`reboot`重启后就可以基本使用了

## 基本配置

### 用户

日常使用root用户是相当危险的，所以配置一个普通用户进行日常操作还是一个明智的做法，我们这里配置一个在user和root之间的用户

```bash
uaeradd -m -G wheel -s /bin/bash username
passwd username
```

赋予其sudo权利，编辑`/etc/sudoers`，将`%wheel ALL=(ALL) ALL`取消注释就行

### 显卡配置

#### intel

```bash
pacman -S xf86-video-intel
```

#### nvidia

```bash
pacman -S nvidia
```

### 安装桌面环境

对于各种桌面环境都有其各自的特点，我就以Gnome的配置作为示例：

```bash
# 安装Gnome
pacman -S gnome

# 启动Gdm
systemctl enable gdm

# 启动NetworkManager
systemctl enable NetworkManager
```

到此为止，应该能进行基本的使用了。

