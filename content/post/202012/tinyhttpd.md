---
title: Tinyhttpd 源码阅读
date: 2020-12-04
tags:
  - C
  - server
categories:
  - 源码阅读
---

仓库地址：[http://tinyhttpd.sourceforge.net/](http://tinyhttpd.sourceforge.net/)

HTTP的相关资料可以再MDN上查到，地址为[https://developer.mozilla.org/zh-CN/docs/Web/HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

![TinyHttpD](/images/202012/http-request.png)

## 一些常用定义

`socketlen_t`定义：在socket编程中的accept函数的第三个参数的长度必须和int的长度相同，于是有了此类型。

`memset`函数：`void *memset(void *str, int c, size_t n)` 复制字符 c到参数 str 所指向的字符串的前 n 个字符。

isspace函数：

## Main函数

**执行流程：**参数的声明 ⇒ 调用`startup`函数启动一个服务 ⇒ 在循环中持续的对端口进行监听，调用`accept`函数处理请求 ⇒ 调用`close`函数关闭服务

## startup

用于启动一个进程在特定端口监听网络服务连接，当端口号是0的时候，会随机选择一个端口，并且会修改原有的端口变量为真实的端口。

**执行流程：**创建`socket`连接 ⇒ 处理socket错误 ⇒ 清空name信息 ⇒ 设置name的参数：使用的协议，`AF_INET`是IPV4，监听的端口，主机地址 ⇒ 设置端口复用并处理错误`setsocketopt` ⇒ 使用`bind`函数绑定socket端口 ⇒ 处理是否是随机端口，是则获取一个可用端口 ⇒ 调用`listen`函数设置监听的上限数

## accept_request

用于从服务器端口上接受一个请求并返回数据。

执行流程：

1. 开始都是一些变量的声明，由于代码比较老，这个地方使用的还是早期的CGI逻辑
2. 读取请求体的第一行

    这里读取的是client端发送过来的报文，通过`#define ISspace(x) isspace((int)(x))`定义的`ISSpace`对读取的数据进行判断，如果不是空字符串就继续读取，直到达到method参数的限制。

    这个地方在数组读取的末尾显示的指定`\0`，可以认为这个地方数据存在复用，这样在后续的读取过程中直接读取此字符则能明白已经是数据的结尾

3. 使用`strcasecmp`函数对请求方法进行判断。
    1. 由于编写这段代码时间的原因，其本身就只能支持`GET`和`POST`方法；并且没有规定方法必须严格的大写，所以得在判断相等的时候忽略大小写
    2. 是`POST`请求的时候，将`cgi`的参数置为1
4. 读取url的信息到`url`的`char`数组中，此时的url中是可能包含有查询参数的，因为历史的原因，可以发现在`POST`请求中带有查询参数是没有用的。
5. 当发现是`GET`请求的时候，要将?后的查询参数读取到`query_string`中，并且由于代码复用的问题，在结尾也要显示使用`\0`
6. 使用`sprintf`函数将url格式化到`htdocs%s`中，最后赋值给path变量
7. 当路径只是`/`，则将`index.html`拼接到`path`变量之后
8. 使用`stat`函数判断在系统中请求路径的信息是否存在，不存在则将后续的数据全部抛弃，最后调用`not_found`函数提示报错信息

    否则进入实际请求。先判断请求的模式是否是文件夹`(st.st_mode & S_IFMT) == S_IFDIR`，也就是根请求，是则将`/index.html`追加到path信息中。

    接下来判断文件是否有可执行权限，有则将`cgi`设置为`1`，最后根据`cgi`的值调用不同的函数进行处理。

## serve_file

用于向客户端传递读取的文件

1. 先读取此次请求的头信息并将其丢弃
2. 读取文件，文件不存在则用`not_found`函数提示报错。否则使用`headers`函数设置响应头信息，用`cat`函数将读取的数据传递给`client`
3. 关闭读取的文件

## execute_cgi

用于执行`cgi`脚本

1. 是`GET`请求的时候，读取并丢弃后续的`headers`信息
2. 是POST请求的时候读取`Content-Length`数据，这里并没有读取其他的请求头信息，如果其为空，则使用`bad_request` 函数向`client`发送错误信息
3. 创建`cgi_input`和`cgi_output`两个管道，创建不成功则使用`canot_execute`函数向`client`传递报错信息
4.  创建一个子进程用于执行脚本
    1. 将子进程的标准输出重定向到`cgi_output`，将子进程的标准输入重定向到`cgi_input`上
    2. 设置环境变量，将这个环境变量添加到子进程的环境变量中
    3. 根据请求类型的不同构造不同的环境变量
    4. 使用`execl`函数执行脚本，然后退出此函数
5. 创建子进程不成功则进行关闭处理
    1. 首先关闭`cgi_ouput`的写和`cgi_input`的读
    2. 是`POST`方法则继续读取body的信息，并将信息写入到`cgi_input`中
    3. 从`cgi_output`管道中读取数，调用send函数向`client`发送数据
    4. 关闭两个管道
    5. 等待子进程的退出

