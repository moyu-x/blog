---
title: Hexo博客：部署与插件
date: 2018-02-22 14:07:33
tags:
  - blog
  - Nginx
  - 博客
categories:
  - 博客
---

在我两年前写的博客的[搭建博客的简单步骤](https://www.mosdev.xyz/2015/11/21/2015/搭建博客/)的这篇博文中，我简单的介绍的如何搭建一个基于Github的Hexo博客，现在我来介绍一些其他的用法和部署的方式。

## 插件

以前使用Hexo而不是Wordpress搭建博客主要原因之一就是可以在Github之类的服务上搭建博客而不需要自己搭建服务器，这样就减少的在金钱方面的投资。如果是现在我还是会在这些方面进行投资的，在以前Hexo的插件不是特别的多，尤其是适合国内的插件和主题都不怎么多，所以我一直都是使用[Next](https://github.com/theme-next/hexo-theme-next)作为我的网站的主题的。

其实这个主题有许多的对国内支持的管理，我感觉十分的方便，如果需要其他的插件可以到官网上去看一下[插件](https://hexo.io/plugins/)这个分类，下面有许多的目录值得我们去使用。

## 部署

在自己的服务器上部署Hexo的方式有许多，可以将服务器的文件打包成Dcoker的镜像进行使用，也可以将使用各种的静态转发的服务器使用，例如：`http-server`以及`httpd`等。我这介绍的将是使用`Nginx`进行部署并且使用`cerbot`生成SSL证书。

*下面我将以我的博客的部署作为演示，并且服务器用的是`Ubuntu 17.10`*

### Nginx

我们先将数据从我们的GitHub或者其他的代码管理服务上面克隆下来，存到服务器的`/var/www/mosdev.xyz`的目录下面。然后在`/etc/nginx/sites-available`中建立一个新的配置文件，例如`blog`，在其中写入如下的配置内容：

```nginx
server {
        root /var/www/mosdev.xyz;

        index index.html index.htm index.nginx-debian.html;

        server_name www.mosdev.xyz;

        location / {
                try_files $uri $uri/ =404;
        }
}

```

这个配置会默认监听80端口，如果休要监听其他的端口可以在配置文件中加入`listen`字段来进行监听。如果你的配置是正确的话，这个时候删除原来在`/etc/nginx/sites-enabled`的默认配置，然后重新建立软连接`ln -sf ../sites-available/mosdev ./`，如果配置都是正确的话，将在输入`nginx -s reload`的信号后访问你的域名将会看到你部署的博客了。

 现在`https`已经是网站部署的主流了，所以我们也考虑将网站部署上SSL证书，启用`https`，最后将不是`https`的访问都强制转换成http的访问。

### Cerbot

部署`https`的一个问题是我们得有SSL证书，以前的SSL证书都比较贵，并且对于个人来说应该不会愿意去购买这么一个证书去部署`https`的，但是从16年开始，Let's Encrypt 提供了免费的SSL证书，这个时候我们就可以使用这么一个免费的服务去部署一个SSL证书了。

#### 安装

首先当然得是先在服务器上安装cerbot，主要的安装方式有使用Docker或者直接从官方源安装两种：

1. 从官方源安装，这个是没有Nginx插件的：

   ```bash
   sudo apt-get update
   sudo apt-get install software-properties-common
   sudo add-apt-repository ppa:certbot/certbot
   sudo apt-get update
   sudo apt-get install certbot 
   ```

2. Docker安装，这个的前提当然是你得在服务器上面安装了个Dockers才行：

   ```bash
   sudo docker run -it --rm -p 443:443 -p 80:80 --name certbot \
               -v "/etc/letsencrypt:/etc/letsencrypt" \
               -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
               certbot/certbot certonly
   ```

虽然我比较喜欢使用Docker进行部署，但是在cerbot这个上面我还是使用了官方源进行安装，这样可以在我更新系统的时候自动进行更新，并且这次我们使用了Nginx进行部署，所以我这次需要安装的是带有Nginx插件的cerbot，具体的安装步骤如下：

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx 
```

#### 使用

因为这次我们使用了带有Nginx插件的cerbot，我们首先得确定我们的配置文件是否是正确的`nginx -t`，然后我们执行一下的命令就可以新生成证书和切换 http流量到https了：

```bash
# 可以自动的修改原有的配置文件，并且在选项中有自动跳转
sudo certbot --nginx
# 如果只是想生成证书，不修改Nginx的配置，可以用如下代码
sudo certbot --nginx certonly
# 配置自动获取证书
sudo certbot renew --dry-run
```

如果使用的是没有Nginx插件的 cerbot，我们得运行如下代码生成证书：

```bash
sudo certbot certonly --webroot -w /var/www/mosdev.xyz -d www.mosdev.xyz
```

然后修改配置文件如下，也是最终的配置文件：

```nginx
server {
        root /var/www/mosdev.xyz;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name www.mosdev.xyz;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/www.mosdev.xyz/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/www.mosdev.xyz/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = www.mosdev.xyz) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80 default_server;
        listen [::]:80 default_server;

        server_name www.mosdev.xyz;
    return 404; # managed by Certbot


}
  
```

至此，你的网站已经在你的VPS上进行了部署，使用了Nginx作为服务器，并且在服务器上面部署了HTTPS，这样也就达成了我们的目标了。

## 参考文章

1. [How To Set Up Let's Encrypt with Nginx Server Blocks on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-with-nginx-server-blocks-on-ubuntu-16-04)
2. [Cerbot Document](https://certbot.eff.org/all-instructions/)
