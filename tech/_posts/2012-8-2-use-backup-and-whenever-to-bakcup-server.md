---
layout: post
title: 用 Backup 和 Whenever 进行备份
---

最近将 [CodeCampo][1] 迁移了服务器，然后学习了 Backup 这个 Ruby gem，抛弃了过去蹩脚的备份脚本，实现了优雅的自动备份方案，跟大家分享一下。

### 备份脚本

首先登录服务器，安装 Backup。

    gem install bakcup

然后初始化备份脚本。

    backup generate:model --trigger codecampo --databases="mongodb" --storages="dropbox"

这时 Backup 会帮初始化备份脚本的目录，和我的第一个备份方案「codecampo」，目录结构如下

    Backup/
      models/
        codecampo.rb
      config.rb

config.rb 文件里面定义了一些通用配置，不过我用不到，先跳过。

`models/codecampo.rb` 就是我要编辑的备份方案，在初始化的时候我已经选择了生成 mongodb 的配置和储存端 dropbox 的配置，所以编辑完成后会是这样子的：

    Backup::Model.new(:codecampo, 'Description for codecampo') do
      split_into_chunks_of 250
    
      database MongoDB do |db|
        db.name = "code_campo"
      end
    
      store_with Dropbox do |db|
        db.api_key     = "***"
        db.api_secret  = "***"
        db.access_type = :app_folder
        db.path        = "/codecampo"
        db.keep        = 25
      end
    end

之前选用了 Dropbox 的储存方案，但是还没有配置 api_key，于是登录 [dropbox developer][2] 创建一个新应用。应用的资料可以随意，之后可以修改，不过应用名必需是唯一的。之后把面板的 api_key 和 api_secret 填到上面的备份脚本中。如果你还没有注册 Dropbox，可以通过这个邀请链接注册 http://db.tt/tdIsTDAm ，你和我都会额外增加 500M 空间。

备份脚本已经准备得差不多了，还缺最后一步，认证 dropbox 的应用授权。先来跑一遍备份脚本：

    backup perform --trigger codecampo

然后脚本会提示打开一个 dropbox 链接确认授权，现在用浏览器打开链接，并点击 **Allow**

![alt text][3]

通过后回到终端，根据提示回车，第一次备份就完成了！Dropbox 认证只需要一次，以后备份脚本会自动使用这次的认证 token 进行备份。

你可以打开 dropbox 的目录，看到你的应用目录下已经有一份备份存档。

### 定时任务

确认备份任务顺利跑通后，是时候让备份任务定时化，自动执行。定时任务我选择用 Whenever 帮助管理。

安装 Whenever

    gem install whenever

创建定时配置文件

    cd
    mkdir config
    wheneverize

然后打开 `config/schedule.rb` 文件，编辑为以下内容：

    every 1.day, :at => '18:00' do
      command "backup perform -t codecampo"
    end

接着执行 whenever，看看生成的 crontab 任务的格式是否正确

    $ whenever
    0 18 * * * /bin/bash -l -c 'backup perform -t codecampo'

这就是我想要的 crontab 任务，于是执行更新

    whenever --update-crontab

现在，备份脚本就会自动在 UTC 时间 18:00 跑了（+0800 时区的凌晨2点）。

### 总结

这个例子只备份了一个数据库 Mongodb，和使用了一个储存方案。如果需要更多类型的备份和储存方案，可以查阅 Backup 的官方文档 https://github.com/meskyanichi/backup 。

小提示：你可以编写不同的备份方案，然后用不同的定时任务执行，区分一些不同定时频率的备份。

用 Bakcup + Dropbox + Whenever 进行系统备份就是这么容易！


  [1]: http://codecampo.com
  [2]: https://www.dropbox.com/developers/apps
  [3]: http://i.minus.com/idsR61ESUFcL8.png
