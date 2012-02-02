---
layout: post
title: Vundle，Vim 的 Bundle
---

长久以来，我管理 Vim 配置的方式都非常原始—— zip 打包，然后发到邮箱上。偶尔会发生忘记备份，或者配置混淆的状况，不过由于懒筋发作，竟然这个方案就这么用了两年。

终有一天，我觉得这个方法太笨了，作为一个高效程序员怎么能使用这么纯手工的备份方案，Vim 可是我的吃饭家伙啊。

Vim 配置备份最麻烦的部分就是脚本管理了，如果不先解决脚本管理，多次安装/卸载 Vim 脚本之后配置文件夹肯定乱糟糟的。于是我去找有什么潮流的插件管理方案，找到了最好的工具：Vundle（[项目页][1]）。

![Vundle install example](https://lh3.googleusercontent.com/-4EnLqLpEZlk/TlqXWpgWxOI/AAAAAAAAHRw/oBAl6s1hj7U/vundle-install2.png)

说它最好是基于几个理由：

* 灵感来源于 Ruby 社区的 Bundle 工具，语法相似。
* 配置干净，只需在 .vimrc 里面写入需要安装的脚本，就可以使 Vim 自动安装。 
* 可以从 github 上安装 Vim 脚本

详细的使用可以参考[项目页][1]的教程。

由于 Vundle 从安装到使用都非常适合脚本化，所以我在学会这个工具之后马上写了一个安装脚本，加上我的 .vimrc 等文件，放到了 github（[chloerei/vimrc][2]）上。

现在我想要在一台新电脑还原我的 vim 配置，只要确保有 vim，ruby，rake，rvm 的情况下，运行以下命令：

    git clone git@github.com:chloerei/vimrc.git
    cd vimrc
    rake deploy

我熟悉的配置就会部署到电脑上。

不妨讲解一下 Rakefile 文件，这是 `rake deploy` 魔法的秘密。

    desc "deploy vimrc"
    task :deploy do
      # Bundle and scripts
      system 'git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle'
      system 'cp .vimrc .gvimrc ~/'
      system 'vim +BundleInstall +qa'
      system 'cd ~/.vim/bundle/Command-T/ruby/command-t/; rvm system do ruby extconf.rb; make; cd -'

      # snipmate-snippets
      system 'git submodule init; git submodule update'
      system 'cd snipmate-snippets/; rake deploy_local; cd -'
    end

第 1～3 个 system 命令安装了 vundle，并且打开 vim 使用 BundleInstall 命令安装所有写在 .vimrc 里的脚本。
第 4 个 system 命令对 Command-T 这个脚本进行了本地编译。
最后 2 个 system 命令用 git submodule 抓取了我放在另外的 github 源的 snipmate 代码片段。

由于我对 Rake 毕竟熟悉，所以脚本用了 Rakefile 的形式，其他开发者完全可以用 make 或者 bash 来写脚本。而除了 1～3 个 system 调用，后面的处理都是可选的，取决于需要什么 vim scrpit。

每个 Vimer 都有自己的喜好配置，我这份配置只算抛砖引玉，重要的是用 vundle + github 的备份方案。所以，行动起来吧，备份你的 vimrc。

[1]: https://github.com/gmarik/vundle "Vundle 项目页"
[2]: https://github.com/chloerei/vimrc "我的 Vim 配置备份项目"
