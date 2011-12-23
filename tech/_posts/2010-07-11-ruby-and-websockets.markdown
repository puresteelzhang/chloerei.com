---
layout: post
title: ! 'Ruby & WebSockets: 浏览器的TCP'
---

原文：<a href="http://www.igvita.com/2009/12/22/ruby-websockets-tcp-for-the-browser/">http://www.igvita.com/2009/12/22/ruby-websockets-tcp-for-the-browser/</a>

<a rel="attachment wp-att-526" href="http://chloerei.com/2010/07/11/514/html5"><img class="alignleft size-full wp-image-526" title="html5" src="http://chloerei.com/wp-content/uploads/2010/07/html5.png" alt="" width="88" height="114" /></a>WebSockets是HTML5中最被低估的革新之一。不像本地储存（local storage），画布（canvas），并行操作（web workers）和视频播放（video playback），<a href="http://dev.w3.org/html5/websockets/">WebSocket API</a>的好处不会立即呈现给最终用户。事实上，过去十年我们已经发明了很多技术去解决浏览器和服务器间异步和双工通讯的问题：AJAX，<a href="http://www.igvita.com/2009/10/21/nginx-comet-low-latency-server-push/">Comet &amp; HTTP Streaming</a>，BOSH，<a href="http://www.igvita.com/2009/08/18/smart-clients-reversehttp-websockets/">ReverseHTTP</a>，<a href="http://www.igvita.com/2009/06/29/http-pubsub-webhooks-pubsubhubbub/">WebHooks &amp; PubSubHubbub</a>，还有Flash sockets等其他技术。话虽如此，上面列出的这些技术都有各自的弱点，并且没有解决根本问题：旧式的浏览器并不是为双向通信设计的。

HTML5中的WebSockets改变了这个状况，它从基础上设计了任意数据（二进制或文本）的双工通信。<strong>WebSockets是浏览器上的TCP</strong>，不像BOSH或者类似物，WebSockets只需要一个连接，这意味着对服务端和客户的更好的资源利用。并且，WebSockets适用于代理和防火墙环境，能通过SSL和HTTP通道完成传输——现有的均衡负载、代理和路由都能正常工作。
<h2>浏览器中的WebSockets：Chrome，Firefox和Safari</h2>
<a rel="attachment wp-att-527" href="http://chloerei.com/2010/07/11/514/websocket-browsers"><img class="alignleft size-full wp-image-527" title="websocket-browsers" src="http://chloerei.com/wp-content/uploads/2010/07/websocket-browsers.png" alt="" width="90" height="77" /></a>WebSocket API还只是草稿，不过主流浏览器的开发人员已经实现了大部分功能。Chrome从<a href="http://blog.chromium.org/2009/12/web-sockets-now-available-in-google.html">开发版本（4.0.249.0）</a>开始官方支持WebSocket API并且默认开启。Webkit每日构建版已经支持WebSockets，而Firefox有一个未决补丁正在复审。换句话说，要主流接受WebSocket还需时日，但作为开发者的我们可以开始思考WebSockets启用后的改良架构。一个最小的jQuery例子：<!--more-->

&gt; websocket.html
<pre lang="html4strict" escaped="true">&lt;html&gt;
  &lt;head&gt;
    &lt;script src='http://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.min.js'&gt;&lt;/script&gt;
    &lt;script&gt;
      $(document).ready(function(){
        function debug(str){ $("#debug").append("&lt;p&gt;"+str+"&lt;/p&gt;"); };

        ws = new WebSocket("ws://yourservice.com/websocket");
        ws.onmessage = function(evt) { $("#msg").append("&lt;p&gt;"+evt.data+"&lt;/p&gt;"); };
        ws.onclose = function() { debug("socket closed"); };
        ws.onopen = function() {
          debug("connected...");
          ws.send("hello server");
        };
      });
    &lt;/script&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;div id="debug"&gt;&lt;/div&gt;
    &lt;div id="msg"&gt;&lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;
</pre>
上面的例子展示了WebSockets的双向通信特性：send方法发送数据到服务端，onmessage回调在服务端发送数据到客户端的时候被掉用。不需要长轮询，HTTP头的额外支出，或者摆弄多链接。实际上，你可以现在就部署WebSocket API，不用等待浏览器支持，这需要用到Flash socket作为中间层：<a href="http://github.com/gimite/web-socket-js">web-socket-js</a>。
<h2>WebSocket客户端间的数据流</h2>
出于安全理由，WebSockets跟裸TCP sockets并不一样。尽管能从浏览器打开一个裸TCP连接看起来很诱人，但浏览器的安全会立即受到影响：任何网站可以利用用户的网络做别的事，并且获得用户方的安全上下文。例如，一个网站可以（利用用户）去连接一个远程SMTP服务器，然后发送spam——真是可怕。WebSockets扩展了HTTP协议，为浏览器发起连接定义了一个特别的握手过程。换句话说，这是一个选择协议，需要服务器方支持。

<a rel="attachment wp-att-517" href="http://chloerei.com/2010/07/11/514/websocket-chat"><img class="aligncenter size-full wp-image-517" title="websocket-chat" src="http://chloerei.com/wp-content/uploads/2010/07/websocket-chat.png" alt="" width="605" height="93" /></a>没有人阻止你连接SMTP，AMQP或者其他服务器，不过你需要引入一个WebSocket服务器作为连接的中介。<a href="http://www.kaazing.org/confluence/display/KAAZING/What+is+Kaazing+Open+Gateway">Kaazing Gateway</a>已经提供了STOMP和Apache ActiveMQ的适配器，你也可以为其他服务器实现你自己的JavaScript封装器。如果基于Java的WebSocket服务器不适合你，我们可以用Ruby EventMachine去构建一个简单的事件驱动型WebSocket服务器，只需要很少几行代码：

&gt; websocket.rb
<pre lang="ruby" escaped="true">require 'em-websocket'

EventMachine::WebSocket.start(:host =&gt; "0.0.0.0", :port =&gt; 8080) do |ws|
 ws.onopen    { ws.send "Hello Client!"}
 ws.onmessage { |msg| ws.send "Pong: #{msg}" }
 ws.onclose   { puts "WebSocket closed" }
end
</pre>
<a href="http://www.github.com/igrigorik/em-websocket/tree/master/.git">em-websocket  (Ruby EventMachine WebSocket Server)</a>
<h2>WebSocket服务的耗费</h2>
Chrome和Safari支持WebSockets意味着我们的移动设备将会支持服务端推送，这既节省了电量，又大大提高了带宽支出的有效性。并且，WebSockets也可以用在浏览器以外的场合（例如：实时数据流），这意味着一个符合规范的Ruby HTTP客户端也可以很好地操作WebSockets：

&gt; em-http-websocket.rb
<pre lang="ruby" escaped="true">require 'eventmachine'

EventMachine.run {
 http = EventMachine::HttpRequest.new("ws://yourservice.com/websocket").get :timeout =&gt; 0

 http.errback { puts "oops" }
 http.callback {
 puts "WebSocket connected!"
 http.send("Hello client")
 }

 http.stream { |msg|
 puts "Recieved: #{msg}"
 http.send "Pong: #{msg}"
 }
}</pre>
<a href="http://www.github.com/igrigorik/em-http-request/tree/master/.git">em-http-request  (Asynchronous HTTP Client)</a>

WebSocket在em-http-requrest中的支持还是实验性的，不过其目标是提供一个稳固并且完全透明的API：简单地指明WebSocket资源然后它会完成剩下的工作，就好像使用一个流式HTTP连接一样！更好的是，HTTP &amp; OAuth认证、代理和现存的均衡负载都能在新的传输模式下正常工作。
<h2>WebHooks，PubSubHubbub，WebSockets，...</h2>
当然，WebSockets不是解决所有问题的灵丹妙药。<a href="http://www.igvita.com/2009/06/29/http-pubsub-webhooks-pubsubhubbub/">WebHooks和PubSubHubbub</a>最适合于间断性的推送更新，而TCP长连接的却会导致低效。同样的，如果你对路由有特别的需求，<a href="http://www.igvita.com/2009/10/08/advanced-messaging-routing-with-amqp/">AMQP是更强大的工具</a>，或是基于其他原因在XMPP中<a href="http://www.igvita.com/2009/11/10/consuming-xmpp-pubsub-in-ruby/">重塑其强大的状态模型</a>。具体问题具体分析，不过WebSockets无疑是开发者工具箱一个必不可少的选项。
