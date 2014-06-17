---
layout: post
title: JavaMail 可能导致强制重启服务器
category : Java
tags : [JavaMail]
---
{% include JB/setup %}

说到 JavaMail 的配置文件还是相当有趣的。基本上你只能按你自己的理解去填写一堆没有类型的 Map 或者 Properties 结构的配置项。网络上大量的指南也只说明了能让它正常工作的最小配置要求（发送/接收 邮件）。

然而，当我们学的无比痛苦的时候，还有几个鲜为人知的配置项你应该同样关注下，那就是为网络输入输出流的超时设置。默认情况下， JavaMail 为所有的网络操作（新建连接，输入输出等等）采用的超时是**正无穷**！。

现在假使你有一个用来往外发邮件的 SMTP 服务器集群，它们通过 DNS 进行均衡。如果其中的一台挂掉了，恰好还有一封邮件需要通过它发出去，那么你的发送线程将会被永远挂起!这就是我们切实碰到的，并且也需要寻找方法解决的。

所以，我们现在为所有的操作都设了超时：

	String MAIL_SMTP_CONNECTIONTIMEOUT ="mail.smtp.connectiontimeout";
	String MAIL_SMTP_TIMEOUT = "mail.smtp.timeout";
	String MAIL_SMTP_WRITETIMEOUT = "mail.smtp.writetimeout";
	String MAIL_SOCKET_TIMEOUT = "60000"; 
	
	// Set a fixed timeout of 60s for all operations - 
	// the default timeout is "infinite"
	props.put(MAIL_SMTP_CONNECTIONTIMEOUT, MAIL_SOCKET_TIMEOUT);
	props.put(MAIL_SMTP_TIMEOUT, MAIL_SOCKET_TIMEOUT);
	props.put(MAIL_SMTP_WRITETIMEOUT, MAIL_SOCKET_TIMEOUT);

同时，如果你使用基于 DNS 进行负载的服务提供商（像亚马逊S3）或者我们那样的邮件服务器集群，不要忘了设置 Java 中的 DNS 缓存时间（它的默认时间也是**正无穷**）。

	// Only cache DNS lookups for 10 seconds 
	java.security.Security.setProperty("networkaddress.cache.ttl","10");

说到这再提一句，实践证明为了系统的可靠性把所有的编码都设置成平台无关的 UTF-8 也是一个很不错的选择：

	System.setProperty("file.encoding", Charsets.UTF_8.name());
	System.setProperty("mail.mime.charset", Charsets.UTF_8.name());

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://andreas.haufler.info/2014/06/javamail-can-be-evil-and-force-you-to.html "JavaMail can be evil (and force you to restart your app server) ")