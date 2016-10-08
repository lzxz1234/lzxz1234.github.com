---
layout: post
title: Shell 实现从 SVN 自动更新部署项目
category : Shell
tags : [Shell, Automation]
---
{% include JB/setup %}

### 主要逻辑 ###
对指定目录执行 `svn update`，如果存在新内容，则关闭 tomcat，执行编译操作，然后启动 tomcat，否则直接退出。

### 基本代码 ###
{% highlight bash linenos %}
#!/bin/bash
 
shelldir=$(cd $(dirname $BASH_SOURCE); pwd)
prjhome="$shelldir/web"
tomcathome="/var/apache-tomcat-7.0.54-8080"

webinfdir="$prjhome/WebContent/WEB-INF"
srcdir=("$prjhome/src" "$prjhome/conf")
libdir=("$webinfdir/lib/*.jar" "$tomcathome/lib/*.jar")
dstdir="$webinfdir/classes/"

svninfofile="$prjhome/.revision"
tmpfile="$prjhome/.tmp"

classpath="."
for eachlibdir in ${libdir[@]}
do 
    for f in $eachlibdir
    do
        classpath=$classpath:$f
    done
done
srcpath=""
for eachsrcdir in ${srcdir[@]}
do 
    srcpath="$srcpath $eachsrcdir"
done

echo "==========================================================="
echo ">>>>自动部署开始"
echo ">>>>读取执行目录 ShellDir: $shelldir"
echo ">>>>读取项目主目录 ProjectHome: $prjhome"
echo ">>>>读取 Tomcat 主目录 TomcatHome: $tomcathome"
echo ">>>>读取变量 WEB-INF 目录: $webinfdir"
echo ">>>>读取变量源代码目录: $srcdir"
echo ">>>>读取变量引用JAR包目录: $libdir"
echo ">>>>读取变量编译目标目录: $dstdir"
echo "-----------------------------------------------------------"

if [ -f $svninfofile ]
then
    source $svninfofile
    echo ">>>>更新前SVN版本号：$preversion"
else
    preversion=0
fi
svn update -q --accept postpone $prjhome
echo ">>>>升级SVN完成"
curversion=$(svn info $prjhome | grep Revision | awk -F "[: ]+" '{ print $2 }' )
echo ">>>>更新后SVN版本号：$curversion"
echo "-----------------------------------------------------------"

if [[ $curversion -gt $preversion ]]
then
    echo ">>>>更新到新内容，开始编译"

    find $srcpath -name *.java |grep -v ".svn" > $tmpfile
    javac -g -classpath $classpath -d $dstdir -encoding utf8 @$tmpfile
    echo ">>>>编译JAVA类完成"
 
    find $srcpath |grep -v ".java" |grep -v ".svn" > $tmpfile
    while read line 
    do 
        if [ -f $line ]
        then
            for eachsrcdir in ${srcdir[@]}
            do
                dest=${line/$eachsrcdir/$dstdir}
            done
            cp -f $line $dest
            echo ">>>>执行: cp -f $line $dest"
        fi
    done<$tmpfile

    echo ">>>>拷贝非JAVA类文件完成"
    
    rm -f $tmpfile
    echo "preversion=$curversion" > $svninfofile
else
    echo ">>>>未发现新文件，部署中止"
fi
echo "==========================================================="
{% endhighlight %}


