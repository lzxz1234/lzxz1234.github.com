---
layout: post
title: Liquid 语法索引
category : jekyll
tags : [Jekyll, Liquid]
---

{% include JB/setup %}

本文只是一个索引，几乎无任何注释。首先，Liquid 包括以下两种标记：

{% raw %}
- 输出块

> {{ 双大括号 }}

- 逻辑块

> {% 大括号+百分号 %}

{% endraw %}

## 输出块 ##

基本样例：

{% highlight liquid %}
{% raw %}
Hello {{name}}
Hello {{user.name}}
Hello {{ 'tobi' }}
{% endraw %}
{% endhighlight %}

#### 进阶：过滤器 ####

{% highlight liquid %}
{% raw %}
Hello {{ 'tobi' | upcase }}
Hello tobi has {{ 'tobi' | size }} letters!
Hello {{ '*tobi*' | textilize | upcase }}
Hello {{ 'now' | date: "%Y %h" }}
{% endraw %}
{% endhighlight %}

#### 可选过滤器列表 ####

- `date` - 日期格式化
- `capitalize` - 首字母大写
- `downcase` - 转换为小写
- `upcase` - 转换为大写
- `first` - 获取传数组的第一个结点
- `last` - 获取传数组最后一个结点
- `join` - 按指定的间隔连接数组元素
- `sort` - 传入的数组排序
- `map` - 
- `size` - 数组或者字符串的长度
- `escape` - 安全输出
- `escape_once` - 
- `strip_html` - 删掉 HTML 标签
- `strip_newlines` - 删掉换行符
- `newline_to_br` - 用 {{ '<br />' | escape }} 替换换行符{% raw %}
- `replace` - 替换，如 `{{ 'foofoo' | replace:'foo','bar' }} #=> 'barbar'`
- `replace_first` - 如 `{{ 'barbar' | replace_first:'bar','foo' }} #=> 'foobar'`
- `remove` - 如 `{{ 'foobarfoobar' | remove:'foo' }} #=> 'barbar'`
- `remove_first` - 如 `{{ 'barbar' | remove_first:'bar' }} #=> 'bar'`
- `truncate` - 截取，如 `{{ 'foobarfoobar' | truncate: 5, '.' }} #=> 'foob.'`
- `truncatewords` - 按单词截取
- `prepend`- 如 `{{ 'bar' | prepend:'foo' }} #=> 'foobar'`
- `append` - 如 `{{ 'foo' | append:'bar' }} #=> 'foobar'`
- `slice` - 截取，参数包括位移和长度，如 `{{ "hello" | slice: -3, 3 }} #=> llo`
- `minus` - 减，如 `{{ 4 | minus:2 }} #=> 2`
- `plus` - 加，如 `{{ '1' | plus:'1' }} #=> '11'`，`{{ 1 | plus:1 }} #=> 2`
- `times` - 乘，如 `{{ 5 | times:4 }} #=> 20`
- `divided_by` - 除，如 `{{ 10 | divided_by:2 }} #=> 5`
- `split` - 分割，如 `{{ "a~b" | split:"~" }} #=> ['a','b']`
- `modulo` - 模，如 `{{ 3 | modulo:2 }} #=> 1`
{% endraw %}

## 逻辑块 ##

- **assign** - 赋值
- **capture** - 捕捉文本并赋值
- **case** - `case when` 语块
- **comment** - 注释
- **cycle** - 几个值中间循环
- **for** -  循环
- **if** - 决断逻辑块
- **include** - 包括另一个模板
- **raw** - 原样输出
- **unless** - 判断块的另一种写法

#### 注释 ####

{% highlight liquid %}
{% raw %}
We made 1 million dollars {% comment %} in losses {% endcomment %} this year
{% endraw %}
{% endhighlight %}

#### 原始输出 ####

{% assign openTag = '{%' %}
{% highlight liquid %}
{% raw %}
{% raw %}
  In Handlebars, {{ this }} will be HTML-escaped, but {{{ that }}} will not.
{% endraw %}{{ openTag }} endraw %}
{% endhighlight %}

#### 判断逻辑块 ####

{% highlight liquid %}
{% raw %}
{% if user %}
  Hello {{ user.name }}
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
#Same as above
{% if user != null %}
  Hello {{ user.name }}
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
{% if user.name == 'tobi' %}
  Hello tobi
{% elsif user.name == 'bob' %}
  Hello bob
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
{% if user.name == 'tobi' or user.name == 'bob' %}
  Hello tobi or bob
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
{% if user.name == 'bob' and user.age > 45 %}
  Hello old bob
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
{% if user.name != 'tobi' %}
  Hello non-tobi
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
# Same as above
{% unless user.name == 'tobi' %}
  Hello non-tobi
{% endunless %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
# Check for the size of an array
{% if user.payments == empty %}
   you never paid !
{% endif %}

{% if user.payments.size > 0  %}
   you paid !
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
{% if user.age > 18 %}
   Login here
{% else %}
   Sorry, you are too young
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
# array = 1,2,3
{% if array contains 2 %}
   array includes 2
{% endif %}
{% endraw %}
{% endhighlight %}

{% highlight liquid %}
{% raw %}
# string = 'hello world'
{% if string contains 'hello' %}
   string includes 'hello'
{% endif %}
{% endraw %}
{% endhighlight %}

#### Case 块 ####

{% highlight liquid %}
{% raw %}
{% case template %}

{% when 'label' %}
     // {{ label.title }}
{% when 'product' %}
     // {{ product.vendor | link_to_vendor }} / {{ product.title }}
{% else %}
     // {{page_title}}
{% endcase %}
{% endraw %}
{% endhighlight %}

#### 常量循环 ####

{% highlight liquid %}
{% raw %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% endraw %}
{% endhighlight %}
执行为
```
one
two
three
one
```

{% highlight liquid %}
{% raw %}
{% cycle 'group 1': 'one', 'two', 'three' %}
{% cycle 'group 1': 'one', 'two', 'three' %}
{% cycle 'group 2': 'one', 'two', 'three' %}
{% cycle 'group 2': 'one', 'two', 'three' %}
{% endraw %}
{% endhighlight %}
执行为
```
one
two
one
two
```

#### 循环 ####

数组循环：

{% highlight liquid %}
{% raw %}
{% for item in array %}
  {{ item }}
{% endfor %}
{% endraw %}
{% endhighlight %}

Map 循环：

{% highlight liquid %}
{% raw %}
{% for item in hash %}
  {{ item[0] }}: {{ item[1] }}
{% endfor %}
{% endraw %}
{% endhighlight %}

在循环中还有以下变量可用：

{% highlight liquid %}
forloop.length      # => 循环长度
forloop.index       # => 当前索引
forloop.index0      # => 基于 0 的当前索引
forloop.rindex      # => 剩余元素数
forloop.rindex0     # => 基于 0 的剩余元素数
forloop.first       # => 判断当前是不是第一个元素
forloop.last        # => 判断当前是不是最后一个元素
{% endhighlight %}

你可以控制循环的开始和结束点：

{% highlight liquid %}
{% raw %}
# array = [1,2,3,4,5,6]
{% for item in array limit:2 offset:2 %}
  {{ item }}
{% endfor %}
# results in 3,4
{% endraw %}
{% endhighlight %}

倒序循环：

{% highlight liquid %}
{% raw %}
{% for item in collection reversed %} {{item}} {% endfor %}
{% endraw %}
{% endhighlight %}

也可以循环一个数字范围：

{% highlight liquid %}
{% raw %}
# 如果 item.quantity 的值是 4...
{% for i in (1..item.quantity) %}
  {{ i }}
{% endfor %}
# results in 1,2,3,4
{% endraw %}
{% endhighlight %}

#### 变量赋值 ####

{% highlight liquid %}
{% raw %}
{% assign name = 'freestyle' %}

{% for t in collections.tags %}{% if t == name %}
  <p>Freestyle!</p>
{% endif %}{% endfor %}
{% endraw %}
{% endhighlight %}

如果你想把几个字符串连接起来后赋值给某变量可以这么干：

{% highlight liquid %}
{% raw %}
{% capture attribute_name %}{{ item.title | handleize }}-{{ i }}-color{% endcapture %}

<label for="{{ attribute_name }}">Color:</label>
<select name="attributes[{{ attribute_name }}]" id="{{ attribute_name }}">
    <option value="red">Red</option>
    <option value="green">Green</option>
    <option value="blue">Blue</option>
</select>
{% endraw %}
{% endhighlight %}


