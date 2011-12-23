---
layout: post
title: 为什么说 Ruby 比 Python 容易阅读
---

这是一篇充满偏见的语言战争文章，是一场无意义的锤子和锤子的比拼。语言的复杂性可以通过选择语言回避，而问题本身的复杂性是选择哪个语言都无法回避的。

不屑语言战争的人可以轻松无视此文。
<h3>1、字符串格式化</h3>
Python
<pre lang="python" escaped="true">"%s=%s" % (k, v)</pre>
在阅读 Python 字符串格式化的时候，视线先看到字符串的 %s 字样，但是不知道这指的是什么，然后看后面的变量 k，再接着看第二个 %s ，再看后面的 v ……视线必须不停地在字符串和变量之间跳动。
Ruby
<pre lang="ruby" escaped="true">"#{k}=#{v}"</pre>
而阅读 Ruby 字符串格式化的时候，看到需要变量的地方，变量就在那里。

顺便一说
<pre lang="ruby" escaped="true">"%s = %s" % [k,v]</pre>
这种风格的代码在 Ruby 里面也能用，Ruby 的理念认为解决问题的方法可以不止一种，选择哪种取决于程序员的喜好。
<h3>2、<span style="text-decoration: line-through;">映射（迭代）</span>（收回本节，详情看评论）</h3>

（内容有错误，收回发言）

<h3>3、DSL（领域语言）</h3>
为了举一个现实中有代表意义、但是又足够简单的例子，我找到了 <a href="http://webpy.org/">webpy</a> 和 <a href="http://www.sinatrarb.com/">sinatra</a>，这分别是 Python 和 Ruby 社区热门的简洁风格 web 框架。

前置的说明是，webpy，甚至是 Python，都不是一个追求 DSL 的社区。而 Ruby 社区则以 DSL 见长，这样比较似乎有失公允。但这里可以比较 DSL 的有无对于代码的可读性有什么帮助。

webpy 的 hello world
<pre lang="python" line="1" escaped="true">import web

urls = (
    '/', 'hello'
)
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'

if __name__ == "__main__":
    app.run()
</pre>
我对 webpy <a href="http://webpy.org/">原本的 helloworld</a> 做了简化，以便和 sinatra 比较。

坦率地说，webpy 的 hello world 已经够简洁了。相比起 Java EE 和 .net 庞大的 IDE 和那根本不知道拿来做什么的规范，webpy 让我们回归了单纯，简约而不简单。

但是，简约方面，Ruby 的 DSL 文化更是做到了极致，看 sinatra 的例子
<pre lang="ruby" line="1" escaped="true">require 'sinatra'

get '/' do
  "Hello World!"
end</pre>
伟大的人民艺术家苍井空老师<a href="http://t.sina.com.cn/1739928273/24F1wChv9">说过两个字</a>：

<a rel="attachment wp-att-740" href="http://chloerei.com/wp-content/uploads/2011/01/geiliable.jpg"><img class="aligncenter size-medium wp-image-740" title="geiliable" src="http://chloerei.com/wp-content/uploads/2011/01/geiliable-224x300.jpg" alt="" width="224" height="300" /></a>

sinatra 的 DSL 非常简练，甚至让人怀疑它是否是一个玩具。或者可以看下 sinatra 的<a href="http://www.sinatrarb.com/documentation">文档</a>或者<a href="http://www.sinatrarb.com/wild.html">用户列表</a>，现在请先暂且相信，它做的事跟 webpy 没什么两样。

DSL 是语言层面的封装，把复杂性留在库的内部，把接口用 DSL 的形式暴露给程序员。这其实跟类和函数方式的 API 没有什么不同。不过 DSL 会让人忘记自己正在使用什么语言，Rubyists 的说法是：魔法。
<h3>总结</h3>
Python 和 Ruby 虽然同为动态语言时代的佼佼者，不过开发和社区风格有很大的不同。这归根于两个语言诞生时的理念不同：Python 注重规范化，一个问题只有一个方法，缩进的强制约束，便于多人合作；而 Ruby 注重人性化，便于阅读，一个问题有几个方法，过多的魔法需要使用者自己锻炼驾驭能力。

这样的结果就是偏重数学，偏重模范化的人喜欢 Python；偏重乐趣，偏重人类语言化的人喜欢 Ruby。

你喜欢哪样呢？

* 本文部分代码来自《Dive into Python》和 webpy，sinatra 的文档
