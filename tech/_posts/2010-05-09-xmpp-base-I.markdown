---
layout: post
title: XMPP 协议笔记 一 基础篇
---

<h3>1. XMPP 优缺点</h3>
XMPP (Extensible Messaging and Presence Protocol) (前称Jabber) 是一种以 XML 为基础的开放式即时通讯协议，是经由互联网工程工作小组 (IETF) 通过的互联网标准。[1]
<h4>1.1 XMPP 协议的优点</h4>
<h5>1.1.1 可扩展性</h5>
XMPP 的数据传输基于 XML 格式，可扩展性强。XMPP 的核心协议栈 (Core Stack) 部分只定义了基础的 Presence，Message，Iq 等最主要数据格式和传输逻辑，更多的功能则通过定义扩展 (Extensions) 实现。
<h5>1.1.2 受 IETF 组织规范</h5>
Internet Engineering Task Force  (IETF) 在2002年开始规范 XMPP 协议，使其协议的修订和扩展的添加都经过严格的流程审核，防止 XMPP 协议因缺乏标准而分裂。并且这也保证了 XMPP 协议是完全开放的。
<h5>1.1.3 应用广泛</h5>
XMPP 协议的应用比其他开放即时通讯协议更为广泛。较有名的使用 XMPP 协议的聊天服务有 Google Gtalk 和 Facebook Chat 等。此外，XMPP 在各平台下都有若干服务端、客户端和程序库的实现，二次开发时成本较低。

XMPP 协议的可扩展性和开放性是该协议被广泛应用的保证。<!--more-->
<h4>1.2 XMPP 协议的缺点</h4>
<h5>1.2.1 不内置支持二进制数据的传输</h5>
XMPP 的核心部分没有包含对二进制数据传输的支持，这使得 XMPP 的基本数据限定在文本文件范围内。XMPP 社区认为，XMPP 应该用于传输 meta 信息，辅助其他应用进行协议握手，XMPP 本身不应负担海量信息的传输。

从当前流行的轻量化观点来看，XMPP 把二进制数据传输的协议移入核心栈，是符合了最小核心的需求。但同时却为实际应用中 XMPP 客户端传输二进制数据增加了开发扩展协议的负担。
<h5>1.2.2 缺乏旗舰应用</h5>
XMPP 是开放的，任何个人和组织都可以使用 XMPP。但同时产生的副作用是每个组织使用 XMPP 的目的不同，侧重点不同，导致XMPP 所开发的应用实际上导致了各个厂商各自为政，比如 Cisco 将 XMPP 用于设备通信，游戏厂商用于游戏内的简易聊天。即时通讯中只有 Google Gtalk 和 Fackbook Chat 较出名，但都没有作为这两家企业的核心产品作为推广。 XMPP 的应用中并没有旗舰应用。

XMPP 的缺点归根结底是因为其已经成为开放标准，制定和修改要顾及多方的利益。其核心栈只能包括各种应用的交集部分。各厂商对 XMPP 的利用多会建立一套新的扩展协议以扩展功能，如 Google 用于文件和语音流传输的 Jingle 协议和完全和其他 XMPP 应用不流通的 Google Wave。

但总的来说，XMPP 的问题是一个开放即时通讯协议不可避免遇到的问题。研究 XMPP 协议的实用效果，对将来开放更好的开放协议有重要的参考意义。
<h3>2. XMPP 基础</h3>
<h4>2.1 网络层次和数据包</h4>
XMPP 使用 TCP 连接，并支持安全传输 (TSL/SASL)。

XMPP 的层次结构如下

XMPP
SASL
TSL
TCP

XMPP 层中传输的数据包采用 XML 格式，称为 XML stanzas，XMPP 节点间 XML stanzas 的传输构成的数据流称为 XML stream。

一个 XML stream 的概览[2]如下
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/xmpp-stream.jpg"><img class="size-full wp-image-245 aligncenter" title="xmpp-stream" src="http://chloerei.com/wp-content/uploads/2010/05/xmpp-stream.jpg" alt="" width="211" height="355" /></a>图一 XML Stream</p>
XMPP 核心栈中，XML stanzas 包括 Presence、Message 和 Iq stanzas。Presence 用于传输节点状态，Message 用于传输信息内容，而 Iq 用于传输更复杂的应答。实际应用中，Presence 多限定用于简单的状态传输，而扩展协议多通过扩展 Iq stanzas 元素实现。
<h4>2.2 XMPP 的节点与路由</h4>
XMPP 中的节点大致有两种，一种是服务器，一种是客户端(暂不讨论各种代理)。客户端需要连接服务器，服务器为客户端提供数据包 (XML stanzas) 的路由和转发。
<p style="text-align: center;"><a href="http://chloerei.com/wp-content/uploads/2010/05/2-xmpp-node.png"><img class="size-full wp-image-254 aligncenter" title="2 xmpp-node" src="http://chloerei.com/wp-content/uploads/2010/05/2-xmpp-node.png" alt="" width="500" height="188" /></a>图二 XMPP Server-Client 节点</p>
客户端与客户端的通信需要通过服务器中转。每个到达服务器的数据包，由服务器分析后发往别的服务器或客户端。与 Email 的通信机制不同的是，XMPP 服务器之间不设立中转点，而是直接连接，以提高即时性和安全性。
<h4>2.3 地址标识</h4>
每个客户端需要拥有一个地址标识用于定位，XMPP 中称之为 JID (Jabber ID)。地址标识的格式如下
<p style="text-align: center;"><code>[ node "@" ] domain [ "/" resource ]</code></p>
例如
<p style="text-align: center;"><code>username@gmail.com/pidgin</code></p>
格式与 Email 地址格式类似，但增添了 resource 项，用于支持同一账号的多客户端登录。上述例子可以解释为：在 gmail.com 服务器注册的 username 用户，且使用 pidgin 客户端登录。当一个 JID 不包含 resource 部分时，该 JID 一般称为 BareJID。

用户地址标识的认证由提供 XMPP 服务的服务器执行。例如，注册于 gmail 服务器的账号由 gmail 服务器进行验证。其他服务器发往 gmail.com 域名的数据包均通过域名查询与服务间验证后发往 gmail 服务器，而不用考虑 gmail 服务器与下属账号间的通信。

[1] http://zh.wikipedia.org/zh-cn/XMPP

[2] http://tools.ietf.org/html/rfc3920#page-9
