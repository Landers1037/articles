---
title: mgek图床项目第五期
name: mgek-prj-imgbed-5
date: 2020-06-19
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第五期

utils工具包设计


<!--more-->


**Mgek项目**-ImgBed第五期

utils工具包设计

<!--more-->

Utils工具包包含本项目需要额外使用的功能函数

位于`app/utils`下

### 包初始化

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: __init__.py.py
# @Date: 2020-05-14

# mgek_imgbed utils

from . import rename
from .format_response import format_response
from .count_page import count_page
from .generate_time import generate_time
from .mail import mail_to
```

### 文件命名

`utils/rename.py`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: rename.py
# @Date: 2020-05-14

# 核心的防止重复命名函数，使用两种方式为文件命名
# 使用base64编码图片的方式和md5hash方式

# 通过配置文件选择重命名方式，默认为b64

from app import global_config
from flask import jsonify
from base64 import b64encode
from hashlib import md5
import time


def rename(file_name: str):
    rule = global_config.image_rule
    time_now = str(time.time())
    file_name = file_name.lower() #小写处理
    if rule == 'base64':
        ext = access_ext(file_name)
        return b64encode((file_name + time_now).encode('utf-8')).decode('utf-8') + ext

    elif rule == 'md5':
        ext = access_ext(file_name)
        new = md5()
        new.update((file_name + time_now).encode('utf-8'))
        return new.hexdigest() + ext

    else:
        # using default base64
        ext = access_ext(file_name)
        return b64encode((file_name + time_now).encode('utf-8')).decode('utf-8') + ext

def access_ext(name):
    #用于扩展名称的获取
    ALLOW_SET = ['jpg', 'jpeg', 'png', 'webp', 'gif']
    li = name.split('.')
    if  len(li) > 1 and li[-1] in ALLOW_SET:
        return "." + li[-1] #返回扩展名

    else:
        #不合法的文件默认保存为无扩展的文件
        #还需要在前端做文件类型的判断
        return ''
```

根据`rule`定义的命名方式来命名保存的文件名，支持md5和base64

### 日期格式化

`utils/generate_time`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: generate_time.py
# @Date: 2020-05-16

#图片添加日期的处理函数

import time


def generate_time():
    local = int(time.time())
    local_format = time.strftime('%Y-%m-%d', time.localtime(local))

    return [local,local_format]
```

### JSON响应格式化

`utils/format_response`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: format_response.py
# @Date: 2020-05-14

#用于格式化需要返回的数据JSON

from flask import jsonify

def format_response(type: str,msg):
    """
    type: 返回数据的类型，应该分为error，forbidden，ok
    """
    return jsonify({
        "type": type,
        "data": msg
    })
```

传入两个参数，响应；类型和数据，数据格式为Python支持的字典或列表格式

### 邮件发送

`utils/mail`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: mail.py
# @Date: 2020-05-17

#发送邮件函数
import smtplib,time
from email.mime.text import MIMEText
from email.header import Header
from flask import current_app

from app import global_config
from app.database import database

def mail_to(address):
    HOST = current_app.config["MAIL_HOST"]
    PORT = current_app.config['MAIL_PORT']
    USER = current_app.config['MAIL_USER']
    PASS = current_app.config['MAIL_PASS']

    sender = USER
    receiver = address

    t = database().get_token(global_config.engine,address) 
    if t:
        token = t
        #生成check标志

        link = get_check(address)
        message = MIMEText('请妥善保管你的认证密钥\n{}，如果你的邮箱没有在本站点注册很抱歉打扰您，请点击以下链接删除您的账户,{}'.format(token,link), 'plain', 'utf-8')
        message['From'] = sender
        message['To'] = address

        subject = 'Mgek_ImgBed认证密钥找回服务'
        message['Subject'] = Header(subject, 'utf-8')
        try:
            smtp = smtplib.SMTP_SSL(host=HOST,port=PORT)
            smtp.login(USER,PASS)
            smtp.sendmail(sender,receiver,message.as_string())
            smtp.quit()
            return True

        except Exception as e:
            print(e.args)
            return False
    else:
        return False     


def get_check(address):
    #生成临时check标志，针对本身不存在的账户生成空信息
    #存在
    check = str(time.time())
    database().update_token(global_config.engine,{"address": address,"check": check})
        
    #请在地址栏填写本站点运行的域名，或者在flask_config里修改server_name参数
    server_name = current_app.config['SERVER_NAME'] if current_app.config['SERVER_NAME'] else 'localhost:{}'.format(
            global_config.port)
            
    return 'http://{}/api/remove_token?mail={}&check={}'.format(server_name,address,check)
```

发送找回token的邮件以及注销账户的链接

自定义`server_name`参数为你的域名以保证找回链接可以正常工作

默认的邮件服务器为外部邮件服务器，使用**SSL**端口`465`，如需自定义请在`config/flask_config`中更改