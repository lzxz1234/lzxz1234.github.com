---
layout: post
title: 关于 Jekyll 中 Disqus 的使用
category : jekyll
tags : [Jekyll, Disqus]
---
{% include JB/setup %}

默认情况下，Disqus 是把 URL 作为 KEY 存储评论的。所以如果有人在地址 `http://example.com/foo.aspx` 发表了评论，那么了为评论能一直正常显示，你得确保这个地址不能动。

但是，通过 Jekyll，发往地址 `http://example.com/foo.aspx` 的请求可以被重定向到 `http://example.com/foo.aspx/`。注意最后的斜杠，对 Disqus 来说，这是两个不同的地址，对前面地址的评论并不会在后面的地址中展示出来。

幸运的是，Disqus 允许你通过设置 [Disqus Identifier](http://help.disqus.com/customer/portal/articles/472099-what-is-a-disqus-identifier-) 来查找某个页面的评论线。

所以你可以在模板的 YAML 头中定义如下：

{% highlight javascript linenos %}
---
layout: post
title: "Code Review Like You Mean It"
date: 2013-10-28 -0800
comments: true
disqus_identifier: 18902
categories: [open source,github,code]
---
{% endhighlight %}

然后就可以通过 Jekyll 模板读取到 **disqus_identifier** 变量了，但现在还不行，因为默认的模板不知道如何使用它。所以我们继续修改 **disqus.html** 模板内容，主要如下：

{% highlight javascript linenos %}
var disqus_identifier = '{% if page.disqus_identifier %}{{ page.disqus_identifier}}{% else %}{{ site.url }}{{ page.url }}{% endif %}';
var disqus_url = '{{ site.url }}{{ page.url }}';
{% endhighlight %}

这样当你的模板中不存在 **disqus_identifier** 变量时，默认仍取 URL 作为主键。不会对原有的资源产生任何影响。


