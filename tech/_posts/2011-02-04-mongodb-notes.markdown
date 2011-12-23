---
layout: post
title: MongoDB 文档阅读笔记 —— 优雅的 NoSQL
---

NoSQL 数据库在上年炒得很热，于是我也萌生了使用 NoSQL 数据库写一个应用的想法。首先来认识一下 NoSQL。NoSQL 是一个缩写，含义从最初的 No-SQL 到现在已经成为了 Not-Only-SQL。确实后面一种解释比较符合 NoSQL 的使用场景。

现在网络上被人所知的 NoSQL 数据库可以在这个网页（<a href="http://nosql-database.org">http://nosql-database.org</a>）看到。这个列表林林总总一大堆，要选择哪个数据库入手呢？
<h3>1. 选择非关系数据库</h3>
在我关注的 Web 领域，特别是 Ruby on Rails 社区，比较多提到的是这几个数据库：
<ul>
	<li><a href="http://incubator.apache.org/cassandra/" target="_blank">Cassandra </a>， apache基金会下的非关系数据库。早前一段时间传言 Twitter 要用 Cassandra 替代 Mysql，一时间坊间流传“NoSQL 要革 SQL 的命了！”。不过 <a href="http://engineering.twitter.com/2010/07/cassandra-at-twitter-today.html">Twitter 博客澄清</a>，Twitter 只是在部分领域使用 Cassandra，存放 Tweets 的主数据库依然是 MySQL。</li>
	<li><a href="http://www.mongodb.org">MongoDB</a>，<a href="http://www.10gen.com/">10gen</a> 公司的开源非关系数据库产品，可以选择他们公司的商业支持。RoR 相关的插件挺多。</li>
	<li><a href="http://couchdb.apache.org/">CouchDB</a>，另一个apache基金会下的非关系数据库。</li>
	<li><a href="http://redis.io/" target="_blank">Redis</a>，特点是运行在内存中，速度很快。相比于用来持久化数据，也许更接近于 memcached 这样的缓存系统，或者用来实现任务队列。（比如<a href="http://github.com/defunkt/resque" target="_blank">resque</a>）</li>
</ul>
<a rel="attachment wp-att-807" href="http://chloerei.com/wp-content/uploads/2011/02/mongo-db-huge-logo.png"><img class="alignright size-medium wp-image-807" title="mongo-db-huge-logo" src="http://chloerei.com/wp-content/uploads/2011/02/mongo-db-huge-logo-300x99.png" alt="" width="300" height="99" /></a>
在这些候选名单中我选择了 MongoDB。因为它最近在 RoR 社区中的露脸率比较高，网页文档完善，并且项目主页的设计也不错 : P。

在陈述 MongoDB 的特性之前，还是给第一次接触 NoSQL 的人提个醒：不要意图用 NoSQL 全盘取代 SQL 数据库。非关系数据库的出现不是为了取代关系数据库。具体的说，MongoDB 并不支持复杂的事务，只支持少量的原子操作，所以不适用于“转帐”等对事务和一致性要求很高的场合。而 MongoDB 适合什么场合，请继续阅读。
<h3>2. 文档型数据库初探</h3>
关系数据库比如 MySQL，通常将不同的数据划分为一个个“表”，表的数据是按照“行”来储存的。而关系数据库的“关系”是指通过“外键”将表间或者表内的数据关联起来。比如 文章-评论 的一对多关系可以用这样的表来实现：
<pre lang="sql" escaped="true">posts(id, author_id, content, ... )
comments(id, name, email, web_site, content, post_id)</pre>
实现关联的关键就是 comments 表的最后一个 post_id 字段，将 comment 数据的 post_id 字段设为评论目标文章的 id 值，就可以用 SQL 语句进行相关查询（假设要查的文章 id 是 1）：
<pre lang="sql" escaped="true">SELECT * FROM comments WHERE post_id = 1;</pre>
相对于关系数据库的行式储存和查询，MongoDB 作为一个文档型数据库，可以支持更具层次感的数据。上面举的 文章-评论 结构，在 MongoDB 里面可以这样设计。
<pre lang="javascript" escaped="true">{
  _id : ObjectId(...),
  author : 'Rei',
  content : 'content text',
  comments : [ { name : 'Asuka'
                 email : '...',
                 web_site : '...',
                 content : 'comment text'} ]
}</pre>
comments 项是内嵌在 post 项中的（作为数组）。在 MongoDB 中，一个数据项叫做 Document，一个文档嵌入另一个文档（comment 嵌入 post）叫做 Embed，储存一系列文档的地方叫做 Collections。顺便一提，MongoDB 中也提供类似 SQL 数据库中的表间关联，叫做 Reference。
<h3>3. 用文档型数据库储存文档</h3>
可以看到，文档性数据库从储存的数据项上就跟 SQL 数据库不同。在 MongoDB 中，文档是以 <a href="http://bsonspec.org/">BSON</a> 格式（类似 JSON）储存的，可以支持丰富的层次的结构。由于数据结构的表达能力更强，用 MongoDB 储存文档型数据可以比 SQL 数据库更直观和高效。
<h4>3.1 简化模式设计</h4>
<a rel="attachment wp-att-808" href="http://chloerei.com/wp-content/uploads/2011/02/sql-smll.jpg"><img class="alignright size-medium wp-image-808" title="sql-smll" src="http://chloerei.com/wp-content/uploads/2011/02/sql-smll-300x178.jpg" alt="" width="300" height="178" /></a>在 SQL 数据库中，为表达数据的从属关系，通常要将表间关系分为 one-to-one，one-to-many，many-to-many 等模式进行设计，这通常会需要很多链接表的辅助。在 MongoDB 中，如果关联文档体积较小，固定不变，并且与另一文档是主从关系，那么通常可以嵌入（Embed）主文档。

常见情景：评论、投票点击数据、Tag。

这类场景的有时单个数据项体积很小，但是数量巨大，与之相应的是查询成本也会上升。如果将这些小数据嵌入所属文档，在查询主文档时一并提取，查询效率要比 SQL 高，后者通常需要开支较大的 JOIN 查询。并且根据文档介绍，每个文档包括 Embed 部分在物理硬盘上都是储存在同一区域的，IO 部分也会比 SQL 数据库快。（注，MongoDB 有单文档大小限制）
<h4>3.2 动态的文档模式</h4>
MongoDB 中的文档其实是没有模式的，不像 SQL 数据库那样在使用前强制定义一个表的每个字段。这样可以避免对空字段的无谓开销。

例如两个用户的联系信息，A 用户带有 email 不带 url，B 用户带有 url 不带 email。在 SQL 数据库中需要为两个数据项都提供 email 段和 url 段，而在 MongoDB 中可以这样保存：
<pre lang="javascript" escaped="true">[ { name : 'A', email : 'A email address' }, { :name : 'B', url : 'B url address' } ]</pre>
在关系数据库中，如果这些不确定的字段很多而且数量很大，为了优化考虑可能又要划分成两个表了，比如 users 和 profiles 两个表。在 MongoDB 中，如果这一类不确定信息确实是属于同一文档的，那么可以轻松的放在一起，因为并不需要预先定义模式，也不会有空字段的开销。

不过因为要支持这样的动态性，并且为了性能考虑进行预先分配硬盘空间，数据外的开销也会带来磁盘占用。我还未实测实际中开销有多大，也许未来不久我会写一个应用测试。
<h3>4. MongoDB 另外一些特点</h3>
<h4>4.1 JSON 文档式查询</h4>
MongoDB 的查询语言看起来是这样的：
<pre lang="javascript" escaped="true">&gt; db.users.find( { x : 3, y : "abc" } ).sort({x:1});</pre>
这是在 MongoDB 内置的 JavaScript 控制台上的查询，表示在名为 users 的 Collections 中查找 x = 3，y = "abc" 的文档，并且以 x 递增的顺序返回数据。

JSON 文档式查询会让写惯应用层代码的开发者眼前一亮，但对于精通 SQL 查询的关系数据库管理员来说就是一个新的挑战了。
<h4>4.2 对分布式的支持</h4>
MongoDB 对大型网站的最大吸引力也许来源于其对分布式部署的支持。当今互联网最流行的数据库 MySQL 在网站扩大到一定规模之后就会遇到扩展瓶颈，解决方案通常是分表分库、配置主从数据库。很多互联网开拓者前仆后继的为数据库扩展性奋斗，留下了一页页的宝贵经验。

既然年复一年的有人为同一个问题奋斗，为什么不将这些问题在数据库层面就解决了呢？而 MongoDB 的优势之一就是内建对分布式的支持。

<a rel="attachment wp-att-804" href="http://chloerei.com/wp-content/uploads/2011/02/sharding.png"><img class="aligncenter size-large wp-image-804" title="sharding" src="http://chloerei.com/wp-content/uploads/2011/02/sharding-500x291.png" alt="" width="500" height="291" /></a>不过我没有组建数据库群集的需求，所以还未阅读这方面的文档。
<h3>5. 总结</h3>
没有银弹，这个教诲已经提过太多。由于缺乏对事务的支持，MongoDB 不太适用于金融等行业的关键部分（我想也没有这个人群来看我博客吧）。但对于现在要应对海量细信息的 web 网站来说，MongoDB 可能恰好出现在了正确的时代。NoSQL 的无模式，能让网站开发的迭代更轻盈；MongoDB 对分布式的支持，可以缓解网站快速成长时在数据库端的瓶颈疼痛。

不要激进的用 NoSQL 替代所有 MySQL 应用，激进的 NoSQL 化只会像5年前的 all rewrite by RoR 浪潮一样，耗费不必要的精力。但在新项目开发之初，可以考虑是否更适合使用 NoSQL。而我，已经打算在下一个 web 项目里面使用 MongoDB。

多谢阅读 : D。
