---
layout: post
title: Linux NTFS 挂载目录误删恢复
category : linux 
tags : [Shell, Deploy]
---

{% include JB/setup %}

#### 取消硬盘挂载:
{% highlight sh %}
pi@raspberrypi ~ $ df -h
Filesystem      Size  Used Avail Use% Mounted on
rootfs           15G  5.4G  8.3G  40% /
dev             378M     0  378M   0% /dev
/dev/mmcblk0p2   15G  5.4G  8.3G  40% /mnt
/dev/loop0       15G  5.4G  8.3G  40% /squashfs
none             15G  5.4G  8.3G  40% /
tmpfs            96M  412K   96M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           192M     0  192M   0% /run/shm
/dev/sda1       2.8T  420G  2.4T  16% /samsung
pi@raspberrypi ~ $ sudo umount /samsung
{% endhighlight %}

#### 安装 ntfsprogs:
{% highlight sh %}
pi@raspberrypi ~ $ sudo apt-get install ntfsprogs
{% endhighlight %}

#### 查找待恢复文件：

{% highlight sh %}
pi@raspberrypi ~ $ sudo ntfsundelete /dev/sda1 -S 10m-30m
Inode    Flags  %age  Date           Size  Filename
---------------------------------------------------------------
523      FN..   100%  2016-08-05  25970024  个人资料.zip
529604   FN..   100%  2012-01-20  12277886  struts2-showcase.war

Files with potentially recoverable content: 2
{% endhighlight %}

/dev/sda1 为 df -h 时的盘符
-S 为文件大小参数
结果为找到两个可恢复的文件

#### 文件恢复:

{% highlight sh %}
pi@raspberrypi ~ $  sudo ntfsundelete /dev/sda1 -u -i 523 -o ziliao.zip -d /                                                                                                                                                  
Inode    Flags  %age  Date            Size  Filename
---------------------------------------------------------------
523      FN..     0%  2016-08-05  25970024  个人资料.zip

Undeleted '个人资料.zip' successfully. 
{% endhighlight %}

/dev/sda1 为 df -h 时的盘符
-i 为 查找待恢复文件 时找到的 Inode
-o 为保存的文件名
-d 为保存目录