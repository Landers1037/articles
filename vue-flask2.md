---
title: flask实现token认证
name: vue-flask2
date: 2020-01-12
id: 0
tags: [flask]
categories: []
abstract: ""
---


在前后端分离的项目中，要实现登录状态可以使用session或token请求头的方式

由于flask-restful是无状态的api，所以不能简单的记住状态信息保存登录状态，于是不能使用flask-login模块。

<!--more-->

使用token请求头的方式完成登录验证

## 依赖

vuex，flask-httpauth，itsdangerous

## 实现原理

使用flask-httpauth完成简单的http认证，从请求头header里提取出`token`，对token进行验证，如果token存在于服务器内存中且没有过期则认定登录

## 源码剖析

```python
class HTTPTokenAuth(HTTPAuth):
    def __init__(self, scheme='Bearer', realm=None):
        super(HTTPTokenAuth, self).__init__(scheme, realm)

        self.verify_token_callback = None

    def verify_token(self, f):
        self.verify_token_callback = f
        return f

    def authenticate(self, auth, stored_password):
        if auth:
            token = auth['token']
        else:
            token = ""
        if self.verify_token_callback:
            return self.verify_token_callback(token)
        return False
```

HTTPTokenAuth类继承自HTTPAuth类，实现其基本的认证方法，自定义了`verify_token`函数当token认证通过返回`true`

## 设计

**test_blueprint.py**

```python
httpauth = HTTPTokenAuth()

@httpauth.verify_token
def verify_token(token):
    try:
         if verify_auth_token(token):
             return True
         else:
             return False
    except:
        return False
    
class Testapi(Resource):
    method_decorators = {
        'get':[httpauth.login_required],
        'post':[httpauth.login_required]
    }
    def post(self):    
        ...
```

`@httpauth.verify_token`使用装饰器在每次请求到来前验证token，如果通过则返回`true`。

`method_decorators`即restful-api提供的添加装饰器的方法，添加了`login_required`装饰器，在每次访问api时都会调用`verify_token`函数验证登录状态

### token

我们在http header里自定义一个`key`值为`token`，其header类似下表

| KEY          | VALUE            |
| ------------ | ---------------- |
| Content-Type | application/json |
| token        | 1234567          |

获取token：

```python
token = request.headers["Token"]
```

## 拓展

### session中的信息

```python
from flask import Flask,Response,session
```

设置session

```python
session['username'] = 'root'
```

获取session

```python
username = session.get('username')
```

清除session

```python
 session.pop('username')  
 session.clear() //删除所有session
```



### cookie中的信息

我们可以通过浏览器获取cookie信息

```python
cookie = request.cookies
```

获取cookie里的值

```python
request.cookies.get('name')
```

设置cookie

```python
from flask import make_response
resp = make_response('set_cookie')
resp.set_cookie('pass', '123456')
```

删除cookie

```python
response.delete_cookie('name')
```

