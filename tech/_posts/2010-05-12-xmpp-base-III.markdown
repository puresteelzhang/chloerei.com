---
layout: post
title: XMPP 协议笔记 三 扩展
---

<h3>4. XMPP 扩展</h3>
XMPP 的核心部分是一个轻量级的协议，不足以满足一般即时聊天应用的需求。 XMPP 的社区在核心协议的基础上定义了众多的扩展协议，用以实现电子名片、二进制传输等功能。
<h4>4.1 通过 vcard-temp 获取电子名片</h4>
电子名片 (个人资料) 是聊天程序中常见的功能。通过电子名片，用户可以查看联系人除了 JID 地址外的其他信息，如：昵称，全名，个人网站 URL 等。

XMPP 的扩展之一 [<a href="http://xmpp.org/extensions/xep-0054.html">XEP-0054</a>] vcard-temp [9] 实现了电子名片。要在聊天程序中支持电子名片，需要客户端和服务端都实现 vcard-temp 扩展。知名的 XMPP 服务器大都支持 vcard-temp 扩展，如 jabber.org， gtalk 等。
<h5>4.1.1 请求电子名片</h5>
vcard-temp 扩展规定的电子名片访问规则如下：

客户端：
<pre lang="xml" escaped="true">&lt;iq from='stpeter@jabber.org/roundabout'
    id='v3'
    to='jer@jabber.org'
    type='get'&gt;
    &lt;vCard xmlns='vcard-temp'/&gt;
&lt;/iq&gt;</pre>
该查询使用 get 类型的 Iq 包请求 jabber.org 服务器返回 jer 用户的 vcard 数据。<!--more-->

接着 jabber.org 服务器返回 jer 用户的 vcard 数据：

服务器：
<pre lang="xml" escaped="true">&lt;iq from='jer@jabber.org'
    to='stpeter@jabber.org/roundabout'
    type='result'
    id='v3'&gt;
    &lt;vCard xmlns='vcard-temp'&gt;
        &lt;FN&gt;JeremieMiller&lt;/FN&gt;
        &lt;N&gt;
            &lt;GIVEN&gt;Jeremie&lt;/GIVEN&gt;
            &lt;FAMILY&gt;Miller&lt;/FAMILY&gt;
            &lt;MIDDLE/&gt;
        &lt;/N&gt;
        &lt;NICKNAME&gt;jer&lt;/NICKNAME&gt;
        &lt;EMAIL&gt;&lt;INTERNET/&gt;&lt;PREF/&gt;&lt;USERID&gt;jeremie@jabber.org&lt;/USERID&gt;&lt;/EMAIL&gt;
        &lt;JABBERID&gt;jer@jabber.org&lt;/JABBERID&gt;
    &lt;/vCard&gt;
&lt;/iq&gt;</pre>
vcard-temp 中定义了大量数据项，以上的示例只返回了用户已设置的部分数据项。
<h5>4.1.2 隐私问题</h5>
目前为止，vcard-temp 扩展被定义为无须经过授权访问，即非好友用户也可以获取任何一个 XMPP 用户的所有 vcard 信息。因此，vcard-temp 扩展建议用户慎重考虑是否要填入个人地址等隐私信息。
<h4>4.2 通过 In-Band Bytestreams 传输二进制数据</h4>
XMPP 核心部分没有提供二进制数据传输的支持，而 XMPP 扩展中有若干提供二进制传输的协议。应用较广的有 [<a href="http://xmpp.org/extensions/xep-0047.html">XEP-0047</a>] In-Band Bytestreams [10] 和 [<a href="http://xmpp.org/extensions/xep-0065.html">XEP-0065</a>] SOCKS5 Bytestreams [11]。由于实际上被广为应用二进制传输协议就只有这两种，所以客户端能做的选择不多。In-Band Bytestreams 和 SOCKS5 Bytestreams 可被整合为 [<a href="http://xmpp.org/extensions/xep-0096.html">XEP-0096</a>] SI File Transfer [12]，这个协议允许接收端选择传输方式。

此外还有 Google 提出的 Jingle 传输协议，Jingle 支持的网络连接方式更多，目前被应用于 Gtalk 客户端上，用于支持二进制文件传输与语音通信的实现，但并未普及。
<h5>4.2.1 In-Band Bytestreams 原理</h5>
所谓 In-Band ，即为 “在 XMPP 流内”的意思，其完整意义为：将二进制数据置入 XMPP 流内传输。但 XMPP 流的内容是纯文本，如何传输二进制数据呢？In-Band Bytestreams 的方案是把二进制数据使用 Base64 编码转换成可打印的纯文本，然后当作纯文本塞入 Iq 或者 Message 数据包中传输。
<h5>4.2.2 In-Band Bytestreams 示例</h5>
In-Band Bytestreams 的流程包括发起、接受/拒绝，分包传输，结束应答这几个过程，详细的流程示例参照 [XEP-0047] [10] 中的定义。

一个发送 In-Band Bytestreams 的 Iq 数据包示例如下：
<pre lang="xml" escaped="true">&lt;iq from='romeo@montague.net/orchard'
    id='kr91n475'
    to='juliet@capulet.com/balcony'
    type='set'&gt;
  &lt;data xmlns='http://jabber.org/protocol/ibb' seq='0' sid='i781hf64'&gt;
    qANQR1DBwU4DX7jmYZnncmUQB/9KuKBddzQH+tZ1ZywKK0yHKnq57kWq+RFtQdCJ
    WpdWpR0uQsuJe7+vh3NWn59/gTc5MDlX8dS9p0ovStmNcyLhxVgmqS8ZKhsblVeu
    IpQ0JgavABqibJolc3BKrVtVV1igKiX/N7Pi8RtY1K18toaMDhdEfhBRzO/XB0+P
    AQhYlRjNacGcslkhXqNjK5Va4tuOAPy2n1Q8UUrHbUd0g+xJ9Bm0G0LZXyvCWyKH
    kuNEHFQiLuCY6Iv0myq6iX6tjuHehZlFSh80b5BVV9tNLwNR5Eqz1klxMhoghJOA
  &lt;/data&gt;
&lt;/iq&gt;
</pre>
其中，data 段内包含的字符串是经过 Base64 编码的二进制数据。可见，该二进制数据经过编码后被当作一般纯文本来传输。data 段的 seq 属性标明了此段数据在所有数据分包中的编号。
<h5>4.2.3 In-Band Bytestreams 缺陷</h5>
In-Band Bytestreams 借助 XMPP 流进行二进制传输，优点是可以稳定交付，只要双方客户端仍然连接各自的服务器就能实现二进制传输。但其重大缺陷是速率过慢，每段二进制数据都要经过分包和多次封包解包，并且每个数据包都通过两个服务器的交付处理，会产生很大的延迟，效率很低。后文有对 In-Band Bytestreams 的简单速率测试。*未完成*
<h4>4.3 通过 SOCKS5 Bytestreams 传输二进制数据</h4>
4.3.1 SOCKS5 Bytestreams 原理

有别于 In-Band Bytestreams 流内传输，SOCKS5 Bytestreams 方案使用双方客户端的 socks5 直连传输，这样可以使传输速率达到双方主机直连的速度。

SOCKS5 Bytestreams 在传输过程的起效部分仅是让双方客户端决定传输方案，交换 IP 地址，端口地址等信息。当双方客户端决定可以使用的 IP 地址和端口地址后，就发起 socks5 连接，此时 XMPP 流不再参与数据传输，不增加额外的传输延迟。

整个 SOCKS5 Bytestreams (S5B) 流程如图三所示：
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/3-socks5-stream.jpg"><img class=" size-full wp-image-322 aligncenter" title="3 socks5-stream" src="http://chloerei.com/wp-content/uploads/2010/05/3-socks5-stream.jpg" alt="" width="349" height="375" /></a>图三 XMPP 发起 SOCKS5 Bytestreams 流程</p>

<h5>4.3.2 SOCKS5 Bytestreams 示例</h5>
一个发起 SOCKS5 Bytestreams 的 Iq 数据包示例如下：
<pre lang="xml" escaped="true">&lt;iq from='requester@example.com/foo'
    id='hu3vax16'
    to='target@example.org/bar'
    type='set'&gt;
  &lt;query xmlns='http://jabber.org/protocol/bytestreams'
         sid='vxf9n471bn46'&gt;
    &lt;streamhost
        jid='requester@example.com/foo'
        host='192.168.4.1'
        port='5086'/&gt;
  &lt;/query&gt;
&lt;/iq&gt;
</pre>
其中 streamhost 包含了发起人可用的 IP 地址，这个 IP 地址可以是主机地址，也可以是 socks5 代理主机地址 (用于突破防火墙和 NAT 网络的限制)。streamhost 项可以有多个，最终由接收方选择可用的地址，然后发起连接。
<h5>4.3.4 SOCKS5 Bytestreams 局限</h5>
由于互联网连接的复杂性，发起传输请求的 XMPP 客户端可能只拥有内网 IP ，并且没有可用的 socks5 代理服务器可用。这种情况下发起人将无法实现二进制文件的传输。

并且即使有代理服务器，由于经过中转，传输速率可能会大大降低。对此，除了假设更好的 socks5 代理服务器外，没有更好的方法。
<h4>4.4 扩展机制的缺点</h4>
XMPP 的扩展协议相当多，详细列表可以在 http://xmpp.org/extensions/ 找到。

但是通过扩展协议实现的功能，通常需要双方客户端均实现才能使用。有些还需要服务器做额外支持 (如 vcard-temp)，而有名的 XMPP 服务器通常只会选择性实现扩展协议，所以要在现实中实现所需的 XMPP 扩展，就需要搭建全套的服务端、客户端方案，开发成本大。并且造成各方 XMPP 服务器实现不一致，出现功能不同的 XMPP “部落”，只能使用核心协议中已定义的部分进行通信。

[9] http://xmpp.org/extensions/xep-0054.html

[10] http://xmpp.org/extensions/xep-0047.html

[11] http://xmpp.org/extensions/xep-0065.html

[12] http://xmpp.org/extensions/xep-0096.html
