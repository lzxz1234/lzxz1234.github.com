---
layout: post
title: Jekyll 使用过程中碰到的一些问题
category : Jekyll
tags : [Jekyll， Problems]
---

## 关于 Python 版本 ##

不要尝试 3.x，如果装了 3.x，卸载重装2.7 吧。

## gem install jekyll 没反应 ##

使用淘宝的镜像站

	$ gem sources --remove https://rubygems.org/
	$ gem sources -a http://ruby.taobao.org/
	$ gem install jekyll

## warning:cannot close fd before spawn ##

	$ gem list --local
	$ gem uninstall pygments.rb --version "=0.5.x"
	$ gem install pygments.rb --version "=0.5.0"