---
layout: post
title: Python 脚本通过邮件自动报告外网IP
category : Python
tags : [Python, Cubieboard, Email]
---
{% include JB/setup %}

有台Cubieboard需要在外网访问，花生壳动态域名不稳定，所以就动手写了个外网IP报告脚本。

基本流程，启动时探测本机外网IP，并与本目录下的 ipFile 中记载 IP 进行比较，如果 ipFile 不存在或者两边 IP 
不一致，那么通过 SMTP 向指定邮箱列表发送邮件。

源代码如下：

{% highlight python linenos %}
#!/usr/bin/python
# -*- coding: utf-8 -*-
import os
import urllib2
import smtplib
import datetime

from email.mime.text import MIMEText

identifier='Cubieboard2'

ipFile='./.lastip' #IP记录文件
ipAddressLookUpUrl='http://utilities.duapp.com/ip' #IP 查询地址

mailto_list=['xxxxxxx@qq.com'] #通知目标邮箱
mail_host='smtp.qq.com'
mail_user='xxxxxxx'
mail_pass='xxxxxxx'
mail_postfix='qq.com'

def send_mail(dstMail, subject, content):
    me = mail_user+'<'+mail_user+'@'+mail_postfix+'>'
    msg = MIMEText(content,_subtype='html',_charset='utf-8')
    msg['Subject']=subject
    msg['From']=me
    msg['To']=';'.join(dstMail)
    try:
        server = smtplib.SMTP_SSL() #QQ邮箱是 SSL 加密的，其它邮箱可能为 smtplib.SMTP()
        server.connect(mail_host)
        server.login(mail_user, mail_pass)
        server.sendmail(me, dstMail, msg.as_string())
        server.quit()
        server.close()
        return True
    except Exception, e:
        return False

def getIP():
    urlOpener = urllib2.build_opener()
    urlOpener.addheaders = [('User-agent','Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/6.0)'),
                            ('Accept-Language', 'zh-CN'),
                            ('Cache-Control', 'no-cache')]
    request = urllib2.Request(ipAddressLookUpUrl)
    return urlOpener.open(request).read();

def previousIP():
    if(os.path.exists(ipFile)):
        return open(ipFile, 'r').read()
    else:
        return ""

def saveIP(ip):
    filePointer = open(ipFile, 'w')
    filePointer.write(ip)
    filePointer.flush()
    filePointer.close()

def buildHtmlIPInfo(nowIP):
    return '''<html>
    <body>
        <h1>{}</h1>
        <p>目前IP: {}</p>
    </body>
</html>'''.format(identifier, nowIP)

if __name__ == '__main__':
    try :
        previousIP=previousIP().strip()
        nowIP=getIP().strip()
        if nowIP != previousIP:
            if send_mail(mailto_list, str(datetime.date.today())+' - '+identifier+'地址更新', buildHtmlIPInfo(nowIP)):
                saveIP(nowIP)
    except Exception, e:
        pass
{% endhighlight %}

脚本完成后将其添加到系统定时任务：

	*/5 * * * * /usr/bin/python /home/linaro/report_ip_via_email.py

转载注明出处：[{{page.title}}]({{permalink}})