---
title: flask的中间件
name: flask-middle1
date: 2020-02-12
id: 0
tags: [flask]
categories: []
abstract: ""
---


flask实现的中间件技术

<!--more-->

# 中间件

Middleware中间件，用于在服务器接收web请求时协助处理请求

在Django中自带的中间件包括请求前处理中间件，请求响应中间件

当路由需要特殊的处理要求时会用到中间件

## 示例

### 验证请求

当服务器是一个图床服务器时，为了避免图床的资源被非法引用而导致流量异常，我们可以使用请求中间件在路由请求前判断该请求的referr请求头是否符合规范。

## 设计请求中间件

```python
@app.before_request
def if_auth():
    r = request.url
    print(r)
```

我们设计一个验证中间件，这个中间件在每次请求到来前会打印请求的链接地址

现在假如有一个路由

```python
@app.route('/test')
def test():
    return "ok"
```

当请求/test地址时就会看到输出的结果

```bash
http://127.0.0.1:5000/test
#console
ok
```

## 请求完成中间件

假如有操作是在请求完成后进行的，则使用响应中间件

```python
@app.after_request
def if_auth(response):
    r = request.url
    print(r)
    return response
```

现在在每次响应数据后，就会在终端打印出本次响应的请求地址

### 实际使用

现在考虑一个需求，我们有一个响应超时需要，在指定的时间里一个用户只能访问某个路由10次

自定义一个`limit_count`中间件

```python
@app.before_request
def count():
    r = request.cookies
    flag = check_cookie(r)
    if flag:
        pass
    else:
        return 403
```

每次请求到来时，会先通过中间件进行判断校验一段时间内的访问次数，如果返回flag则可以继续访问该路由，否则返回403状态码

```html
{
    "message": "You don't have the permission to access the requested resource. It is either read-protected or not readable by the server."
}
```

