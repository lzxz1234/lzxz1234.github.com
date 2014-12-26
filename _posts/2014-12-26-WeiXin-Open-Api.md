---
layout: post
title: 微信公众号开发框架
category : WeiXin
tags : [Java, WeiXin]
---
{% include JB/setup %}

本文是项目 [WeiXin](https://github.com/lzxz1234/WeiXin "WeiXin") 的说明文档。

### 起步 ###

在 web.xml 中配置如下信息：

{% highlight xml linenos %}
	<servlet>
	    <servlet-name>weixin</servlet-name>
	    <servlet-class>com.chn.wx.WeiXinServlet</servlet-class>
	    <init-param>
	        <param-name>package</param-name>
	        <param-value>com.chn.wx.listener</param-value>
	    </init-param>
	    <init-param>
	        <param-name>appid</param-name>
	        <param-value>wx391f84dcab2d80bc</param-value>
	    </init-param>
	    <init-param>
	        <param-name>secret</param-name>
	        <param-value>cf4347b8c75bb09602d16ad3495ff280</param-value>
	    </init-param>
	</servlet>
	<servlet-mapping>
	    <servlet-name>weixin</servlet-name>
	    <url-pattern>/*</url-pattern>
	</servlet-mapping>
{% endhighlight %}

### 关于 ###

`com.chn.wx.listener` 中的所有实现 `Service` 接口的类会被组装成一棵流程树，目前结点如下：

![流程图]({{ site.JB.POST_IMG_PATH }}/20141226144110.png)

结点与父结点的关系通过 `@Node(value = "raw", parent = EncryptRouter.class)` 指定。
父结点到子结点的路由通过 `tree.route(context, key)` 完成，key 即为 `Node` 中的 value 属性。

### 消息返回 ###

`Service` 的实现方法中返回的字符串会被写回到请求流中，需要返回消息时，调用 `com.chn.wx.template.PassiveMessage` 中的对应方法生成报文返回即可。

### 主动调用 ###

- `com.chn.wx.invocation.FileManager` 上传下载多媒体文件
- `com.chn.wx.invocation.GroupManager` 分组管理
- `com.chn.wx.invocation.ServiceMessageSender` 客服消息发送
- `com.chn.wx.invocation.TokenAccessor` 唯一的 token 获取入口, token 只能能过该类获取，不能另做缓存

### 语法糖 ###

`Service` 实现类中被`@Param`注解标记的字段，会被注入成 `Context` 中对应的属性值，当然也可以直接通过 `Context` 读取。

### 相关资源 ###

- [[接口文档]](http://mp.weixin.qq.com/wiki/home/index.html "接口文档") 接入指南
- [[测试地址]](http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login "测试地址") 接口体验账号申请地址
- [[本机公网发布]](https://ngrok.com "ngrok") 可以将本机服务发布到公网80端口便于测试

转载注明出处：[{{page.title}}]({{permalink}})
