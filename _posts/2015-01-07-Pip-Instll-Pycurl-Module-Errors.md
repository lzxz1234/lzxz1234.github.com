---
layout: post
title: 记 pycurl 安装过程中的错误
category : python
tags : [Python, pip]
---
{% include JB/setup %}

## 错误一 ##

{% highlight sh %}
In file included from src/module.c:1:
src/pycurl.h:152:5: warning: #warning "libcurl was compiled with SSL support, but configure could not determine which " "library was used; thus no SSL crypto locking callbacks will be set, which may " "cause random crashes on SSL requests"
src/module.c: In function ‘initpycurl’:
src/module.c:723: error: ‘CURLPROTO_IMAP’ undeclared (first use in this function)
src/module.c:723: error: (Each undeclared identifier is reported only once
src/module.c:723: error: for each function it appears in.)
src/module.c:724: error: ‘CURLPROTO_IMAPS’ undeclared (first use in this function)
src/module.c:725: error: ‘CURLPROTO_POP3’ undeclared (first use in this function) src/module.c:726: error: ‘CURLPROTO_POP3S’ undeclared (first use in this function)
src/module.c:727: error: ‘CURLPROTO_SMTP’ undeclared (first use in this function)
src/module.c:728: error: ‘CURLPROTO_SMTPS’ undeclared (first use in this function)
src/module.c:729: error: ‘CURLPROTO_RTSP’ undeclared (first use in this function) src/module.c:730: error: ‘CURLPROTO_RTMP’ undeclared (first use in this function)
src/module.c:731: error: ‘CURLPROTO_RTMPT’ undeclared (first use in this function)
src/module.c:732: error: ‘CURLPROTO_RTMPE’ undeclared (first use in this function)
src/module.c:733: error: ‘CURLPROTO_RTMPTE’ undeclared (first use in this function)
src/module.c:734: error: ‘CURLPROTO_RTMPS’ undeclared (first use in this function)
src/module.c:735: error: ‘CURLPROTO_RTMPTS’ undeclared (first use in this function)
src/module.c:736: error: ‘CURLPROTO_GOPHER’ undeclared (first use in this function)
error: command 'gcc' failed with exit status 1
{% endhighlight %}

**解决办法：**

1. 直接上http://curl.haxx.se/download.html下载最新版本的curl源码。
2. 解压curl
3. 安装curl ./configure --disable-shared make make install
4. 安装pycurl pip install pycurl

## 错误二 ##

{% highlight sh %}
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: pycurl: libcurl link-time version (7.19.7) is older than compile-time version (7.37.0)
{% endhighlight %}

**解决办法:**

1. 修改环境变量即可：export LD_LIBRARY_PATH=/usr/local/lib

## For Windows ##

此处 [下载](http://www.lfd.uci.edu/~gohlke/pythonlibs/ "pythonlibs") 对应包安装。

{% highlight sh %}
D:\Python27\Lib\site-packages>pip wheel pycurl-7.19.5.1-cp27-none-win_amd64.whl
D:\Python27\Lib\site-packages>pip install --no-index --find-links=wheelhouse pycurl-7.19.5.1-cp27-none-win_amd64.whl
{% endhighlight %}


