---
layout: post
title: Django 启动错误
category : python
tags : [Python, Django]
---
{% include JB/setup %}

错误信息：

	F:\Workspaces\Python\proxy>python manage.py runserver
	Performing system checks...
	
	System check identified no issues (0 silenced).
	October 20, 2014 - 11:31:35
	Django version 1.7, using settings 'proxy.settings'
	Starting development server at http://127.0.0.1:8000/
	Quit the server with CTRL-BREAK.
	Unhandled exception in thread started by <function check_errors.<locals>.wrapper
	 at 0x0000000003F0DD90>
	Traceback (most recent call last):
	  File "D:\Python34\lib\site-packages\django\utils\autoreload.py", line 222, in
	wrapper
	    fn(*args, **kwargs)
	  File "D:\Python34\lib\site-packages\django\core\management\commands\runserver.
	py", line 134, in inner_run
	    ipv6=self.use_ipv6, threading=threading)
	  File "D:\Python34\lib\site-packages\django\core\servers\basehttp.py", line 165
	, in run
	    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
	  File "D:\Python34\lib\site-packages\django\core\servers\basehttp.py", line 117
	, in __init__
	    super(WSGIServer, self).__init__(*args, **kwargs)
	  File "D:\Python34\lib\socketserver.py", line 429, in __init__
	    self.server_bind()
	  File "D:\Python34\lib\site-packages\django\core\servers\basehttp.py", line 121
	, in server_bind
	    super(WSGIServer, self).server_bind()
	  File "D:\Python34\lib\wsgiref\simple_server.py", line 50, in server_bind
	    HTTPServer.server_bind(self)
	  File "D:\Python34\lib\http\server.py", line 135, in server_bind
	    self.server_name = socket.getfqdn(host)
	  File "D:\Python34\lib\socket.py", line 460, in getfqdn
	    hostname, aliases, ipaddrs = gethostbyaddr(name)
	UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe5 in position 0: invalid
	continuation byte


居然是计算机名中含有中文导致的，整一大杯具


转载注明出处：[{{page.title}}]({{permalink}})
