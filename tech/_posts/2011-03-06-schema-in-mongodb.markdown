---
layout: post
title: 文档型数据库的数据模式
---

[文章首发在 <a href="http://codecampo.com/topics/4d73354c9f328b6eda000024">codecompo</a> ，那边编辑器比较好用 ^ ^。欢迎订阅我在 codecampo 的<a href="http://codecampo.com/~Rei/topics.rss">feed</a>]

campo 项目一个有意思的地方是使用 mongodb 数据库。跟 mysql 相比，mongodb 数据库模式方面的文章还比较少，且大多是英文。这里列出一些 campo 项目使用中的数据库模式，也作为一点心得记录。
<h2>1. Tag</h2>
Tag 可以说是最适合文档型数据库数据之一了。

模式
<pre><code>&gt; db.topics.findOne()
{
    _id : ObjectID(...),
    content : "Topic Content ...",
    tags: ['one', 'two', 'three']
}
&gt; db.topics.ensureIndex({tags: 1})
</code></pre>
查询
<pre><code>&gt; db.topics.findOne({tags : 'one'})
{
    _id : ObjectID(...),
    content : "Topic Content ...",
    tags: ['one', 'two', 'three']
}

&gt; db.topics.findOne({tags : {$in : ['one', 'four']}})
{
    _id : ObjectID(...),
    content : "Topic Content ...",
    tags: ['one', 'two', 'three']
}
&gt; db.topics.findOne({tags : {$all : ['one', 'four']}})
null
</code></pre>
<h2>2. 马克</h2>
马克有三种方案
<ol>
	<li>topic 保存 user_ids</li>
	<li>user 保存 topic_ids</li>
	<li>1、2点都用</li>
</ol>
分析

因为用户每打开一篇文章，需要知道该用户有没有 mark 过该文章，所以需要随着文章或者用户把 mark 的数据读取出来，这种需求适合用 embed。如果用户很多，mark 数据很大(&gt; 4M)怎么办？这时候即使单独用一个数据集也解决不了，目前不考虑。

问题是将关联 ID 存放在 topic 端，还是 user 端，或者两边都放。这可能取决于应用。对 codecampo  的需求来说，每篇文章的marker一般很少或没有，可能出现个别文章有大量 marker。而用户 mark  的文章数不一，但基本确定不会有海量数据，如果有，可以认为该用户是个机器人。

按上面分析，似乎应该选择将关联数据放到 user 端，但实际上 campo 选择了放在 topic 端。这是出于两点考虑：
<ol>
	<li>希望关联数据随着文章的下沉而下沉，不需要用户看每一篇文章都将无用的关联数据读出</li>
	<li>文章删除时，关联数据也一同删除。（相对的，用户一般只封禁，不删除）</li>
</ol>
至于两端都放置关联数据的做法，我觉得过于冗余，难于管理，不建议。

模式
<pre><code>&gt; db.topics.findOne()
{
    _id : ObjectID(...),
    content : "Topic Content ...",
    marker_ids : [_id : ObjectID(...), _id : ObjectID(...)]
}
&gt; db.topics.ensureIndex({marker_ids: 1})
</code></pre>
插入
<pre><code>&gt; t = db.topics.findOne()
...
&gt; u = db.user.findOne()
...
&gt; db.topics.update({_id : t._id}, {$push : {maker_ids : u._id}})
</code></pre>
删除
<pre><code>&gt; db.topics.update({_id : t._id}, {$pull : {maker_ids : u._id}})
</code></pre>
查询
<pre><code>&gt; db.topics.find({maker_ids : u._id})
...
</code></pre>
<h2>3. 用户个性资料（profile）</h2>
典型的 embed 文档类型：只随着 user 数据读取而读取，数据量小

模式
<pre><code>&gt; u = db.users.findOne()
{
    _id : ObjectID(...),
    username : "username",
    email : "<a href="mailto:you@exmaple.com">you@exmaple.com</a>",
    profile :
        {
            name : "nickname",
            url : "you.domain.com"
        }
}
&gt; u.profile.url
you.domain.com
</code></pre>
<h2>4. 小结</h2>
mongodb 的功能很强大，很多特性并没有用在 campo 当中，也并不需要强塞进去（比如 mapReduce、分布式）。

当我刚决定选择 mongodb  做主数据库的时候，还是多少有点担心，这是否太激进，不符合循序渐进的原则？但两个月后的现在，我觉得这是个合适的选择。文档型数据库的数据模式更灵活， 不用花费额外的链接表，开发过程去掉了数据模式迁移的负担；json 式的查询接口对使用过 ActiveRecord 的人很容易上手。

此外，mongodb  并不是一个“已冻结”的技术。我们或多或少都会希望自己所用的某个工具的功能一直不增不减，就只完成自己所熟识的那部分工作。mongodb  还在演变中，它的在线文档有部分功能是标记为测试版本实现的，它的数据模式范例最后告诉你，没有最好的模式，最终要取决于你的应用。

我不确定 mongodb 是否能够或者已经进入 web 开发的主流，但我觉得与其等待，目睹或者助其成为主流会是一件更有意思的事。
