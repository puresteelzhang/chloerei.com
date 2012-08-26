---
layout: post
title: 写给大家看的 Rails 部署：第二篇 Linux 用户权限
---

在上一篇《[简单快捷的部署方案][1]》中已经展示了如果用 Passenger 部署一个 Rails 应用，观察评论后发现比较多的问题出在用户权限上，所以增加一篇用户权限的向导。

## Linux 用户权限

要在 Linux 上部署 Web 应用，对于用户权限的知识是绕不开的。本文只会简单介绍部署用到的权限知识，想要搭建安全的服务端环境，还需要阅读一些全面的 Linux Server 方面书籍。

### 用户和用户组

Linux 系统用于划分用户权限的单位是用户和用户组。用户登录、访问文件、执行命令，都会牵涉到用户和用户组的权限问题。你可以用以下命令查看自己当前的用户和用户组：

    $ whoami # 查看当前用户名
    $ groups [name] # 查看名为 name 的用户所属的用户组

一个用户可以属于多个用户组，一个用户组也可以包含多个用户。用户和用户组的灵活运用，就可以定制更安全的系统策略。我们之前用到的 Multi-User RVM 就是使用了用户组，将需要使用 rvm 的用户添加到 rvm 用户组中，只有该用户组的成员可以正常使用 rvm。

### 文件权限

每个 Linux 文件本身储存了用户权限的信息，现在来看一个 Rails 项目默认的权限情况：

<pre><code class="bash">~/workspace/code_campo:master$ ls -l
总用量 72
drwxrwxr-x 9 rei rei 4096  6月 24 17:06 app
-rw-rw-r-- 1 rei rei  317  6月 24 17:06 Capfile
drwxrwxr-x 5 rei rei 4096  8月  1 02:34 config
-rw-rw-r-- 1 rei rei  159  6月 24 17:06 config.ru
drwxrwxr-x 2 rei rei 4096  6月 24 17:08 db
drwxrwxr-x 2 rei rei 4096  6月 24 17:06 doc
-rw-rw-r-- 1 rei rei  898  8月 14 23:54 Gemfile
-rw-rw-r-- 1 rei rei 4654  8月 14 23:56 Gemfile.lock
drwxrwxr-x 4 rei rei 4096  6月 24 17:06 lib
drwxrwxr-x 2 rei rei 4096  6月 24 17:31 log
drwxrwxr-x 2 rei rei 4096  7月  1 21:40 public
-rw-rw-r-- 1 rei rei  274  6月 24 17:06 Rakefile
-rw-rw-r-- 1 rei rei  605  7月 24 23:42 README.md
drwxrwxr-x 2 rei rei 4096  6月 24 17:06 script
drwxrwxr-x 7 rei rei 4096  6月 24 17:06 test
drwxrwxr-x 6 rei rei 4096  6月 24 17:18 tmp
drwxrwxr-x 4 rei rei 4096  6月 24 17:06 vendor</code></pre>

首先看第一列的内容，这串字符要怎么理解呢？以 app 目录的权限 `drwxrwxr-x` 为例，第一个 `d` 的意思是这是个目录，紧接着的 9 个字符(rwxrwxr-x)可以分开成 3 组，分别代表「所属用户」「所属组的成员」「其他用户」对该文件或文件夹的权限。每组权限都是由「rwx」字符组成，每个字符分别代表 r-读权限，w-写权限，x-执行权限。rwx 的位置是不变的，如果是 - 号，则代表没有这项权限。

所以 `drwxrwxr-x` 的涵义如下：

> 这是个文件夹，文件夹的所属用户和所属组的成员对其有读写和执行的权限，而其他用户则只有读和执行的权限。

这里要特别指出的是 `x` 权限在对应文件夹和文件的时候意义是不同的，如果这是个文件夹，`x` 权限实际指的是「打开文件夹阅读里面的内容」的权限，而对于文件，`x` 才代表「执行这个文件」的权限。

`rwx` 权限也可以用数字表示，它们分别对应 r-4 w-2 x-1，`rwx` 就是将每个字母分别对应的数字相加，结果是 7，所以 `rwxrwxr-x` 也就对应 775。数字形式的权限会用在改变文件权限时候用到。

接着我们看第三、四列，这分别代表文件所属的用户和用户组，可以看到这个目录下所有文件的所属用户和用户组都是 rei。

有时需要对整个项目的所属用户和用户组变更，可以在项目目录下使用这条命令

    sudo chown webuser:webuser . -R

这条命令把 . 目录（也就是 codecmapo 目录)下的所有文件转交给 webuser 用户和 webuser 用户组。大部分情况下，Rails 项目的文件权限都是不需要改变的。

## Nginx + Passenger 部署的用户权限

正常情况下，Nginx + Passenger 的权限应该是这样的：

    nginx worker(nobody) -> passenger worker(项目文件所属用户) -> 项目文件

其中，nginx worker 默认情况下就是 nobody 了，不需要管理。passenger worker 的用户是根据项目文件自动切换的，如果项目文件是属于 rei 的，那么 passenger worker 就是以 rei 的身份执行的（原理就是 passenger 的管理进程是以 root 来跑的，可以决定 worker 的身份），所以需要管理的其实是项目文件所属的用户。

项目文件属于什么用户没有什么规定，我喜欢新建一个专门用来跑 web 服务的用户：webuser，并且把项目文件放到 webuser 的 home 目录下。然后 webuser 用户要能正常使用 rvm，ruby，正常访问数据库、缓存服务等资源。

但是有一个例外，项目目录不能是属于 root 的。因为 passenger 在遇到 root 拥有的项目文件时，会认为以 root 身份启动 passenger worker 权限过大了，passenger 会自动降级到一个没什么权限的备用用户身份（通常是 nobody），很多人在部署时遇到权限问题都是因为用了 root 身份来拷贝项目文件，项目文件成了 root 拥有的文件，被 passenger 自动降权。

所以，如果用 passenger 的时候遇到了权限问题，看看项目文件的所有者是不是 root，如果是的话改成别的用户试试。

## 总结

Linux 是 Rails 最好的部署平台，使用 Linux 的话，权限和其他 Linux 方面的知识是不能缺的。如果在部署 Rails 程序的时候遇到了很多问题，那么就应该好好的补一下 Linux 的知识了。

本文简单介绍了 Linux 权限相关的知识，并且分析了 Passenger 部署模式的用户身份策略。

下一篇部署向导：自动化部署。

  [1]: http://chloerei.com/2012/08/05/rails-deploy-guides-1-base-deploy/
