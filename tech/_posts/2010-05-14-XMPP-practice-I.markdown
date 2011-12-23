---
layout: post
title: XMPP 实践一 Qt 和 qxmpp 库
---

<h3>5. 选择用于编写 XMPP 客户端的工具</h3>
为了深入了解 XMPP 是否足够用于开发一个开放、功能丰富的即时聊天工具，我决定写一个 XMPP 的客户端聊天程序。服务端直接连接 Google Talk 服务器。客户端编程基于 C++/Qt 和 qxmpp 库。最终程序将可以使用 Gtalk 帐号登录，并与其他 Gtalk 用户通信。

用于编写 XMPP 客户端的工具如下：
<ul>
	<li>Google Talk 服务器</li>
	<li>Qt 编程框架和工具链</li>
	<li>qxmpp 程序库</li>
</ul>
<h4>5.1 Google Talk (Gtalk) 服务器的优势</h4>
Google Talk 是 Google 公司于2005年8月24日推出的一款IP 电话及即时通讯的服务。Google Talk 使用开放的 XMPP 协议。得益于 XMPP 的开放性，使用 Google Talk 服务不一定要通过官方客户端。Google Talk用户端仅支持 Windows (2000、XP、Server 2003、7)。[13]

实际上从 Gtalk 发布开始，XMPP 才真正走入大众的视野。以 Google 公司为后盾，Gtalk 的服务器拥有安全和稳定的口碑。

此次编程实践不涉及服务端的编程，而是遵循 XMPP 协议，让自己编写的客户端与 Gtalk 服务器通信，通过 Gtalk 服务器转发客户端的数据包。不涉及服务端编程，也可以使我专注与客户端的功能和操作性的优化上。
<h4>5.2 Qt 编程框架简介</h4>
Qt (发音同cute) 是一个跨平台的C++应用程序开发框架，有时又被称为 C++ 部件工具箱。Qt 被用在 KDE 桌面环境、Opera、OPIE、VoxOx、谷歌 Earth、Skype 和 VirtualBox的 开发中。它是诺基亚 (Nokia) 的 Qt Development Frameworks 部门的产品。[14]<!--more-->

Qt 的一个优势是跨平台，使用 Qt 写成的软件经过重新编译后可以运行在 Windows，Linux，Mac 平台上，实现了一次编写，随处编译。并且 Qt 拥有 LGPL 授权的版本，即可以用于编写开源软件，也可以编写闭源商业软件。

Qt 的本地语言是 C++，开发框架内包含了图形、网络、XML等多个方面的组件，使用 Qt 框架可以轻松编写跨平台的软件。
<h4>5.3 qxmpp 库简介</h4>
qxmpp 是一个基于 Qt 框架编写的 XMPP 通讯库。其开发者有 manjeetdahiya ，jeremy.laine，ian.geiser，项目源码[15]托管在 google code 上。

qxmpp 目前实现了 XMPP 标准中的核心部分 [RFC3920] [RFC3921]，以及 [XEP-0054] vcard-temp，[XEP-0096] SI File Transfer，必要时可以扩展功能。qxmpp 仅依赖 Qt 库，不需引入其他第3方的程序库，可使程序构建过程更清晰。

在实践初期，我曾尝试自己编写 XMPP 通讯部分的代码，但发现要遵循 XMPP 文档的定义重写一个通讯库是非常繁琐的事情。为了有效利用时间，着重研究 XMPP 协议在即时聊天上的应用效果，决定不重复发明轮子，选用了轻量级，并且依赖关系少的 qxmpp 库。

[13] http://zh.wikipedia.org/zh-cn/Google_Talk

[14] http://zh.wikipedia.org/zh-cn/Qt

[15] http://code.google.com/p/qxmpp/
