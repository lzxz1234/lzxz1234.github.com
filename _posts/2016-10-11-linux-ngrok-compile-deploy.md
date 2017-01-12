---
layout: post
title: Linux 反向代理 Ngrok 编译安装
category : linux 
tags : [Shell, Deploy, Ngrok, Linux]
---

{% include JB/setup %}

## Ngrok 是什么
ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。

#### 关于为什么要使用 Ngrok

作为一个Web开发者，我们有时候会需要临时地将一个本地的Web网站部署到外网，以供他人体验评价或协助调试等等，通常我们会这么做：

1. 找到一台运行于外网的Web服务器
2. 服务器上有网站所需要的环境，否则自行搭建
3. 将网站部署到服务器上
4. 调试结束后，再将网站从服务器上删除

但是当有了 Ngrok 之后，世界可以如此美好

1. 下载 ngrok
2. 运行命令 ngrok http 80，80 是你本地Web服务的端口
3. 你会得到一串网址，通过这个网址就可以访问你本地的Web服务了

## 安装步骤
#### 安装 go 环境:

{% highlight sh %}
[root@localhost ~]# wget http://www.golangtc.com/static/go/1.4.2/go1.4.2.linux-386.tar.gz
[root@localhost ~]# tar -zxvf go1.4.2.linux-386.tar.gz
[root@localhost ~]# mv go /usr/local/
[root@localhost ~]# ln -s /usr/local/go/bin/* /usr/bin/
{% endhighlight %}

#### 编译 Ngrok:

{% highlight sh %}
[root@localhost ~]# cd /usr/local/
[root@localhost ~]# git clone https://github.com/lzxz1234/ngrok.git
[root@localhost ~]# export GOPATH=/usr/local/ngrok/
[root@localhost ~]# export NGROK_DOMAIN="ngrok.sample.com"
[root@localhost ~]# cd ngrok
{% endhighlight %}

#### 生成 ssl 证书：

{% highlight sh %}
[root@localhost ~]# openssl genrsa -out rootCA.key 2048
[root@localhost ~]# openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
[root@localhost ~]# openssl genrsa -out server.key 2048
[root@localhost ~]# openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
[root@localhost ~]# openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000
[root@localhost ~]# cp rootCA.pem assets/client/tls/ngrokroot.crt
[root@localhost ~]# cp server.crt assets/server/tls/snakeoil.crt
[root@localhost ~]# cp server.key assets/server/tls/snakeoil.key
{% endhighlight %}

#### 编译服务器：

{% highlight sh %}
[root@localhost ~]# cd /usr/local/go/src
[root@localhost ~]# GOOS=linux GOARCH=386 ./make.bash
[root@localhost ~]# cd /usr/local/ngrok/
[root@localhost ~]# GOOS=linux GOARCH=386 make release-server
{% endhighlight %}

#### 编译 Mac 客户端：

{% highlight sh %}
[root@localhost ~]# cd /usr/local/go/src
[root@localhost ~]# GOOS=darwin GOARCH=amd64 ./make.bash
[root@localhost ~]# cd /usr/local/ngrok/
[root@localhost ~]# GOOS=darwin GOARCH=amd64 make release-client
{% endhighlight %}

#### 编译 Windows 客户端：

{% highlight sh %}
[root@localhost ~]# cd /usr/local/go/src
[root@localhost ~]# GOOS=windows GOARCH=amd64 ./make.bash
[root@localhost ~]# cd /usr/local/ngrok/
[root@localhost ~]# GOOS=windows GOARCH=amd64 make release-client
{% endhighlight %}

#### 编译 Arm 平台客户端：

{% highlight sh %}
[root@localhost ~]# cd /usr/local/go/src
[root@localhost ~]# GOOS=linux GOARCH=arm ./make.bash
[root@localhost ~]# cd /usr/local/ngrok/
[root@localhost ~]# GOOS=linux GOARCH=arm make release-client
{% endhighlight %}

#### 服务端启动：

{% highlight sh %}
/usr/local/ngrok/bin/ngrokd -domain=ngrok.sample.com -httpAddr=":80"
{% endhighlight %}

服务器启动后会访问 /root/sqlite3.db 作为鉴权来源，基本表结构如下：

{% highlight sh %}
[root@localhost ~]# sqlite3 sqlite3.db 
SQLite version 3.6.20
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .schema user_info
CREATE TABLE user_info(id int, name varchar(100), token varchar(100));
{% endhighlight %}

#### 客户端使用：

新建配置文件 ngrok.cfg :
{% highlight yml %}
server_addr: "ngrok.sample.com:4443"
auth_token: 195217068345403292191024
{% endhighlight %}

auth_token 的来源为服务器 sqlite3.db 中的 token 字段
执行：
{% highlight cmd %}
D:\Program Files\Ngrok> ngrok.exe -config=./ngrok.cfg -subdomain=blog 80
{% endhighlight %}

## All Done !