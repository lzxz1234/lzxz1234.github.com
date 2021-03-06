---
layout: post
title: Jekyll-指南
category : jekyll
tags : [Jekyll]
---
{% include JB/setup %}

本文将会简要介绍下 Jekyll 是什么和你为什么用它，再之后是它能干什么和具体怎么干的。

## 简介

### Jekyll 是什么

Jekyll 是一个基于Ruby的解析引擎，它可以用于将各种模板语言构建成一个静态网站，如templates, partials, liquid, markdown 等。也就是一个简单的博客形态的静态站点生产机器。

### 例如

这些网站是用 Jekyll 创建的. [查看](https://github.com/mojombo/jekyll/wiki/Sites).


### Jekyll 能干什么?

Jekyll 是一个 Ruby 组件，你可以在本地安装它。安装完成你就可以在一个按符合 Jekyll 要求的目录调用 `jekyll --server`了，然后它就会解析 markdown 文件，计算文档分类，标签，生成永久链接，最后按归 layout 目录下的模板组建整个网站。

一旦处理完成， Jekyll 就会把这些结果保存在一个名叫 `_site` 的私有目录中。这样做的好处是你可以通过任何静态网络服务器发布这个目录下的全部内容并提供服务了。

你可以认为 Jekyll 是一个正常的动态博客，但不是在每次请求来的时候解析内容，处理模板然后返回，它会把这些工作全部预告完成并缓存在一个目录里，最后提供静态服务。

### Jekyll 不是一个博客程序

**Jekyll 是一个处理引擎。**

Jekyll 没有任何内容也没有任何模板或设计元素。这是一个初始者很容易搞错的地方。Jekyll 不包含任何你经常用到或者看到的功能 - 你必须自己做。

### 我应该关注什么?

Jekyll 是非常简单高效的。关于 Jekyll 你应该明白的最重要的一件事情是它可以把你的网站发布成一个仅依赖静态服务器的静态网站。Wordpress 之类的动态博客系统都需要一台数据库和运行在服务器上的对应代码。这种博客系统在非常繁忙的情况下为了达到和 Jekyll 相同的效率只能通过一个缓存层，最后还是提供静态服务。

所以如果你喜欢保持简捷，并且你更喜欢命令行而不是管理后台那么尝试下 Jekyll 吧。

**开发者更喜欢 Jekyll 因为通过它我们可以像写代码一样写内容:**

- 允许你使用你最喜欢的文本编辑器按 markdown 或者 textile 格式编辑文章.
- 允许通过 localhost 编辑和预览你的文章.
- 不需要联网.
- 可以通过 git 发布.
- 允许把你的网站发布到静态 WEB 服务器.
- 可以使用的免费的 GitHub Pages.
- 不需要数据库.

# Jekyll 是如何工作的

接下来是完整的并且的简明的 Jekyll 工作原理介绍。

需要注意的是本章会通过没有代码例子的快速介绍一下核心概念。本章不会告诉你如何做任何事情，而会让你对 Jekyll 世界发生了什么有一个全图概览。

学习这些核心概念可以帮你避免常见的挫折，并从根本上帮你更好的理解后面的代码示例。

## 初始设置

安装完 Jekyll 之后你需要把你的网站目录按 Jekyll 期待的那样进行下修改。 

### Jekyll 程序的基本格式

Jekyll 需要你的网站目录看起来是这样的:

    .
    |-- _config.yml
    |-- _includes
    |-- _layouts
    |   |-- default.html
    |   |-- post.html
    |-- _posts
    |   |-- 2011-10-25-open-source-is-good.markdown
    |   |-- 2011-04-26-hello-world.markdown
    |-- _site
    |-- index.html
    |-- assets
        |-- css
            |-- style.css
        |-- javascripts


- **\_config.yml**
	存储配置信息.

- **\_includes**
	这个目录下是一些通用的基本组件。你可以加载这些包含部分到你的布局或者文章中以方便重用。

- **\_layouts**
	这个目录是你的文章最终的展示模板，你可以为不同的页面设置不同的模板布局。

- **\_posts**
	这个目录保存着你的文章，文件名字需要满足 `YEAR-MONTH-DATE-title.MARKUP` 格式。

- **\_site**
	一旦 Jekyll 完成转换，就会将生成的页面放在这里。最好将这个目录放进你的 `.gitignore` 文件中。 

- **assets**
	这个目录不是 Jekyll 标准结构中的一部分。这个 `assets` 目录代表你在根目录下创建的任何文件夹。这些非标准的目录均会被原封不动的拷贝到 `_site` 中。

(详细阅读: <https://github.com/mojombo/jekyll/wiki/Usage>)


### Jekyll 配置文件

Jekyll 支持的各种配置项在此处均有说明：(<https://github.com/mojombo/jekyll/wiki/Configuration>)




## Jekyll 中的内容

Jekyll 中的内容包括博客页(post)和页面页(page)。这些内容对象会在生成最终的静态页面时被插入到一个或者多个模板中。

### 博客和页面

博客和页面都需要按 markdown、textile 或者 Html 格式编写，当然也可能包含 Liquid 模板语法。每个页面也都可以有自己的元数据，包括标题、URL地址或者各种自定义元数据等。

### 撰写博客

**创建文章**
文章的创建是通过新建指定格式的文件并将文件放到 `_posts` 目录完成的。

**关于格式**
一篇文章有一个满足格式 `YEAR-MONTH-DATE-title.MARKUP` 的合法文件名并被放在 `_posts` 目录中。日期格式不正确导致 Jekyll 不能正确识别的文件是不会被当作博客文章的。因为博客文章的日期和名字属性是通过文件名自动识别的。此外，每个文件在实际内容前面必需有一个 [YAML 头](https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter)。YAML 头是一种 YAML 语法可以用来指定该文件的元数据。

**排序**

排序是 Jekyll 中非常重要的一部分但它不支持自定义策略排序。Jekyll 只支持按时间顺序和按时间倒序。

因为时间是通过文件名按特定格式写死的，所以为了改变顺序，你只能通过修改文件名实现。

**标签**
博客中的文章可以具有与之相关的标签属性作为元数据的一部分。标签可以放在文章文件中作为 YAML 头的一部分。你在模板中可以任意使用这些标签。

**目录**
各文章可以通过 YAML 头指定的一个或者多个目录名称，然后做归类操作。目录比标签具有更多的意义因为它可以体现在文章的 URL 地址中。
需要注意的是 Jekyll 的目录是按一种特别的方式工作的。如果你定义了多于一个目录名，那么它是是一个按层次的目录结构。例如:

    ---
    title :  Hello World
    categories : [lessons, beginner]
    ---

这个声明定义了一个目录结构 "lessons/beginner". 注意这在 Jekyll 中是一个目录结点.你找不到两个名为 "lessons" 和 "beginner" 的独立目录，除非你在别的地方把他们当作单独目录重新定义。

### 撰写页面

**创建页面**
页面的创建是通过新建指定格式的文件并把它们放在根目录或者不以下划线开头的任何子目录下完成的。

**关于格式**
为了注册成 Jekyll 页面页这个文件必须包含一个 [YAML 头](https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter)。注册页面意味着

1. Jekyll 会处理这个页面
2. 这个页面会被包含在 `site.pages` 数组中，然后你可以在你的模板中引用他们

**目录和标签**
页面既不会计算目录也不会计算标签，所以即使定义了也没用。

**子目录**
如果页面是在子目录中定义的，到达这个文件的路径会在 URL 中体现出来。例如：

    .
    |-- people
        |-- bob
            |-- essay.html

这个页面将会通过地址 `http://yourdomain.com/people/bob/essay.html` 访问。


**需要关注的页面**

- **index.html**
  你通常会重定义根目录下的 index.html 因为这个页面会作为你的网站首页展示。
- **404.html**
  在根目录下创建一个 404.html 然后 Github Pages 会把这个页面当作你的 404 页面。
- **sitemap.html**
  生成一个网站地图是进行 SEO 的一条最佳实践。
- **about.html**
  做一个好的关于页面不难而且他会使你的网站给所有人留下一个好印象。


## Jekyll 中的模板

模板是用来展示文章或者页面的模板。所有的模板都可以访问一个网站全局变量 `site` 和一个页面变量 `page`。网站变量存有网站中全部可访问的资源和网站元数据。页面变量存有将要渲染的页面或者文章中的全部可用数据。

**创建模板**
模板的创建是通过新建指定格式的文件并把它们放`_layouts`目录下完成的。

**关于格式**
模板需要按 HTML 格式编码并且包含 YAML 头。所有模板都可以包含 Liquid 代码并能访问全部网站资源。

**在模板中渲染 页面/文章 内容**
在所有模板都有一个特殊变量：`content`。这个 `content` 变量持有页面或者文章的全部内容（包括通过任何子模板预先定义的内容）。你可以将 `content` 变量嵌入到你的模板中的任何地方：

{% capture text %}...
<body>
  <div id="sidebar"> ... </div>
  <div id="main">
    |.{content}.|
  </div>
</body>
...{% endcapture %}
{% include JB/liquid_raw %}

### 子模板

子模板也是模板，它和普通模板唯一的区别是它需要在 YAML 头中指定一个 "root" 模板。所以它本质上就是一个可以在别的模板中渲染的模板。

### 引用文件
在 Jekyll 中你可以通过将文件放在 `_includes` 目录中做到定义引用文件。引用文件不是模板，它们只是可以被包含在模板中的代码片段。在这种方式下，你可以认为引用文件中的代码和在模板中是完全一样的。

任何可用的模板代码都可以在引用文件中使用。


## 在模板中使用 Liquid

模板可能是 Jekyll 中最让人困惑和沮丧的一部分。导致这个现象的很大一部分原因就是 Jekyll 模板必须使用 Liquid 模板语言。

### Liquid 是什么?

[Liquid](https://github.com/Shopify/liquid) 是 [Shopify](http://shopify.com) 开发的一个相当安全的模板语言。Liquid 是为了让用户能够在不引入服务器安全风险的前提下在模板文件中执行业务逻辑而设计的。

Jekyll 使用 Liquid 作为工作在你的网站和 文章/页面 数据间的主要接口，并按页面布局文件生成最终博客内容页。

### 为什么我们必须使用 Liquid?

GitHub 使用 Jekyll 来驱动 [GitHub Pages](http://pages.github.com/).
GitHub 不能承担在他们的服务器上任意运行代码的风险，所以他们通过 Liquid 锁定开发者的低权限。

### Liquid 不是程序员友好的.

简单来说 Liquid 不是真的代码而且它也不打算运行实际代码。在 Liquid 中你不能执行它明确指出允许操作之外的任何操作。更严格的说法是你只能获取被明确指出的数据结构。

在 Jekyll 中，不通过修改安装包或者运行自定义插件是不可能修改传递给 Liquid 的数据的。以上这两个在 GitHub Page 中也都不能被支持。

作为一个程序员 - 这是非常令人沮丧的.

与其探究这种机制的价值不如尝试利用它，把它当成一个在有限条件下工作的机会，并尽量从客户端找寻解决方案。

**此外**
我的个人立场是不要浪费时间去突破 Liquid 的这些限制。从一个程序员的角度来看也确实是不需要的。如果你有编写自定义插件的能力（即编写任意 Ruby 代码），用 Ruby 解决还是好一些的。基于这种考虑我写了 [Mustache-with-Jekyll](http://github.com/plusjade/mustache-with-jekyll)。


## 静态资源

表态资源是指在根目录或者非关键子目录（下划线开头的）下的任意非页面文件。它们没有 YAML 头信息并且也不会被当作 Jekyll 页面处理。

静态资源主要用来处理图片、CSS、Javascript文件。

## Jekyll 如何解析文件

记住 Jeykyll 只是一个解析引擎。在 Jekyll 中有两种主要解析需要：

- **内容解析.**
	通过 textile 或者 markdown 实现。
- **模板解析.**
    通过 Liquid 模板语言实现.

所以也就有两种文件格式需要解析：

- **文章文件和页面文件.**
  在 Jekyll 中的内容要么是文章要么是文件，他们都是通过 markdown 或者 textile 实现解析的。
- **模板文件.**
	这些文件都在 `_layouts` 目录下，它们就是你整个网站的**模板**。这些文件都是包含 Liquid 语法的 HTML 文件。因为模板引用只是简单的嵌入所以它们和直接原生在模板中没有任何解析上的区别。
	
**未提及的任意文件和目录.**
不能当作页面文件处理的文件都会被当作静态文件，它们会被 Jekyll 原封不动的发布到你的博客中，访问路径和它们的存储原始路径一致。

### 关于格式化文件.

我们已经指出过合法的格式需要存在 **YAML 头** 信息。模板、文章和页面都需要有 YAML 头信息，即使它是空的。只有这样 Jekyll 才会知道你希望这个文件被它处理。

YAML 头信息必须被放在 模板/文章/页面 文件的最开始部分：

    ---
    layout: post
    category : pages
    tags : [how-to, jekyll]
    ---

    ... 内容 ...

YAML 头信息以三个连字符开始和结束。中间的内容必须是合法的 YAML 信息。

YAML 头的可用配置参数在此处查看：
[YAML 格式指南](https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter)

#### 关于文章和模板的布局解析.

在 YAML 头标注中的 `layout` 参数定义了给定的文章或者模板应当嵌入的布局文件。如果一个布局文件指定了它自己的布局文件，它会被当作一个 `子模板` 使用。也就是说把一把文章按引用其它模板的模板渲染的时候它会按你希望的方式工作，作为一个子模板。

## Jekyll 如何生成最终的静态文件.

归根结底， Jekyll 的工作是为你的网站做静态化处理。接下来会描述它是怎么做到的：

1. **Jekyll 收集数据.**
  Jekyll 扫描文章目录并收集所有的文章文件。然后扫描收集布局文件和静态文件，最后扫描其它目录收集页面文件。
  
2. **Jekyll 预处理文件.**
  Jekyll 获取前面扫描到的对象，计算元数据（链接、标签、目录、标题、日期等）并组装成一个包含所有文章、页面、布局视图元数据的大 `site` 对象。到达这一步的时候你的整个网站就是一个大 Ruby 对象。

3. **Jekyll 使用 Liquid 处理文章和模板.**
  然后 Jekyll 遍历全部文章文件并用 markdown 或者 textile 转化最后通过 Liquid 根据布局文件渲染。一旦文章文件被解析完成并被按布局结构渲染完成，那么它就是 Liquid 处理完成的。
  **Liquification** 定义如下: Jekyll 初始化一个 Liquid 模板，并把代表网站的简单对象和代表文章的 Ruby 对象传递给它。这些对象就是你能在模板中访问的全部内容。

3. **Jekyll 生成输出.**
  最终这些 Liquid 模板就是经过渲染的，模板出现的任何 Liquid 语法都是处理完成的，并且把生成的静态内容保存到文件中。

**Notes.**
因为 Jekyll 会一次性计算整个网站，所以模板也就得到了一个包含全部有效数据的全局 `site` 对象。你可能会希望遍历这个对象并用 Liquid 标签和过滤器格式化后渲染到特定页面上。

记住，在 Jekyll 里你只是一个最终用户。你的 API 只有两个组成部分：

1. 自定义目录的方式.
2. Liquid 语法和传递给 Liquid 模板的变量.

通过 Liquid 你能在模板中获取的全部对象在 [**API 章节**](/jekyll/2015/01/09/Jekyll-Variables/) 已给出说明。你也可以从这阅读原始文档：<http://jekyllcn.com/docs/variables/>

## 结论

我希望这篇文章已经说清楚 Jekyll 的用途和工作原理。正如前面说的，我们的程序能且只能限制在 Liquid 允许的范围内。

Jekyll-bootstrap 的目标就是通过提供一些方法和策略让你能更直观和轻松的使用 Jekyll 工作。

**感谢** 你能读完全文.

