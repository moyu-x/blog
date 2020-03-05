---
title: pyenv的使用
date: 2017-07-03 23:47:47
tags:
  - pyenv
  - python
categories:
  - python
---

在使用python的时候，总是需要使用不同版本的python环境，虽然可以使用`virtualenv`，但是对于系统级别的应用和软件来说可能不怎么合适，现在来介绍和使用`pyenv`这个python的版本控制软件。

# 安装pyenv

在系统中装上`git`，然后在执行如下的命令：

```bash
# 安装git
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

# 启用

## 配置命令

安装完成之后，需要进行配置，才能在系统中使用`pyenv`，否则则会出现找不到命令的情况。

对于`bash`来说，执行如下命令：

```bash
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

对于`zsh`来说，执行如下命令：

```bash
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshenv
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshenv
$ echo 'eval "$(pyenv init -)"' >> ~/.zshenv
```

然后需要重新启动shell，执行如下命令：

```bash
# 重启shell
$ exec $SHELL
```

## 安装编译环境

对于Ubuntu来说，执行如下命令安装编译的依赖：

```bash
sudo apt install make build-essential libssl-dev zlib1g-dev
sudo apt install libbz2-dev libreadline-dev libsqlite3-dev wget curl
sudo apt install llvm libncurses5-dev libncursesw5-dev
```

对于`centos`来说，执行如下命令：

```shell
sudo yum install readline readline-devel readline-static
sudo yum install openssl openssl-devel openssl-static
sudo yum install sqlite-devel
sudo yum install bzip2-devel bzip2-libs
```

# 使用

## 基本使用

查看目前能安装版本可以使用`pyenv list`

安装制定版本（例如3.6.1）使用如下指令`pyenv install 3.6.1 -v`

在每次安装完成之后，都需要执行`pyenv rehash`来更新数据库。

切换使用的环境指令为`pyenv global 3.6.1`

## 对于大文件的安装

在安装Anaconda这类的大python的包的时候，下载的时间会非常的长，可以考虑使用以下的方法：

1. 去`https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/`查找需要版本的链接
2. 用wget从下载链接中获取文件 `Anaconda3-4.3.1-Linux-x86_64.sh`
3. 将安装包移动到 `~/.pyenv/cache/Anaconda3-4.3.1-Linux-x86_64.sh`
4. 重新执行 `pyenv install anaconda3-4.3.1 -v` 命令。该命令会检查 cache 目录下已有文件的完整性，若确认无误，则会直接使用该安装文件进行安装。

# 参考文章

1. [Python 多版本共存之 pyenv](http://seisman.info/python-pyenv.html)
2. [pyenv](https://github.com/pyenv/pyenv)