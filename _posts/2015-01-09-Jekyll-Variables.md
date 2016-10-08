---
layout: post
title: Jekyll 常用变量列表
category : jekyll
tags : [Jekyll]
---
{% include JB/setup %}

## 全局变量 ##

<table>
  <thead>
    <tr>
      <th>变量</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>site</td>
      <td>来自_config.yml文件，全站范围的信息+配置。详细的信息请参考下文</td>
    </tr>
    <tr>
      <td>page</td>
      <td>页面专属的信息。</td>
    </tr>
    <tr>
      <td>content</td>
      <td>被 layout 包裹的那些 Post 或者 Page 渲染生成的内容。但是又没定义在 Post 或者 Page 文件中的变量。</td>
    </tr>
    <tr>
      <td>paginator</td>
      <td>每当 paginate 配置选项被设置了的时候，这个变量就可用了。</td>
    </tr>
  </tbody>
</table>

## 全站变量 ##

<table>
  <thead>
    <tr>
      <th>变量</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>site.time</td>
      <td>当前时间（运行jekyll这个命令的时间点）。</td>
    </tr>
    <tr>
      <td>site.pages</td>
      <td>所有 Pages 的清单。</td>
    </tr>
    <tr>
      <td>site.posts</td>
      <td>一个按照时间倒序的所有 Posts 的清单。</td>
    </tr>
    <tr>
      <td>site.related_posts</td>
      <td>如果当前被处理的页面是一个 Post，这个变量就会包含最多10个相关的 Post。默认的情况下，相关性是低质量的，但是能被很快的计算出来。如果你需要高相关性，就要消耗更多的时间来计算。用jekyll 这个命令带上 --lsi (latent semanticindexing) 选项来计算高相关性的 Post。</td>
    </tr>
    <tr>
      <td>site.categories.CATEGORY</td>
      <td>所有的在 CATEGORY 类别下的帖子。</td>
    </tr>
    <tr>
      <td>site.tags.TAG</td>
      <td>所有的在 TAG 标签下的帖子。</td>
    </tr>
    <tr>
      <td>site.[CONFIGURATION_DATA]</td>
      <td>所有的通过命令行和 _config.yml 设置的变量都会存到这个 site 里面。举例来说，如果你设置了 url: http://mysite.com 在你的配置文件中，那么在你的 Posts和 Pages 里面，这个变量就被存储在了 site.url。Jekyll 并不会把对 _config.yml做的改动放到 watch 模式，所以你每次都要重启 Jekyll 来让你的变动生效。</td>
    </tr>
  </tbody>
</table>

## 页面变量 ##

<table>
  <thead>
    <tr>
      <th>变量</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>page.content</td>
      <td>页面内容的源码。</td>
    </tr>
    <tr>
      <td>page.title</td>
      <td>页面的标题。</td>
    </tr>
    <tr>
      <td>page.excerpt</td>
      <td>页面摘要的源码。</td>
    </tr>
    <tr>
      <td>page.url</td>
      <td>帖子以斜线打头的相对路径，例子： /2008/12/14/my-post.html。</td>
    </tr>
    <tr>
      <td>page.date</td>
      <td>帖子的日期。日期的可以在帖子的头信息中通过用以下格式YYYY-MM-DD HH:MM:SS (假设是 UTC), 或者YYYY-MM-DD HH:MM:SS +/-TTTT ( 用于声明不同于 UTC 的时区，比如 2008-12-14 10:30:00 +0900) 来显示声明其他 日期/时间 的方式被改写，</td>
    </tr>
    <tr>
      <td>page.id</td>
      <td>帖子的唯一标识码（在RSS源里非常有用），比如/2008/12/14/my-post</td>
    </tr>
    <tr>
      <td>page.categories</td>
      <td>这个帖子所属的 Categories。Categories 是从这个帖子的 _posts 以上的目录结构中提取的。距离来说, 一个在 /work/code/_posts/2008-12-24-closures.md目录下的 Post，这个属性就会被设置成 ['work', 'code']。</td>
    </tr>
    <tr>
      <td>page.tags</td>
      <td>这个 Post 所属的所有 tags。</td>
    </tr>
    <tr>
      <td>page.path</td>
      <td>Post 或者 Page 的源文件地址。举例来说，一个页面在 GitHub 上的源文件地址。</td>
    </tr>
  </tbody>
</table>

>Use custom front-matter
任何你自定义的头文件信息都会在 page 中可用。 距离来说，如果你在一个 Page 的头文件中设置了 custom_css: true， 这个变量就可以这样被取到 page.custom_css。

## 分页器 ##

<table>
  <thead>
    <tr>
      <th>变量</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>`paginator.per_page`</td>
      <td>每一页 Posts 的数量。</td>
    </tr>
    <tr>
      <td>`paginator.posts`</td>
      <td>这一页可用的 Posts。</td>
    </tr>
    <tr>
      <td>`paginator.total_posts`</td>
      <td>Posts 的总数。</td>
    </tr>
    <tr>
      <td>`paginator.total_pages`</td>
      <td>Pages 的总数。</td>
    </tr>
    <tr>
      <td>`paginator.page`</td>
      <td>当前页号。</td>
    </tr>
    <tr>
      <td>`paginator.previous_page`</td>
      <td>前一页的页号。</td>
    </tr>
    <tr>
      <td>`paginator.previous_page_path`</td>
      <td>前一页的地址。</td>
    </tr>
    <tr>
      <td>`paginator.next_page`</td>
      <td>下一页的页号。</td>
    </tr>
    <tr>
      <td>`paginator.next_page_path`</td>
      <td>下一页的地址。</td>
    </tr>
  </tbody>
</table>

>分页器变量的可用性
这些变量仅在首页文件中可用，不过他们也会存在于子目录中，就像 /blog/index.html。


