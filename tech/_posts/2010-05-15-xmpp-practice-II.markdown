---
layout: post
title: XMPP 实践 二 程序架构
---

<h3>6. QTalk 客户端的设计</h3>
在此，将我编写的 xmpp 客户端命名为 QTalk ( Qt + Gtalk)。

(blog 特有内容： 开源了，放在 <a href="http://github.com/chloerei/qtalk">http://github.com/chloerei/qtalk</a>)

由于选择了 Gtalk 服务器作为服务端，qxmpp 库作为程序的 XMPP 的通信模块，所以 QTalk 编写的主要工作在于功能与用户界面的实现。

QTalk 的预期功能如下：
<ul>
	<li>GTalk 帐号登录</li>
	<li>列出联系人名单，获取联系人 VCard 资料、状态信息</li>
	<li>向已有联系人收发纯文本消息</li>
	<li>向已有联系人收发文件</li>
	<li>增删联系人</li>
	<li>人性化设置</li>
</ul>
QTalk 的运行截图见附录 一。
<h4>6.1 QTalk 结构</h4>
QTalk 的结构可分为通信模块和用户界面两大块，层次结构如图四所示：<!--more-->
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/4-qtalk-frame.jpg"><img class="aligncenter size-full wp-image-375" title="4 qtalk-frame" src="http://chloerei.com/wp-content/uploads/2010/05/4-qtalk-frame.jpg" alt="" width="514" height="298" /></a>图四 QTalk 层次结构</p>
通信模块的 QXmppClient 由 qxmpp 库提供，QXmppClient 一方面通过 XMPP 协议与 GTalk 服务器通信，一方面通过 qxmpp API 协助 MainWindow 收发 XMPP 数据包。

MainWindow 是用户界面的中心，管理了登录窗口，联系人列表，聊天窗口和文件传输等窗口部件。MainWindow 通过 qxmpp API 收发 XMPP 数据包，将数据包分类，更新到相应窗口部件上。
<h4>6.2 程序登录、获取联系人列表</h4>
QTalk 的登录流程为：用户输入账户密码，发送登录请求，获取保存在服务端的联系人列表。程序逻辑如图五所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/5-login-logic.jpg"><img class="aligncenter size-full wp-image-378" title="5 login-logic" src="http://chloerei.com/wp-content/uploads/2010/05/5-login-logic.jpg" alt="" width="433" height="358" /></a>图五 登录逻辑</p>
由于使用的是 GTalk 服务器，用户帐号可以通过注册 Gmail 获得。

联系人列表将会以列表视图的方式显示在 QTalk 主窗口中。
<h4>6.3 收发聊天信息</h4>
用户成功登录后，可以通过点击主窗口中的联系人列表，打开聊天窗口，对某个联系人发起对话。发送信息时，在聊天窗口输入的信息会被传递到 qxmpp 中进行发送；收到信息时，如果该联系人的聊天窗口已打开，则直接显示在聊天框内，否则会在系统托盘中提示收到了新消息。收发信息的逻辑如图六所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/6-message-logic.jpg"><img class="aligncenter size-full wp-image-384" title="6 message-logic" src="http://chloerei.com/wp-content/uploads/2010/05/6-message-logic.jpg" alt="" width="523" height="333" /></a>图六 收发信息逻辑</p>
qxmpp 库提供了良好的接口函数用于发送 Message 数据包。当收到服务端发来的 Message 数据包时，qxmpp 使用 Qt 的信号机制将数据包传递到 QTalk MainWindow 进行处理。
<h4>6.4 传输文件</h4>
已登录用户除了可以跟联系人发送纯文本信息外，还可以通过XMPP 的扩展协议 [XEP-0096] SI File Transfer 发送二进制文件 (如图片，PDF 文档)。

用户发送文件时，会首先发送文件传输的请求，联系人确认后开始进入传输，如果联系人拒绝，文件传输会中止。联系人向用户发送文件时也需要经过确认，否则不会开始传输。要求用户确认文件传输时，QTalk 会在系统托盘区和程序主界面显示请求提示。

qxmpp 库提供了 TransferManager 类用于管理文件传输。文件传输的逻辑如图七所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/7-transfer-logic.jpg"><img class="aligncenter size-full wp-image-388" title="7 transfer-logic" src="http://chloerei.com/wp-content/uploads/2010/05/7-transfer-logic.jpg" alt="" width="727" height="379" /></a>图七 文件传输逻辑</p>
Qtalk 提供了统一的文件传输窗口，可以用于查看传输进度和终端传输进程。
<h4>6.5 增删联系人</h4>
用户的联系人列表存放在 GTalk 服务器上，无论从什么客户端登录都可以得到一致的联系人列表。已登录用户可以对联系人列表进行操作，增删联系人。

增删联系人有两个层面，一个是把对方 JID 放入自己的联系人列表，一个是向联系人发送订阅请求。

根据 RFC3920 的定义，订阅，即订阅联系人的 Presence 状态，这个状态包括用户的在线、离线、忙碌、活跃等。

订阅类型分为4种：
<ul>
	<li>none : 双方均没有进行订阅</li>
	<li>to : 用户订阅了联系人的状态</li>
	<li>from: 联系人订阅了用户的状态</li>
	<li>both : 相当于 to + from</li>
</ul>
实际操作中，发现订阅状态会影响 Gtalk 服务器是否转发 Message 信息。当状态为 "to" 时，只能接收对方的聊天信息；当状态为 "from“ 时，只能发送聊天信息。只有当状态为 both 时，双方才能正常发起聊天会话。

联系人的订阅操作如图八所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/8-subscribe-logic.jpg"><img class="aligncenter size-full wp-image-392" title="8 subscribe-logic" src="http://chloerei.com/wp-content/uploads/2010/05/8-subscribe-logic.jpg" alt="" width="717" height="398" /></a>图八 联系人订阅逻辑</p>
接受请求时，在确认了对方的订阅请求后，qxmpp 库会自动发送订阅请求，以使订阅状态变为 "both"。
<h4>6.6 其他人性化设置</h4>
QTalk 实现了前述所有功能，并包括了一些方便用户使用的人性化功能，包括但不限于：
<ul>
	<li>联系人分组，更改联系人名称</li>
	<li>联系人 VCard 信息读取，头像显示，头像大小调整</li>
	<li>主窗口中隐藏离线联系人</li>
	<li>用户状态切换</li>
	<li>发送聊天信息的快捷键设置</li>
	<li>系统托盘图标、未读信息列表</li>
</ul>
QTalk 的运行截图在附录一。
