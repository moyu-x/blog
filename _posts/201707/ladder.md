---
title: 梯子的搭建与优化
date: 2017-07-02 08:55:08
tags:
  - shadowsock
  - vps
categories:
  - shadowsock
---

当了两个月的失踪人口，现在我回来了，继续写作、学习和工作了，所以这第一篇文章是关于shadowsock的搭建的。

现在在日常的使用中需要大量的访问国外的网站，对于以前，我还可以购买各种服务来访问国外的网站，但是现在我的要求变高了，所以只能选择自己的搭建服务器了。

# 基本要求

对于这种私人服务器的搭建，由于可能涉及到内核的修改和优化，就别在正式的服务器上使用了，我现在给出的是最小的服务器要求：

```shell
# 操作系统
Ubuntu 16.04 LTS

# 内存
>= 521M
```

# 软件安装和配置

这次使用的是对内存占用十分低的`libev`来进行安装，直接使用PPA源就可以完成这个过程。

## 安装

```shell
# 直接使用ppa源进行安装就行
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev
sudo apt-get update
sudo apt install shadowsocks-libev
```

## 配置

在使用的时候，这次使用的是多人可以公用的配置版本，主要是因为在使用单人版本的时候可能因为出现异常流量而导致不能使用，需要频繁的重启服务器。配置的文件实例如下：

```json
{
    "server": ["[::0]", "0.0.0.0"],
    "port_password": {
        "8080": "youpassowrd",
    },
    "method":"chacha20",
    "timeout": 300,
}
```

## 开启服务

对于服务的启动，我建议使用`screen`，当然也可以使用`nohup`，都是一样的。

```shell
# 创建一个新的screen窗口
screen -S shadowsock

# 开启服务
ss-manager -c shadowsock.conf
```

# 优化和安全

对于国外的服务来说，会存在高延迟和高丢包的问题，主要可从这几个方面入手进行优化：服务器位置、内核、kcp以及其它的配置。

## 安全

安全是十分重要的，谁都不想服务器刚安装好就被爆破了，虽然没有多少重要的东西，但是还是需要注意安全方面的处理。

### 修改ssh的端口

```shell
# 修改端口，修改成其他的端口就行
sudo vim /etc/ssh/sshd_config

# 重启ssh服务
sudo systemctl restart ssh
```

### 使用公钥登陆

使用公钥来登陆系统可以极大的增加安全性。

```shell
# 将公钥拷贝到此次
vim ~/.ssh/authorized_keys

# 配置ssh服务，关闭密码连接
sudo vim /etc/ssh/sshd_config

# 修改的内容如下
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      %h/.ssh/authorized_keys
PasswordAuthentication no

# 重启ssh服务
sudo systemctl restart ssh
```

## 优化

优化主要分为系统优化和链路方面的有优化

### 系统优化

#### 内核优化

优化系统内核

```shell
# 编辑内核参数
sudo vim /etc/sysctl.conf
```

在文件中添加如下内容

```shell
# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla
# forward ipv4
net.ipv4.ip_forward = 1
```

保存生效及检验

```shell
# 保持生效
sudo sysctl -p
```

检测生效

``` shell
sudo sysctl net.ipv4.tcp_available_congestion_control
```

如果结果中有hybla，则证明你的内核已开启hybla，如果没有hybla，可以用命令速冻`sudo modprobe tcp_hybla`开启。

#### TCP 优化

修改句柄文件

```shell
# 修改文件
sudo vim /etc/sysctl.conf

# 修改内容
fs.file-max = 1024000
```

编辑profile，使其自动生效

``` shell
# 编辑profile
sudo vim /etc/profile

# 追加内容
ulimit -SHn 1024000
```

然后重启服务器执行`ulimit -n`，查询返回1024000即可

### 加密优化

#### 使用M2Crypto

在Ubuntu中直接执行一下命令

```shell
sudo apt install python-m2crypto
```

#### 使用gevent

```shell
# 没有安装python环境则执行，否则跳过
sudo apt install python-dev libevent-dev python-setuptools python-gevent

# 安装gevent
sudo easy_install greenlet gevent
```

#### 使用CHACHA20加密算法

使用`libev`的话会默认支持这个加密算法，手动安装方式如下：

```shell
sudo apt install build-essential
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar xf LATEST.tar.gz && cd libsodium*
./configure && make && make install
ldconfig
```

### 网络层面

#### 使用常见端口

广大SS用户的实践经验表明，检查站（GFW）存在一种机制来降低自身的运算压力，即常用的协议端口（如http，smtp，ssh，https，ftp等）的检查较少，所以建议SS绑定这些常用的端口（如：21，22，25，80，443），速度也会有显著提升。如果你还要给小伙伴爬，那我建议开启多个端口而不是共用，这样网络会更加顺畅。

#### 开启ipV4转发

```shell
sudo sysctl -w net.ipv4.ip_forward=1
```

#### 开启TCP Fast Open

编辑内核配置文件

```shell
# 编辑内核配置文件
sudo vim /etc/sysctl.conf

## 加入如下配置
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3

# 编辑ss的配置文件
# "fast_open": false改为"fast_open": true
```

## 开启TCP BBR拥塞控制算法

开启TCP BBR拥塞控制算法需要在`Linux kernel 4.9+`内核。

### 安装新内核

新内核不受支持，所以需要谨慎安装。

```shell
# 下载新内核
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.8/linux-image-4.11.8-041108-generic_4.11.8-041108.201706290836_amd64.deb

# 安装内核
sudo dpkg -i linux-image-4.*.deb

# 删除旧内核
sudo apt autoremove

# 更新引导
sudo update-grud

# 重启
sudo reboot now

# 验证
uname -a
```

### 开启BBR

执行 `lsmod | grep bbr`，如果结果中没有 `tcp_bbr` 的话就先执行

```shell
# 启用模块
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```

执行

```shell
# 修改内核参数
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

保存生效
```shell
sudo sysctl -p
```

执行

```shell
sudo sysctl net.ipv4.tcp_available_congestion_control
sudo sysctl net.ipv4.tcp_congestion_control
```

如果结果都有`bbr`, 则证明你的内核已开启bbr

看到有 tcp_bbr 模块即说明bbr已启动

## 使用KCPtun

这个是一个双边加速的方案，欢迎各位使用

### 下载kcptun

```shell
# 下载
wget https://github.com/xtaci/kcptun/releases/download/v20170525/kcptun-linux-amd64-20170525.tar.gz

# 解压
tar -zxvf kcptun*.tar.gz
```

### 服务端使用

执行以命令

```shell
./server_linux_amd64 -t "127.0.0.1:8080" -l ":4000" -mode fast2
```

### 客户端使用

执行一下命令，可以将其保持问`.bat`文件

```shell
client_windows_amd64 -r "server_ip:4000" -l ":8081" -mode fast2
```

### ss配置

此时的配置和常规不同，具体如下

服务端地址为：`127.0.0.1`

服务端端口为：`8081`

加密算法为：`chacha20`

密码：`ss的密码`

这个时候就可以正常的使用梯子了。

# 参考文献

1. [shadowsocks optimize](https://github.com/iMeiji/shadowsocks_install/wiki/shadowsocks-optimize)
2. [DEPLOY SHADOWSOCKS-GO ON VPS](https://wuwenhan.top/web/deploy-shadowsocks-go-on-vps/)
3. [开启TCP BBR拥塞控制算法](https://github.com/iMeiji/shadowsocks_install/wiki/%E5%BC%80%E5%90%AFTCP-BBR%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95)