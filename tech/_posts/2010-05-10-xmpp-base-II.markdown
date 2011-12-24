---
layout: post
title: XMPP 协议笔记 二 XML数据包
---

<h3>3. XMPP 核心数据包</h3>
XMPP 的核心数据包类型有Precense，Message，Iq ，此外加上初始化 stream 用到的 Stream 数据包。这些数据包是 XMPP 信息传输的载体，被用于 XMPP 核心功能和扩展功能的实现。

该部分仅对 XMPP 中使用的数据包进行概览，用于感受基于 XML 的数据包与其他非 XML 数据包协议的差别，不能替代 IETF 关于 XMPP 协议的 RFC 文档 [<a href="http://tools.ietf.org/html/rfc3920">3920</a>] [<a href="http://tools.ietf.org/html/rfc3921">3921</a>][3][4]，以及 XMPP 的扩展协议文档 [<a href="http://xmpp.org/extensions/">extensions</a>][5] 中描述。
<h4>3.1 公有属性</h4>
在 XML stream 中，每个数据包都是 XML 格式纯文本。而每个 XML 数据包有以下公有属性：
<ul>
	<li>to 数据包要发送的目的地址</li>
	<li>from 数据包发送的源地址</li>
	<li>id 数据包标示符</li>
</ul>
此三项属性在 XML stanza 中最为常见。

to 和 from 属性用于服务器决定该数据包的路由规则。某些情况下，to 和 from 属性可以只有一个，例如：客户端向服务端发送设置配置的 Iq 包只含有 to (不向外路由)，客户端向联系人发送 Message 只含有 to (from 属性总是被改写为客户端的地址)。

id 用于节点间判断请求和应答数据包的对应状况，大多数情况可以不处理。<!--more-->
<h4>3.2 初始化 XML stream，身份验证</h4>
在客户端与服务器产生 TCP 连接后，需要与服务器初始化 XML stream，以及进行身份验证。

初始化时，客户端发送 stream 头部 XML：
<pre lang="xml" escaped="true">&lt;?xml version='1.0'?&gt;
&lt;stream:stream
    to='example.com'
    xmlns='jabber:client'
    xmlns:stream='http://etherx.jabber.org/streams'
    version='1.0'&gt;
</pre>
服务器在收到客户端的 stream 头后，回应一个 stream 头：
<pre lang="xml" escaped="true">&lt;?xml version='1.0'?&gt;
&lt;stream:stream
    from='example.com'
    id='someid'
    xmlns='jabber:client'
    xmlns:stream='http://etherx.jabber.org/streams'
    version='1.0'&gt;</pre>
接着服务器向客户端发送服务端支持的身份验证方式列表，常见的方式有基于安全传输 SASL 的 BASE64 编码账户密码验证。身份验证的种类多样，且过程较为繁琐，可以参考 《XMPP: The Definitive Guide》<a href="http://books.google.com/books?id=SG3jayrd41cC&amp;lpg=PP1&amp;pg=PA165#v=onepage&amp;q&amp;f=false">第12章</a>的介绍。

在对话结束时，客户端和服务端要先后发送 stream 尾部 XML，以使整个 XMP stream 闭合。(如果 TCP 异常中断，则服务端直接中断对话)

客户端：
<pre lang="xml" escaped="true">&lt;/stream:stream&gt;</pre>
服务端：
<pre lang="xml" escaped="true">&lt;/stream:stream&gt;</pre>
<h4>3.3 Roster 获取联系人列表</h4>
在即时聊天 (IM) 应用中，客户端登录服务器后做的第一个操作通常是获取联系人列表。获取联系人列表需要发送 get 类型的 Iq 数据包。(Iq数据包将会在3.6节解释)

客户端：
<pre lang="xml" escaped="true">&lt;iq from='juliet@example.com/balcony' type='get' id='roster_1'&gt;
    &lt;query xmlns='jabber:iq:roster'/&gt;
&lt;/iq&gt;</pre>
该请求的意义为：名为 juliet 的用户 (登录资源为 balcony) 向 example.com 服务器请求获得 (get) roster 表。

服务器收到请求后，返回 roster 表。

服务端：
<pre lang="xml" escaped="true">&lt;iq to='juliet@example.com/balcony' type='result' id='roster_1'&gt;
    &lt;query xmlns='jabber:iq:roster'&gt;
        &lt;item jid='romeo@example.net'
            name='Romeo'
            subscription='both'&gt;
            &lt;group&gt;Friends&lt;/group&gt;
            &lt;/item&gt;
        &lt;item jid='mercutio@example.org'
            name='Mercutio'
            subscription='from'&gt;
            &lt;group&gt;Friends&lt;/group&gt;
            &lt;/item&gt;
        &lt;item jid='benvolio@example.org'
            name='Benvolio'
            subscription='both'&gt;
            &lt;group&gt;Friends&lt;/group&gt;
        &lt;/item&gt;
    &lt;/query&gt;
&lt;/iq&gt;</pre>
可以看到，juliet 的 roster 表内有3个联系人，分别名为 Romeo，Mercutio，Benvolio，都属于 Friends 分组。Roster 列表中的 JID 信息将会用在稍候客户端发送信息包的目的地址中。

Item 中的 subscription 关系到联系人状态信息的传输，有 none，both，from，to 四种。详细的 subscription 操作在 RFC 3921 <a href="http://tools.ietf.org/html/rfc3921#page-26">Managing Subscriptions</a> 章节[7]中定义。
<h4>3.4 Presence 状态数据包</h4>
Presence 数据包被设计用来发送轻量级的节点状态信息，例如 IM 中用来发送用户的在线、离开、忙碌、离线等简单的状态消息。

一个 presence 数据包范例如下：

客户端：
<pre lang="xml" escaped="true">&lt;presence&gt;
    &lt;show&gt;away&lt;/show&gt;
&lt;/presence&gt;</pre>
该 presence 向服务端表明，客户端用户处于离线状态。服务端收到 presence 后，会自行填充 from 和 to 属性，发送到订阅 (subscribeb) 了该用户状态信息的联系人服务器上。

Presence 数据包也可以在发送时填上 to 属性，用于指定 presence 的接收方。例如：

客户端：
<pre lang="xml" escaped="true">&lt;presence to="romeo@example.net"&gt;
    &lt;show&gt;away&lt;/show&gt;
&lt;/presence&gt;</pre>
该 Presence 只会被转发至 romeo@example.net。
<h4>3.5 Message 信息数据包</h4>
Message 是即时聊天应用中最常用的数据包，其功能是发送用户聊天信息。一个 Message 例子如下：

客户端：
<pre lang="xml" escaped="true">&lt;message
    to='romeo@example.net'
    from='juliet@example.com/balcony'
    type='chat'
    xml:lang='en'&gt;
    &lt;body&gt;Wherefore art thou, Romeo?&lt;/body&gt;
&lt;/message&gt;</pre>
该 message 包将会被服务器转发至 example.net 服务器，随后转交给 romeo 已登录的客户端上(如果该用户没有登录，message 信息会储存在服务端直至用户上线)。

其中，body 标签中包含用户要传输的聊天信息。

要传输格式化的富文本信息，可以通过支持扩展 <a href="http://xmpp.org/extensions/xep-0071.html">XEP-0071</a>[8]，引入 html 标签。

3.6 Iq 数据包

Iq 是一个为其他操作提供 get，result，set，error 动作的数据包，本身并没有限定用途的范围。核心协议中就有大量需要用到 Iq 数据包的操作，例如添加 roster 联系人，需要用到 set 类型的 Iq 数据包：

客户端：
<pre lang="xml" escaped="true">&lt;iq from='juliet@example.com/balcony' type='set' id='roster_2'&gt;
    &lt;query xmlns='jabber:iq:roster'&gt;
        &lt;item jid='nurse@example.com'
        name='Nurse'&gt;
        &lt;group&gt;Servants&lt;/group&gt;
        &lt;/item&gt;
    &lt;/query&gt;
&lt;/iq&gt;</pre>
服务器用 result 类型的 Iq 数据包返回操作结果：

服务端：
<pre lang="xml" escaped="true">&lt;iq to='juliet@example.com/balcony' type='result' id='roster_2'/&gt;</pre>
XMPP 扩展协议大都通过扩展 Iq 数据包的元素来添加操作。Iq 数据包的 type 属性就像 HTTP 协议中的 Method 项一样，提供增删改查 (CURD) 动作，具体实现何种操作并不限制。

[3] http://tools.ietf.org/html/rfc3920

[4] http://tools.ietf.org/html/rfc3921

[5] http://xmpp.org/extensions/

[6] http://books.google.com/books?id=SG3jayrd41cC&amp;lpg=PP1&amp;pg=PA165#v=onepage&amp;q&amp;f=false

[7] http://tools.ietf.org/html/rfc3921#page-26

[8] http://xmpp.org/extensions/xep-0071.html
