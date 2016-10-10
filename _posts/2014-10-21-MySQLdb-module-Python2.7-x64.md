---
layout: post
title: Python2.7 64位版 MySQLdb 模块下载
category : python
tags : [MysqlDB, Python, x64]
---
{% include JB/setup %}

`pip install mysqldb` 无限失败，不是少这就是少那的，无奈找了个现成版。

1. 下载链接：
[MySQL-python-1.2.3.win-amd64-py2.7](http://www.lfd.uci.edu/~gohlke/pythonlibs/ "MySQL-python-1.2.3.win-amd64-py2.7")

2. 安装运行
3. 检查是否安装成功:

如果安装成功,将没有任何提示,如下

		>>> import MySQLdb
		>>>
		
安装不成功的提示:

		>>> import MySQLdb
		Traceback (most recent call last):
		  File "<stdin>", line 1, in <module>
		ImportError: No module named MySQLdb
		>>>


