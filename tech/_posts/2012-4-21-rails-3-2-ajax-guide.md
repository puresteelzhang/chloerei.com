---
layout: post
title: Rails 3.2 的 Ajax 向导
---
前不久入手了《[Web开发敏捷之道][1]》的中文第4版，翻看了 Ajax 部分，发现竟然还是使用 .rjs 模板。.rjs 模板在 3.1 版以后已经被移除。另外我又去看了官方的 [Rails guide][2]，发现也没有讲述 Ajax 的章节。

也就是说一个新手入门，很可能搞不清楚 Rails 对 Ajax 是如何支持的。所以我想在这写写 Rails 3.2 的 Ajax 向导，让不了解 Ajax 的新手了解 Rails 3.2 的 Ajax 处理，或者了解 Ajax 不了解 Rails 的人知道 Rails 对其提供了什么支持。

### Ajax 是什么？

虽然 Ajax 已经遍地都是，不过还是介绍下 Ajax。简单的说，Ajax 是提供一种浏览器和服务端的异步交互方式，实现无刷新页面的情况下更新页面。

最原始的网页浏览方式是通过超链接，每点击一个超链接，浏览器会从服务端下载完整的网页并更新用户看到的内容，这样用户会看到短暂的白页或者闪烁。但是通过 Ajax 处理，浏览器可以只从服务端下载需要更新的页面片段，然后在不重载页面的情况下更新当前网页的某个部分，实现内容更新。

不过 Ajax 已经不局限于页面局部了，小至 CodeCampo 的提交评论，大至 Twitter，Gmail 的整页 Ajax，都属于 Ajax 范围。Ajax 给用户更好的操作体验，并且节省流量。

明白 Ajax 的目的之后，就可以开始考虑如何在 Rails 里面实现 Ajax。

### Rails Ajax 的两种类型

Ajax 的实现不拘一格，Rails 所处的服务端即既可以处于控制者的角色，完全处理数据和更新逻辑，又可以仅仅处理数据，由浏览器端的 JavaScript 决定更新逻辑。

于是在 Rails 里面处理 Ajax 大致可以分为两种：

 1. 用服务端模板(.js.erb)提供的服务端的 Ajax
 2. 服务端只提供 json api 的客户端的 Ajax

这两种 Ajax 实现，实际上都是由客户端发起的 Ajax 请求（通常由 JavaScript 控制），然后由服务端返回数据。而不同的是，服务端既可以只返回一段 json 的纯数据，也可以将更新所需的 JavaScript 逻辑一起打包返回，浏览器在接收到这段数据+逻辑之后，完全按照这段逻辑进行更新操作，看起来就是完全由服务器控制一样。数据+逻辑这种打包方式我就暂称为服务端 Ajax。

需要选择哪种方式的 Ajax，取决于你的网站的定位。如果网址是传统的浏览方式为主，只是需要在局部加上 Ajax 效果，比如 CodeCampo 的评论回复，那么用服务端的 Ajax 是最方便的。如果你想做一个类似 Twitter 的全 Ajax 站点，那么 json api 形式的 Ajax 更适合，你还可以看看 [spine.js][3] 这个前端 mvc 框架。

### 服务端的 Ajax

先看一段代码

    # POST /replies
    def create
      @reply = @topic.replies.new params[:reply].merge(:user => current_user)
      if @reply.save
        redirect_to @topic
      else
        render :new
      end
    end

这段代码是通常的创建 Reply 的逻辑，它的表单是这样的（haml）

    = form_for @reply do |f|
      = f.text_area :body

创建回复的时候，如果成功，服务端会要求重定向到话题页面，如果不成功，服务器会渲染 :new 页面，以便用户重新编辑。

现在来为它加上 Ajax 效果，回复成功的时候，将回复添加到页面回复列表的后面，否则显示一条错误信息。

    # POST /replies
    def create
      @reply = @topic.replies.new params[:reply].merge(:user => current_user)
      respond_to do |format|
        if @reply.save
          format.html { redirect_to @topic }
          format.js { render :layout => false }
        else
          format.html { render :new }
          format.js { render :layout => false, :status => 406  }
        else
        end
      end
    end

注意看 respond_to 这个方法，通过这个方法，Rails 可以分辨客户端请求的是 html 格式，还是 js 格式的返回，不同的格式请求可以进行不同的渲染输出。

这里的 `format.js { render :layout => false }` 声明了，请求 js 格式的时候去渲染名为 create.js.erb的模板。现在来添加一个 create.js.erb 模板。

    <% if @reply.errors.empty? %>
      $('<%= escape_javascript(render :partial => 'reply', :object => @reply) %>').hide().appendTo($('#replies table > tbody')).fadeIn('fast').css('display', 'table-row');
    <% else %>
      $('<span class="error-message"><%= escape_javascript(@reply.errors.full_messages.join(',')) %></span>').appendTo($('#new_reply_form'))
    <% end %>

这个模板区分了 @reply 保存成功和失败的两种情况，当保存成功的时候，新回复会被添加到回复列表的后面（还加了一些视觉效果），否则会在评论框后添加一条错误信息。

最后还有关键的一步，让评论表单在提交的时候发出 Ajax 请求。

    = form_for @reply, :remote => true do |f|
      = f.text_area :body
      = f.submit 'post'

这就行了！注意看 `:remote => true` ，这是一个魔法参数，添加了这个参数之后，表单渲染结果将会加上一个 `data-remote="true"` 属性，这会通知 Rails 的 jquery-ujs 前端模块去执行 Ajax 请求。如果你想知道这魔法是怎么实现的，可以去看看 [jquery-ujs][4] 的源码。

这就是一个服务端 Ajax 的基本做法。上面的代码做了一定程度的简化，要实际跑起来估计还需要一些调试，这就留给你啦。

### 客户端 Ajax

在服务端 Ajax 看来，客户端只是个纯粹的执行者，服务端控制了所有逻辑。但这有时候会产生问题，因为服务端并不总能确定客户端所需的处理方案。例如 Twitter 的 Follow 功能，可能是在用户页面触发的，也可能是在用户信息的弹出框触发的，如果让服务端区分所有调用情况，那会变得异常复杂，页面结构的一些调整，都需要服务端逻辑的配合。

最好的方法，就是将逻辑部分下放到客户端处理，让逻辑跟随页面，服务端退居为数据 API。

那么要怎么实现呢，继续沿用之前的例子，得益于 Rails 的灵活性，完全可以在不干扰先前实现的情况下添加 Json API 特性。

将 Controller 代码进行以下修改

    # POST /replies
    def create
      @reply = @topic.replies.new params[:reply].merge(:user => current_user)
      respond_to do |format|
        if @reply.save
          format.html { redirect_to @topic }
          format.js { render :layout => false }
          format.json { render :json => @reply } # this
        else
          format.html { render :new }
          format.js { render :layout => false, :status => 406  }
          format.json { render :json => {:errors => @reply.errors.full_messages.join(',')} } # and this
        else
        end
      end
    end

仅仅增加了两行，就为 create 这个动作增加了 json api 功能！并且，服务端的任务到此结束，接下来的工作交给客户端逻辑。

这意味着你的页面也许会包括这样的代码

表单（不使用 `:remote => true`）

    = form_for @reply do |f|
      = f.text_area :body
      = f.submit 'post'

JavaScript

    $('new_reply_form').on('submit', function(event){
        $.ajax({url: $(this).prop('action'), dataType: 'json', /* more option */} 
        )
    })

接下来的，就都是 Javascript 的事了。

### 该用哪种实现？

如果是刚接触 Rails 的新手，并且以前没有 Ajax 的经验，那么建议先体验一下 Rails 服务端 Ajax。

如果是从前端框架（例如 Spine.js）起步，一开始就打算搭建全 Ajax 站点，只是将 Rails 当作 API 服务器，那么可以果断用 JSON API 方式的 Ajax。

### 总结

总的来说，对于 Rails 你需要了解几点：

1. respond_to api [http://apidock.com/rails/ActionController/MimeResponds/respond_to][5]
2. 对于服务端 Ajax，了解 js.erb
3. 对于客户端 Ajax，了解 Javascipt 的 Ajax 处理，比如 [http://api.jquery.com/jQuery.ajax/][6]

希望这篇日志对你了解 Rails 的 Ajax 有帮助 : P

  [1]: http://book.douban.com/subject/10528446/
  [2]: http://guides.rubyonrails.org/
  [3]: http://spinejs.com/
  [4]: https://github.com/rails/jquery-ujs
  [5]: http://apidock.com/rails/ActionController/MimeResponds/respond_to
  [6]: http://api.jquery.com/jQuery.ajax/
