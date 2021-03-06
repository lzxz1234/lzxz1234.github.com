---
layout: post
title: XCMS简介
category : xcms
tags : [XCMS]
---
{% include JB/setup %}
基于Nutz和Beetl的通用[CMS](https://github.com/lzxz1234/XCMS "XCMS")基本完工。

这个版本是在四维版本的基础上改的，删除了配置文件直接采用模板当配置文件，更强大也更直观，取消系统级的多表联查，仅包含基本的单表增删改查。

这也要求系统在设计时对于数据库的继承关系尽量使每个具体类一张表，即将对象User， Teacher， Student 设计成表 Teacher 和 Student，User中的全部字段均被包含在具体表中。

## 项目地址 ##
Fork on [Github](https://github.com/lzxz1234/XCMS).

## 可用页面列表 ##
- http://domain.com/ 列表页面
- http://domain.com/get/{domainSymbol}/{id} 详情页面
- http://domain.com/add-form/{domainSymbol} 新建记录表单页面
- http://domain.com/add/{domainSymbol} 执行添加记录操作，一般为表单提交用
- http://domain.com/mod-form/{domainSymbol}/{id} 修改记录表单页面
- http://domain.com/mod/{domainSymbol}/{id} 修改记录操作，一般为表单提交用
- http://domain.com/del/{domainSymbol}/{id} 删除记录
- http://domain.com/qry/{domainSymbol} 查询页面

> {domainSymbol} 为实体类在网页上的标志符<br>
> {id} 为数据记录在数据库中的主键

## 基本流程 ##
![流程图]({{ site.JB.POST_IMG_PATH }}/20140606124220.png)

## 系统定制 ##

### AbstractDao ###
数据库操作实体类，部分类的数据库操作需要有单表之外的操作时，新建该类的子类并在class2dao.properties中指定映射关系。

### 系统样式 ###
系统初次获取请求时会在 TplRepository 中创建对应操作的基本模板，如对 book 的查询操作会触发在 Repository 中创建文件 com.siwei.domain.Book-QryResult.html，进行界面微调时直接修改此处模板即可，删除操作会在下次请求到达时重新生成。

在包 `com.chineseall.xcms.nb.tpl` 中包含 create.html, index.html, info.html, modify.html 和 query-result.html，此模板作用为当TplRepository中找不到对应模板时该模板生成，修改整个系统的界面时修改此处模板。

### 快捷更新 ###
**自定义SQL** 支持虚拟类型以实现自定义SQL快速更新少量字段，基本配置如下：

	class2Dao.properties:
	quick-set-workflow=com.chineseall.xcms.nb.dao.QuickSetDao
	custsql.properties:
	quick-set-workflow=update s_resource set status = ${status} where id = ${id}
	
配置完成浏览器直接访问：http://domain.com/mod/quick-set-workflow/26?id=26&status=1 即可。

### 模板语法 ###
模板基本语法参考 [Beetl](https://github.com/javamonkey/beetl2.0 "Beetl")。

查询模板特例：

    <input id='S_id' name='_EQ_id' type='text' value='${_EQ_id !}' placeholder="id">

name 属性前半部分可以指定后台具体查询操作

- \_GT_ GreateThan
- \_LT_ LittleThan
- \_EQ_ Equal
- \_LK_ Like 
- \_ASC_ 非查询，用于排序，此时value随便，排序列为[ASC]后紧跟部分
- \_DSC_ 非查询，用于排序，此时value随便，排序列为[DSC]后紧跟部分

