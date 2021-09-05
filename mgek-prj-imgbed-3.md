---
title: mgek图床项目第三期
name: mgek-prj-imgbed-3
date: 2020-06-18
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第三期

路由设计2 登录认证


<!--more-->


**Mgek项目**-ImgBed第三期

路由设计2 登录认证

<!--more-->

##  登录认证路由

位于包`api/auth`下

1. 数据库初始化(仅在本地测试环境下使用)
2. 用户登录获取token令牌
3. 用户token找回服务
4. 用户注销服务

### 路由蓝图

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: __init__.py
# @Date: 2020-05-16

#用于认证用户的api接口

from flask import Blueprint
from flask_cors import CORS


auth = Blueprint('auth_api', __name__)
CORS(auth)

from . import get_token
from . import db_init
from . import mail_notify
from . import remove_token
```

设计名为`auth`的蓝图

### 数据库初始化

仅在本地测试时使用，用于初始化生成sqlite的数据库文件

### 用户注册与登录

默认的用户使用**邮箱**作为唯一凭证，所以在用户注册时应该对邮箱的合法性进行校验

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: get_token.py
# @Date: 2020-05-16

from app.middleware.jwt_middleware import genernate
from app.api.auth import auth
from flask import jsonify,g,request
from app import global_config

from app.database import database

@auth.route('/api/get_token',methods=['POST'])
def get_token():
    mail = request.json["mail"]
    g.token = genernate(mail)
    """
    在这里添加保存至数据库的函数
    注意如果原本账户存在 不能更新只能找回
    """
    ifexist = database().get_token(global_config.engine,mail)
    if not ifexist:
        res = database().set(global_config.engine,'token',{"mail": mail,"token": g.token})
        if res:
            return jsonify({"token": g.token})
        else:
            return jsonify({"token": ''})

    else:
        return jsonify({"token": 'already exist'})
```

在每次生成token时都会将新的token保存至数据库中，在访问需要验证的api时作为验证

可以在`setting`包内设置token的有效期，默认为3600秒

> ### 注意：这里没有做用户注册判定，只要输入合法的邮箱，就会生成唯一的token用于登录
>
> 新的邮箱将会加入数据库中

### 用户找回token

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: mail_notify.py
# @Date: 2020-05-17

#用于找回信息的邮件通知服务
from app.api.auth import auth
from app.utils import format_response,mail_to
from flask import request

@auth.route('/api/mail_for_reset',methods=['POST'])
def mail_for_reset():
    #获取用户的mail地址 给目标用户发送找回邮件
    address = request.json["mail"]
    if mail_to(address):
        return format_response('ok', '找回邮件已发送')
    else:
        return format_response('error', '请求找回错误')
```

找回服务使用邮件方式会发送到指定的邮箱内，邮件内容中包含**找回链接**

自定义找回方式和邮件内容请参考`utils/mail`包

### 用户注销

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: remove_token.py
# @Date: 2020-05-17

#用户注销账户的接口

from app.api.auth import auth
from flask import request
from app.utils import format_response
from app import global_config
from app.database import database

@auth.route('/api/remove_token')
def remove_token():
    #支持两种判断方式 输入邮箱token后验证
    #通过邮件里的check标志验证
    mail =request.args.get('mail')
    token = request.args.get('token')
    check = request.args.get('check')

    if mail and token:
        res = database().remove_token(global_config.engine,{"mail":mail,"token":token})
        if res:
            return format_response('ok', '密钥删除成功')
        else:
           return format_response('error', '密钥删除失败')

    elif mail and check:
        res = database().remove_token(global_config.engine,
                               {"mail": mail, "check": check})
        if res:
            return format_response('ok', '密钥删除成功')
        else:
           return format_response('error', '密钥删除失败')
     
    else:
        return format_response('error', '密钥删除失败,信息不全')
```

注销的本质是从数据库内删除指定邮箱，这里并没有删除对应邮箱账户下的文件

如需自定义请添加内容

提供两种注销方式：

- 邮箱+token
- 邮箱+check码

check码是通过`utils/mail`包生成的，是跟随用户找回邮件一同发送的，如果邮箱接受者认为该账户不应存在于本服务器上，可以通过包含check码的链接注销账户。

**以上便完成了登录认证的三大功能**

默认是没有用户注销的，考虑到`token`默认有3600s的有效期，过期后需要重新获取。如需保证token唯一且永久请在`setting`包中设置token的`expires`为`false`