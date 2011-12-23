---
layout: post
title: 《Metaprogramming Ruby》读后感
---

花了点时间看完了《<a href="http://book.douban.com/subject/4086938/">Metaprogramming Ruby</a>》，才知道以前根本不算认识 Ruby。Meta Programing 不是 Ruby 的特性之一，而是 Ruby 本身就是 Meta Programing。

<ol>
	<li>"定义"一个类（class）的时候，其实本质不是定义，而是打开一个类的域（scope）。所以写 Ruby 代码的时候可以在任何地方打开一个类（Open Class）。</li>
	<li>Ruby 代码的编写和运行并没有明显的界限。比如看似一个宏的 attr_*，其实是一个类方法。并且，Ruby 可以在运行的时候定义新方法，真正实现 write code to write code。</li>
</ol>

不过 Ruby 的对象体系有点意思，但又会让人迷惑：Class 继承自 Object，Object 的 class 是 Class，因为 class 是 object，所以 Class 是 object。（首字母大写是一个常量，小写是一个概念）

<a href="http://chloerei.com/wp-content/uploads/2011/01/objects-classes.png" rel="attachment wp-att-705"><img src="http://chloerei.com/wp-content/uploads/2011/01/objects-classes-500x373.png" alt="" title="objects-classes" width="500" height="373" class="aligncenter size-large wp-image-705" /></a>

Ruby 的 Meta Programing 的集大成者就是 Rails，学会用 Rails 之后，会发觉大多数 Ruby 魔法都已经见识过了。不过这本书更进一步的是，教怎么释放魔法。

动态语言与静态语言相比，动态语言让人避免写重复代码，不过由于其动态性难于被 IDE 理解，所以重构一类的工具比较难做。有得必有失，对于我来说，怎么也不会再去写 Java 那些重复繁杂的代码了。
