---
layout: post
title: 如何向一个基于Qt的开源项目贡献翻译
---

我毫无疑问是一个菜鸟，不过我是比较有上进心的菜鸟，一直希望有朝一日能成为说中的牛人。菜鸟不会自然而然的成为牛人，这需要一个锻炼的过程，在哪里可以找到锻炼呢？那就是参与开源项目的开发。三人行，必有我师，何况是开源项目。能开发一个软件，并且这个软件能让我看到，那这个人几乎可以肯定水平比我高。所谓近朱者赤，往牛人身边靠，能提升自己的能力。

但是现实中的开源项目的代码质量不是我这种没有经过锻炼的菜鸟可以涉足的，这里就出现一个矛盾：要想提高我的能力，我必须参加一个开源项目；要参加一个开源项目，我必须有相应的能力。那难道我这种菜鸟就没有参加开源项目的方法了？好在，能向开源项目贡献的并不只有代码，还有别的方面，比如bug报告、美工、翻译。经过我探索发现，其实参与一个开源项目门槛最低的是参与翻译。而且收获远远不止英语单词的积累，还可以学到以下内容：
<ul>
	<li>与开发者交流的礼义</li>
	<li>观摩一个开源软件开发的流程</li>
	<li>从易懂的角度阅读开源代码</li>
</ul>
我最近厚着脸皮，参加了 <a href="http://qt.gitorious.org/qt-creator/qt-creator-zh_cn">qtcreator 中文版</a>的翻译，这让我学到了不少东西，远胜于自己闭门造车。

于是，我想记录一下我第一次参与开源项目翻译的心得，也为跟我一样想参与开源项目的人提供借鉴。
<h3>1. 使用 Qt Lingust</h3>
<em>该部分仅适用于基于 Qt 开发的项目</em>

Qt 是一个开源的应用程序开发框架(<a href="http://qt.nokia.com/title-cn">官网</a>)，基于 Qt 的开源软件有 KDE SC，SMplayer等。

Qt 为所有基于 Qt 开发的软件提供了一个翻译的解决方案，这包含了针对项目管理者，开发者和翻译人员的对应工具。这里只说翻译人员要用的工具 Qt Lingust。
<p style="text-align: center;"><a name="Qt-Linguist-run" href="http://chloerei.com/wp-content/uploads/2010/05/Qt-Linguist.jpg"><img class="size-full wp-image-186 aligncenter" title="Qt-Linguist" src="http://chloerei.com/wp-content/uploads/2010/05/Qt-Linguist.jpg" alt="" width="492" height="266" /></a>Qt Lingust运行图</p>
Qt Lingust 是一个图形界面的应用程序，翻译人员使用的时候并不需要拥有编程知识。(当然为了翻译得更准确，Qt 字符串里的 % 标记还是要会看的)。<!--more-->

它的使用方法如下：
<h4>1.1 安装</h4>
到官网的 <a href="http://qt.nokia.com/downloads-cn">这个页面</a> 下载自己相应平台的 SDK 包。比如使用的是 Windows 操作系统，则下载 “Qt SDK for Windows”。下载完成后双击安装包安装，安装过程与一般的 Windows 程序无异。

Linux 平台用户也可以从发行版的软件源中用包管理安装。据我经验，Archlinux 的版本教新，与官方稳定版一致。而 Ubuntu 则落后官方几个版本，想要用最新的 Qt Lingust ，也可以选择用 SDK 包的方式安装。
<h4>1.2 读取翻译源文件并录入翻译</h4>
安装完成后，运行 Qt Lingust，然后打开要翻译的 ts 格式翻译源文件(这个文件怎么来在<a href="#section2">第2部分</a>说)。
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/Qt-Linguist-2.jpg"><img class="size-full wp-image-190 aligncenter" title="Qt-Linguist-2" src="http://chloerei.com/wp-content/uploads/2010/05/Qt-Linguist-2.jpg" alt="" width="313" height="204" /></a>载入要翻译的 ts 文件</p>
载入 ts 文件后，Qt Lingust 的界面就如早前看到的<a href="#Qt-Linguist-run">运行截图</a>一样了。

Qt Lingust 的 Help -&gt; Manual 里面有完整的帮助手册。这里讲一些基本的知识和常用的操作。
<h5>1.2.1 Qt Linguts 的界面有什么？</h5>
Qt Linguts 的界面所有的模块和功能如图所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/Qt-Lingust-comment.jpg"><img class="size-full wp-image-193 aligncenter" title="Qt-Lingust-comment" src="http://chloerei.com/wp-content/uploads/2010/05/Qt-Lingust-comment.jpg" alt="" width="494" height="270" /></a>区域注释(点击大图)</p>

<ul>
	<li>Context 将翻译词条按程序中的类分组。</li>
	<li>Strings 显示某个组中要翻译的词条。</li>
	<li>Sources and Forms 显示词条在源代码中的位置，如果位置处于界面文件上，还可以直接绘制翻译后的部件。</li>
	<li>Phrases and guesses 翻译提示。可以显示以往翻译的相近词条和程序提供的翻译猜测。</li>
	<li>中心区域 就是翻译工作的主区域了。在 Chinese Translation 栏填入 Source Text 的对应翻译</li>
</ul>
<h5>1.2.2 工作流程</h5>
打开 ts 文件 &gt; 选择要翻译的词条组和要翻译的词条(显示为问号的词条为需要翻译的词条) &gt; 在主区域输入改词条的翻译 &gt; 点击词条前的问号，使其变为绿色的勾。

要注意的是右下角的 Waring 框，里面显示词条翻译有无通过相应的校验(比如标点和变量的数量)，如果没有经过校验，词条是不会变成绿色勾勾的。这时候就要修正翻译，具体的参考软件手册中的说明。
<h5>1.2.3 常用快捷键</h5>
Ctrl + Enter : 当前翻译完成，进入下一条未决翻译
Ctrl + J : 进入下一条未决翻译
Ctrl + Shift + J : 下一条翻译(无论状态)
Ctrl + S : 保存

更多的快捷键参阅软件手册。

至此，Qt Linguts 的简单使用已经介绍完毕。可见，参与翻译工作并不需要任何编程知识，像我这样的菜鸟都能完成。并且Qt Linguts 右侧的 Sources and Forms 提供了源码的浏览！这成为我阅读 Qt creator 源码的一个入口。
<h3>2 与项目管理者联系，获取 ts 文件<a name="section2"></a></h3>
前面一直说的 ts 文件是什么呢？这个是要翻译词条的源文件(我理解为 translate source)，一般由项目管理者生成。向项目管理者请求加入翻译组，说明要翻译的语言。管理者在同意请求后就会用某种方式发送 ts 到翻译者手中，如果是单人翻译，用 Email 就行了，如果是多人同时翻译，可能需要翻译者懂得用 Git 这样的源码管理工具来签出 ts 文件。
<h4>2.1 与项目人员联络的方法。</h4>
要提交翻译请求，就要联络到项目人员，那么不外乎下面的方法：

<a href="http://chloerei.com/wp-content/uploads/2010/05/MailingList_3831200.jpg"><img class="size-full wp-image-211 alignnone" title="Net Meeting" src="http://chloerei.com/wp-content/uploads/2010/05/MailingList_3831200.jpg" alt="" width="230" height="173" /></a>
<h5>2.1.1 项目网站留言</h5>
一些小型的项目，甚至是个人项目，可能就是把程序发布在博客上。这些项目的翻译词条比较少，翻译起来比较轻松。如果要联络项目人员，只要在项目网站上留言或者直接发邮件到作者邮箱就可以了。
<h5>2.1.2 邮件列表 mail-list</h5>
颇具规模的项目，项目人员一般都使用邮件列表来和用户和贡献者沟通。查看项目主页上有无邮件列表的链接，按照说明订阅该项目的邮件列表，然后在邮件列表中提问是否可以为翻译做贡献。
<h5>2.1.3 其他渠道</h5>
一般而言国外的项目都会使用邮件列表来沟通，如果实在难以适应邮件列表的通讯方式，这时候就要多翻阅项目主页上的介绍，留意 developer 之类的链接。如果项目规模较少，也许在项目的论坛中提问也会有人答复。但是大型项目的开发人员一般没足够时间从论坛中获得反馈。(email是很重要的)
<h4>2.2 阅读翻译须知</h4>
在开始翻译前，一定要留意有没有翻译须知，有的话一定要仔细阅读。

比如 Qt 的翻译须知就规定了，不得翻译关于软件授权的信息。

此外，如果是多人共同翻译，需要熟悉翻译组内是否有一些惯用语的翻译约定，以免出现翻译不一致的情况。
<h3>3. 多人协作翻译的心得</h3>
当一个软件的翻译词条多达几千条 (qtcreator 有4000条) 的时候，单靠一个人进行翻译是不现实的。而多人协作翻译有一些要点要注意。

<a href="http://chloerei.com/wp-content/uploads/2010/05/octocat.png"><img class="alignright size-full wp-image-208" title="octocat" src="http://chloerei.com/wp-content/uploads/2010/05/octocat.png" alt="" width="200" height="200" /></a>
<h4>3.1 版本管理器的使用</h4>
多人编程，源码的合并是通过版本管理器进行。而多人翻译，由于 ts 文件是一个 xml 的纯文本，所以一般也通过版本管理器来管理。版本管理器可以对纯文本进行合并，冲突解决，版本回退等操作，实际上在多人协作上，没有版本管理器是根本无法操作的。常见的版本管理器有 svn，git等，版本管理器的使用要参阅该软件的手册说明。一般而言，使用版本管理器需要一些命令行的知识。至于选择哪个版本管理器，就要看项目本身使用什么版本管理器。向 qtcreator 的翻译提交是需要用 git merge 的，所以翻译项目也理所当然用 git 进行管理。
<h4>3.2 翻译任务的分配</h4>
即使有了版本管理器对 ts 文本进行管理，也不是合并操作就能一帆风顺了。实际上如果 ts 文件的翻译合并真的产生了冲突(比如有两个译者的翻译中有几十条词条重合了)，要合并会非常麻烦。一般的源代码文件不过2、3千行，而目前 qtcreator ts 文件的行数是 25030！要在这么大的文件内处理合并是需要非常谨慎的工作。

所以，最好的处理方法是：<strong>不要冲突</strong>。

为了让译者的劳动成果不产生冲突，就需要在开始翻译前给译者分配工作区域。

对于 Qt Linguts，我发现到 Context 列表中的项目是可以按照首字母顺序排序的，所以我提议按以下方案分配翻译任务：
<blockquote>a. 将 Context 列表按首字母排序
b. 按照首字母分成 A ~ Z 区域
c. 向译者分配某一首字母区域的翻译任务。任务可以滚动分配，或者按照译者对该段词条的熟悉程度调整。</blockquote>
xtfllbl 兄赞同我的建议，并用到了 qtcreator 的翻译过程中。
<h3>4. 结语</h3>
翻译是进入开源社区的一个很好的切入点。门槛低，但是实际上对开源软件的普及有很大贡献。

国内大量流通着一些低质量，但是中文资料多的软件或者工具，比如 MFC。英文确实对入门用户有很大门槛。为了推广开源软件、自由软件，有能力的人应该多参与开源软件的翻译工作。既锻炼了自己团队合作的能力，又为开源软件做了贡献，何乐而不为？
