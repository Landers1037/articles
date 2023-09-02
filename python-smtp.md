---
title: python使用smtp邮件服务
name: python-smtp
date: 2019-05-24
id: 0
tags: [python]
categories: []
abstract: ""
---


python3.7实现的smtp邮件系统服务
在python3.*里，提供了基础的邮件服务，由`smtplib`库和`email`实现

<!--more-->

我们导入需要的库，创建邮件服务

```python
import smtplib
import schedule
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.header import Header
import time

# 设置smtplib所需的参数
# 下面的发件人，收件人是用于邮件传输的。
smtpserver = 'smtp.163.com'
username = 'name@163.com'
password = '123456'
sender = 'name@163.com'
# receiver='XXX@126.com'
# 收件人为多个收件人
# receiver = ['vvv@qq.com', 'vvv@qq.com']

subject = 'python email test'
# 通过Header对象编码的文本，包含utf-8编码信息和Base64编码信息。以下中文名测试ok
#subject = '中文标题'
#subject=Header(subject, 'utf-8').encode()

# 构造邮件对象MIMEMultipart对象
# 下面的主题，发件人，收件人，日期是显示在邮件页面上的。
msg = MIMEMultipart('mixed')
msg['Subject'] = subject
msg['From'] = 'name@163.com <name@163.com>'
#msg['To'] = 'XXX@126.com'
# 收件人为多个收件人,通过join将列表转换为以;为间隔的字符串
msg['To'] = ";".join(receiver)
# msg['Date']='2019-5-24'

# 构造文字内容
text = "快点起床,要努力学习,已经九点了,求求你快点起床，都要胖成猪了"
text_plain = MIMEText(text, 'plain', 'utf-8')
msg.attach(text_plain)

# 构造图片链接
# sendimagefile = open(r'D:\testimage.jpg', 'rb').read()
# image = MIMEImage(sendimagefile)
# image.add_header('Content-ID', '<image1>')
# image["Content-Disposition"] = 'attachment; filename="testimage.jpg"'
# msg.attach(image)

# #构造html
# #发送正文中的图片:由于包含未被许可的信息，网易邮箱定义为垃圾邮件，报554 DT:SPM ：<p><img src="cid:image1"></p>
# html = """
# <html>
#   <head></head>
#   <body>
#     <p>Hi!<br>
#        How are you?<br>
#        Here is the <a href="http://www.baidu.com">link</a> you wanted.<br>
#     </p>
#   </body>
# </html>
# """
# text_html = MIMEText(html,'html', 'utf-8')
# text_html["Content-Disposition"] = 'attachment; filename="texthtml.html"'
# msg.attach(text_html)


# #构造附件
# sendfile=open(r'D:\pythontest\1111.txt','rb').read()
# text_att = MIMEText(sendfile, 'base64', 'utf-8')
# text_att["Content-Type"] = 'application/octet-stream'
# #以下附件可以重命名成aaa.txt
# #text_att["Content-Disposition"] = 'attachment; filename="aaa.txt"'
# #另一种实现方式
# text_att.add_header('Content-Disposition', 'attachment', filename='aaa.txt')
# #以下中文测试不ok
# #text_att["Content-Disposition"] = u'attachment; filename="中文附件.txt"'.decode('utf-8')
# msg.attach(text_att)

# 发送邮件
def email():
    smtp = smtplib.SMTP()
    smtp.connect('smtp.163.com')
    # 我们用set_debuglevel(1)就可以打印出和SMTP服务器交互的所有信息。
    # smtp.set_debuglevel(1)
    smtp.login(username, password)
    smtp.sendmail(sender, receiver, msg.as_string())
    smtp.quit()
email()
```

我们实现了邮件的文字，图片和附件的发送，构造html页面
<font color=crimson>因为发送图片，附件时，服务器没有许可认证，使用网易的smtp服务器会报554错误，被认定为垃圾邮件</font>

修改为简单的文本内容发送

```python
#coding: utf-8
import smtplib
import schedule
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.header import Header
import time

smtpserver = 'smtp.163.com'
username = '@163.com'
password = '123456'
sender = '@163.com'
# receiver='XXX@126.com'
receiver = ['@qq.com']

subject = 'test'
msg = MIMEMultipart('mixed')
msg['Subject'] = subject
msg['From'] = '@163.com <@163.com>'
#msg['To'] = 'XXX@126.com'
msg['To'] = ";".join(receiver)
text = "hello from py"
text_plain = MIMEText(text, 'plain', 'utf-8')
msg.attach(text_plain)


def email():
    smtp = smtplib.SMTP()
    smtp.connect('smtp.163.com')
    smtp.login(username, password)
    smtp.sendmail(sender, receiver, msg.as_string())
    smtp.quit()

email()
```

如果再次遇到垃圾邮件错误提醒，检查`subject`是否过短，邮件标题还是要正式一点

<font size=4>问题</font>

<font color=crimson>在Windows下测试可以发送接收邮件，在linux服务器下测试失败，一直卡住？</font>

linux云服务器大多没有开启smtp服务端口，默认25，阿里云无法开启，默认关闭。
解决办法改为ssl传输，更改端口为465，不行换成587

```python
smtp = smtplib.SMTP_SSL()
smtp.connect('smtp.163.com',465)
smtp.login(username, password)
#或者这样写
smtp = smtplib.SMTP_SSL('smtp.163.com',465)
smtp.login(username, password)
```

在防火墙里开启对应的465端口

<font color=crimson>smtp连接超时</font>

检查自己的邮箱是否开启smtp服务，如果是网易邮箱，登录邮箱后，进入账户设置开启smtp,pop3

<font color=crimson>密码正确，登录失败</font>

大部分邮箱开启保护，不能使用其他非认证客户端登录，所以需要申请授权码作为登录密码，具体百度