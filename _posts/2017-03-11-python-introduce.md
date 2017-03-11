---
layout: post
title: Python 起源
category : python 
tags : [Python]
---

{% include JB/setup %}

# Python简介                                                                                                     

- 起始时间:1989年圣诞
- 它爸:吉多.范罗苏姆(English Guido van Rossum)
- 起因:它爸无聊打发时间呢，就用双手创造了它，对，大牛就这么猛
- 语言类型:解释型语言 
- 目前语言排名:第五名

# 主要应用领域：

- 系统运维-自动化运维工具开发-例:ansible
- 云计算- 例:openstack
- WEB开发- 例:豆瓣、Youtube 有自己的框架 Django Flask
- 科学计算、人工智能-典型库NumPy, SciPy, Matplotlib, Enthought librarys,pandas
- 金融-量化交易、金融分析、自动化交易
- 图形GUI-PyQT、Wxpython、TkInter
- Python爬虫-网上爬虫、图片爬虫
- 黑客编程-Python有一个hack的库，内置的函数可以作为黑客编程

# Python 优点:

- 简单、易学
- 开发效率高
- 高级的开发语言
- 跨平台可移植性
- 可扩展性，能够使用C或者C++编写的程序接口
- 可嵌入型，可嵌入C/C++ 程序中
- 面向对象：Python既支持面向对象的编程也支持面向对象的编程。

# Python 缺点:

- 速度慢，解释一下 慢归慢除非你要写对速度要求极高的搜索引擎、短时间内并发很大的秒杀程序，那么请您用C去实现
- 代码不能加密，再解释一下 Python是解释型语言，是以名文的形式存放，在这个互联网时代即便是加密型代码，一样可以破解，要的只是时间
- 线程无法利用多CPU，GIL（全局解释器锁）是计算机程序设计语言解释器用于同步线程的工具，使得任何时刻仅有一个线程在执行，Python的线程是操作系统的原生线程。在Linux上为pthread，在Windows上为Win thread，完全由操作系统调度线程的执行。一个python解释器进程内有一条主线程，以及多条用户程序的执行线程。即使在多核CPU平台上，由于 GIL的存在，所以禁止多线程的并行执行。
- 其实Python还有一些其他的缺点，这里要说的语言和人一样，人(语言)无完(语言)人，不要总想着拿自己的优点或者缺点去和别人的优点或缺点比较，这样就没意思了，比如SB能活在这个世界上必有他活在这个世界上的道理，要是没有傻逼，那么也就没有傻逼这个词语了，这也许就是他的道理长处，你能给他比吗？
 

# Python解释器

- CPython 这个解释器是C语言开发的，所以称为 CPython，我们在 Win 和 Linux 命令行中用的就是 CPython，也是我们平时用的最多的解释器。
- IPython 是基于 CPython 之上的一个交互式解释器，也就是说，IPython 只是在交互方式上有所增强，但是执行 Python 代码的功能和 CPython 是完全一样的。好比很多国产浏览器虽然外观不同，但内核其实都是调用了IE。
- PyPy 是一个以速度著称的 Python 解释器，它的目标是执行的速度，对 Python 进行动态编译,采用的 JIT 技术，因为 PyPy 和 CPython 有一些不同，所以代码执行的时候会有不同的结果。
- Jython 是一种完整的语言，而不是一个 Java 翻译器或仅仅是一个 Python 编译器，它是一个Python 语言在 Java 中的完全实现。Jython 也有很多从 CPython 中继承的模块库。最有趣的事情是 Jython不像 CPython 或其他任何高级语言，它提供了对其实现语言的一切存取。所以 Jython 不仅给你提供了 Python 的库，同时也提供了所有的 Java 类。这使其有一个巨大的资源库。
- IronPython 是一种在 .NET 和 Mono 上实现的 Python 语言，由 Jim Hugunin（同时也是 Jython 创造者）所创造，可以直接把 Python 代码编译成.Net的字节码。

# Python发展史

前面已经说过诞生于1989年的圣诞假期，也就是它爹无聊练双手，出来的它 

1991年，第一个Python编译器诞生。它是用C语言实现的，并能够调用C语言的库文件。从一出生，Python已经具有了：类，函数，异常处理，包含表和词典在内的核心数据类型，以及模块为基础的拓展系统。

- Python 1.0 - January 1994 增加了 lambda, map, filter and reduce.
- Python 2.0 - October 16, 2000，加入了内存回收机制，构成了现在Python语言框架的基础
- Python 2.4 - November 30, 2004, 同年目前最流行的WEB框架Django 诞生
- Python 2.5 - September 19, 2006
- Python 2.6 - October 1, 2008
- Python 2.7 - July 3, 2010
- Python 3.0 - December 3, 2008
- Python 3.1 - June 27, 2009
- Python 3.2 - February 20, 2011
- Python 3.3 - September 29, 2012
- Python 3.4 - March 16, 2014
- Python 3.5 - September 13, 2015
- Python 3.6 -  December  23, 2016


