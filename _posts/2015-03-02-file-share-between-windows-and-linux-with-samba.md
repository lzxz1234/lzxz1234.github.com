---
layout: post
title: 通过 samba 实现 Windows 和 Linux 的文件共享
category : deploy 
tags : [Shell, Deploy]
---

{% include JB/setup %}

SMB（Server Messages Block，信息服务块）是一种为局域网内的不同计算机之间提供文件及打印机等资源共享的服务，而 Samba 是在 Linux 和 UNIX 系统上实现SMB协议的一个免费软件，通过它我们就可以实现 Windows 和 Linux 间的文件共享。

#### 安装 samba:
{% highlight sh %}
yum install samba
{% endhighlight %}

#### 编辑 /etc/samba/smb.conf
在配置文件的最后添加：
{% highlight sh %}
[share name]
comment = Public Stuff
path = /var/share
public = yes
writable = yes
{% endhighlight %}

#### 启动 samba 服务：
{% highlight sh %}
service smb start
{% endhighlight %}