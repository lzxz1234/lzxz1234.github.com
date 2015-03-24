---
layout: post
title: 基于 httpd 的 session 保持负载均衡
category : deploy 
tags : [Shell, Deploy]
---

{% include JB/setup %}

#### 安装 httpd:
{% highlight sh %}
[root@localhost ~]# yum install httpd
{% endhighlight %}

#### 安装 mod_jk:
{% highlight sh %}
[root@localhost ~]# wget http://mirrors.cnnic.cn/apache/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.40-src.tar.gz
[root@localhost ~]# tar xvf tomcat-connectors-1.2.40-src.tar.gz
[root@localhost ~]# cd native/
[root@localhost ~]# ./configure --with-apxs=/usr/sbin/apxs
[root@localhost ~]# make
[root@localhost ~]# make install
{% endhighlight %}

> 当不存在 apxs 时还需要安装 httpd-devel.x86_64 模块

#### 修改配置文件：

修改 mod_jk2.conf:

{% highlight sh %}
[root@localhost ~]# vi /etc/httpd/conf.d/mod_jk2.conf

LoadModule jk_module modules/mod_jk.so
JkWorkersFile conf.d/jkworkers.properties
JkLogFile logs/mod_jk.log
JkLogLevel info
JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "
JkOptions +ForwardKeySize +ForwardURICompat -ForwardDirectories
JkRequestLogFormat "%w %V %T"
JkMount /* loadBalancer
{% endhighlight %}

新建 jkworkers.properties:

{% highlight sh %}
[root@localhost ~]# vi /etc/httpd/conf.d/jkworkers.properties

worker.list=tomcat1, tomcat2, loadBalancer

worker.tomcat1.port=8009
worker.tomcat1.host=127.0.0.1
worker.tomcat1.type=ajp13
worker.tomcat1.lbfactor=100

worker.tomcat2.port=29080
worker.tomcat2.host=125.39.216.182
worker.tomcat2.type=ajp13
worker.tomcaat2.lbfactor=100

worker.loadBalancer.type=lb
worker.loadBalancer.balance_workers=tomcat1, tomcat2
worker.loadBalancer.sticky_session=1
{% endhighlight %}

最后修改 tomcat 结点名：

{% highlight sh %}
<Engine jvmRoute="tomcat1" name="Catalina" defaultHost="localhost">
{% endhighlight %}

此处的 jvmRoute 名字需要和前面 jkworkers.properties 中配置的名字保持一致。

#### 最后重启 tomcat 和 httpd:

如果出现以下错误：
{% highlight sh %}
[Fri Mar 13 15:24:55 2015] [6879:139689216636896] [error] init_jk: (3366): Initializing shm:/etc/httpd/logs/jk-runtime-status. 6879 errno—13. Load balancing workers will not function roper ly. 
[Fri Mar 13 15:24:55 2015] [6879:139689216636896] [info] init_jk: (3383): mod_jk/1.2.4 initiali zed 
[Fri Mar 13 15:24:55 2015] [6880:139689216636896] [error] init_jk: (3366): Initializing shm:/etc/httpd/logs/jk-runtime-status. 6880 errno—13. Load balancing workers will not function roper ly. 
[Fri Mar 13 15:24:55 2015] [6880:139689216636896] [info] init_jk: (3383): mod_jk/1.2.4 initiali zed 
{% endhighlight %}

需要关闭　SELinux:

{% highlight sh %}
[root@localhost ~]# setenforce 0
{% endhighlight %}