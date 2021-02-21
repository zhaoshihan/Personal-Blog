---
title: 使用Hexo搭建个人博客（VPS+Git+Apache2）
categories: 运维
tags:
  - Hexo
  - Apache
abbrlink: 24212
date: 2020-05-19 19:49:52
description:
---

长期以来购买的国外VPS只是在进行代理转发的服务，然而现在DigitalOcean的最低价格的服务器配置也相当可以了。因此最近想来再多利用一下花钱买到的资源，就有了搭建个人博客的想法。希望能通过这种方式，同时勉励一下自己养成学习记录的好习惯吧。（CS知识真的是博大精深，有时候看一遍过段时间就忘了）

我使用的是Hexo框架搭建的博客。它的好处是可以直接**在服务端部署静态的网站**，本地打包上传，相比于WordPress这种主流框架来说，更加轻量级一些。

整个的框架流程是这样的：本地的Hexo分为源代码和`generate`后的静态文件两部分；Hexo源代码放在GitHub上进行托管（`.git`与远端连接）；本地的静态文件（`.deploy_git`）与VPS上的Git仓库连接（`/home/git`），最后再通过Git Hooks更新到Apache2的服务目录下（`/var/www`）

![博客流程图](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexv2xfxowj30uf0cwwfg.jpg)

整个搭建过程最主要参考了[Hexo建站记录(VPS+git+apache2)](http://paranoth.com/2017/10/29/Hexo%E5%BB%BA%E7%AB%99%E8%AE%B0%E5%BD%95/)这篇文章，在一些细节上稍有不同。

个人的服务器是Ubuntu 18.04的版本，本地电脑是Mac OS。



## 本地端搭建Hexo

这里的搭建过程按照[Hexo 博客搭建与主题配置（零基础版）](https://mupceet.com/2019/08/build-blog-based-on-hexo/)来就可以了，只用看文章的前半部分。以上操作全部在PC端本地完成。需要注意的是，这里的Node.js版本不要使用最新的v14（我使用的是`v13.1.0`），否则在使用`npm`安装`hexo-cli`时会有莫名其妙的错误。

对于Hexo的配置（主要是`_config.yml`）和基础命令，可以查看[Hexo的官方文档](https://hexo.io/zh-cn/docs/)来进行了解。这里吐槽一下，文档写的不怎么样，但是了解一下基础还是可以的。

我不是很喜欢Hexo自带的Landscape主题。这里我搭建的Hexo使用Next.Pisces主题，当时第一次看到这个主题的博客网页就非常喜欢，就用它了！

主要参考了这篇文章：[Hexo使用next.Pisces主题](https://sjq597.github.io/2018/05/15/Hexo%E4%BD%BF%E7%94%A8next-Pisces%E4%B8%BB%E9%A2%98/)

以及Next主题的[Github源码](https://github.com/theme-next/hexo-theme-next)



## 配置VPS的Git仓库

这里我在Ubuntu上新创建了一个名为`git`的用户，远程代码仓库放在了`/home/git/BlogGit`下。在笔记本本地项目HexoBlog中的`_config.yml`配置文件里，可以查看deploy的配置。

与远程服务器的连接通信，需要生成SSH的密钥，这里[SSH官网](https://www.ssh.com/ssh/keygen/)上说的很清楚。我使用的是ECDSA的521位的密钥算法，相比于RSA 2048位，4096位的算法更为安全一些，也是目前推荐使用的新密钥算法。在使用`ssh-copy-id`命令向服务器上的`/home/git/authorized_keys`文件中传递公钥时，需要注意设置`sshd_config`的**PasswordAuthentication**为**yes**，之后再改回来。这里参考了[ssh-copy-id not working Permission denied (publickey)](https://www.digitalocean.com/community/questions/ssh-copy-id-not-working-permission-denied-publickey)问题下面的回答。

对于使用SSH进行`命令行选项`或`用户配置文件`的方式进行连接，[使用 SSH config 文件](https://daemon369.github.io/ssh/2015/03/21/using-ssh-config-file)这篇文章说的比较清楚。简而言之，使用命令行和使用用户配置文件的方式是可以等价互换的。我使用的是`用户配置文件的`方式，这样在之后的命令中更为方便一些。配置文件config在本地Mac OS上的`/Users/zsh/.ssh`目录下，上面有详细的注释。




## 配置VPS的Apache2相关

由于最初自己在搭建服务器的时候使用的是LAMP+WordPress的模式，所以基本上所有和Apache2服务器有关的坑几乎都踩过一遍了。一般来说，按照教程来是没有问题的。这里我想特别推荐一下DigitalOcean Community中的有关教程，是在是太友好了，让我这个什么都不懂的人也能比较规范的来进行服务器的搭建操作。为此每个月使用DigitalOcean花5刀维持服务器我也愿意！下面就简单说一下当时的一些操作，以及事后需要注意维护的地方。

使用namecheap申请注册免费域名，和DigitalOcean建立联系，主要参考了：

[在namecheap申请域名并在digitalocean中对该域名进行绑定](https://www.jianshu.com/p/b0fa42218ae6)

对DigitalOcean服务器droplet的操作，这里主要参考了以下几篇文章：

1. [How To Install MySQL on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04)

2. [How To Install the Apache Web Server on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04)

3. [How To Secure Apache with Let's Encrypt on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04)

4. [How To Install WordPress with LAMP on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lamp-on-ubuntu-18-04)

其中，最重要的是最后一篇文章，系统讲述了各类配置操作，前3篇都是铺垫

各个软件自动启动的设置参见了: [基于DigitalOcean+LAMP+WordPress搭建个人网站](https://zhuanlan.zhihu.com/p/55579024)

目前，申请到的域名为`zhaoshihan.me`，可以在**NameCheap**和**DigitalOcean**上登陆并在**domain-list**中查看DNS-IPv4/IPv6的对应关系。这里我只用到了3个DNS Record，分别是A Record（用于IPv4地址）、AAAA Record（用于IPv6地址）、CNAME Record（用于www的host域名转发）。对于各类DNS地址的介绍可以参考[DNS 原理入门](https://www.ruanyifeng.com/blog/2016/06/dns.html)这篇文章。



## VPS上创建Git Hooks进行自动更新

Git Hooks简单来说就是Git为开发者提供的一个生命周期的钩子，方便我们定义在Git的某些规范操作（`git commit`，`git merge`，`git push`这样的）前后的自定义个人处理操作。还是DigitalOcean Community，[How To Use Git Hooks To Automate Development and Deployment Tasks](https://www.digitalocean.com/community/tutorials/how-to-use-git-hooks-to-automate-development-and-deployment-tasks)这篇文章介绍的非常清楚，看这一篇就足够了。简单来说就是只要我们创建自己的脚本，并将它起名为规定的**Hook Name**并赋予执行权限，就可以保证在相应的Git规范操作前后执行我们自定义的脚本。在这里我使用了**post-receive**脚本，放在了`/home/git/BlogGit/hooks`目录下面。这里的BlogGit只是项目名称，可以替换。以下是**post-receive**脚本的内容。

```shell
# 这是我们自定义的一个脚本文件，
# 用途是在Hexo deploy上传到BlogGit时同步更新/var/www/zhaoshihan.me/Hexo下的文件
#!/bin/bash
GIT_REPO=/home/git/BlogGit
PUBLIC_WWW=/var/www/zhaoshihan.me/Hexo

git --work-tree=${PUBLIC_WWW} --git-dir=${GIT_REPO} checkout -f
```

最后修改一下权限，使得`git`用户拥有更改`/var/www/zhaoshihan.me/Hexo`目录下内容的能力，这里我简单粗暴的将目录下的所有文件所属用户和用户组改为了`git`用户。



## 主题美化和额外配置

这一点完全看个人的需求。我在此主要参考了[Hexo的Next主题美化设置](https://blog.mrzorg.top/Hexo/2020-02-12-hero-next-theme-settings/)这篇文章，为Hexo添加了字数统计，阅读搜索，代码块样式等功能。这里需要注意下，Next 从v7.6版本以后不再维护auto_excerpt的功能，我使用了第三方的工具[hexo-excerpt](https://github.com/chekun/hexo-excerpt)进行了自动的配置，目前看应该是按段落行数截取的。



## 日后的日常维护操作

Hexo-cli常用的一些命令，[参见中文文档](https://hexo.io/zh-cn/docs/commands)

```bash
# 发表一篇新文章的流程
$ hexo new draft "文章名"

# 使用本地server运行查看，默认访问网址为：http://localhost:4000/
$ hexo server --draft

# 将草稿文章添加到post中
# 也可以直接将_drafts文件夹中的文件直接手动移动到_posts文件夹中
$ hexo publish "文章名"

# 生成public静态文件
$ hexo generate

# 进行远程部署
$ hexo deploy

# 特定情况下，需要进行清除操作
# 清除缓存文件 (db.json) 和已生成的静态文件 (public)
$ hexo clean
```

