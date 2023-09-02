---
title: Open API的保护解决方法探究
name: solution-open-api-auth
date: 2020-08-07
id: 0
tags: [python]
categories: []
abstract: ""
---


公共API接口防止滥用的解决方法探究

<!--more-->

对于使用量较小,公共的接口还是有保护的需要的.

为了保证服务器的健壮性,对接口的使用策略进行限制很有必要

> 对于需要验证的接口有AuthToken,Session等方式进行验证

## 需求

频率限制,对用户的接口访问频率进行限制

匿名用户限制

字段校验

时间戳校验

### 频率限制

利用反向代理服务的IP查询记录功能记录来访的IP地址作为限制的条件

维护一个全局的**字典**,键为IP地址,值为请求频率

当用户的请求频率过快超出限制时,就会触发禁用策略

```python
def reach_limit():
    #根据ip的频率限制,根据时间戳限制
    timestamp = request.args.get('timestamp')
    ip_address = request.remote_addr
    global limit_dict
    TIME_WAIT = current_app.config["TIME_WAIT"]
    TIME_PENDING = current_app.config["TIME_PENDING"]
    LIMIT_TIME = current_app.config["LIMIT_TIME"]
    LIMIT_FREQ = current_app.config["LIMIT_FREQ"]

    if timestamp:
        pass
    else:
        now = time.time()
        if limit_dict.get(ip_address):
            if limit_dict[ip_address]["count"] < LIMIT_FREQ:
                #是否进入等待期
                if now - limit_dict[ip_address]["mod"] < 0:
                    #等待期内直接禁止
                    return True
                else:
                    #等待期过 开始重新统计
                    limit_dict[ip_address]["count"] += 1
                    return False

            else:
                #判断时间差
                #是否已经被限制
                if now - limit_dict[ip_address]["mod"] < LIMIT_TIME:
                    limit_dict[ip_address]["count"] = 0
                    limit_dict[ip_address]["mod"] = now + TIME_WAIT
                    return True
                else:
                    limit_dict[ip_address]["count"] = 0
                    limit_dict[ip_address]["mod"] = now + TIME_PENDING
                    return False

        else:
            limit_dict[ip_address] = {"count": 1, "mod":now}
            return False

    return True
```

`limit_dict`为全局维护的名单字典，当新用户到来时会将其加入名单中

`count`为请求次数，`mod`为最后请求时间

当最后请求时间计算正确时表明没有触发禁用，`count`值递增。当超出请求次数后最后请求时间会被计算为**下一次允许请求的时间**，接下来的所有请求都会被禁用直到达到允许请求的时间

这只是一个简单的按频率屏蔽的思路，真实的情况远比这复杂

### 匿名用户

对于网络测试工具发出的请求，其`referrer`一般为空或者`127.0.0.1`

根据`referrer`判断请求是否发自互联网上的服务器，有效地屏蔽掉大部分无效请求

```python
def is_referrer_in(referrer):
    if referrer:
        for r in current_app.config["ALLOW_REFERRER"]:
            if r in referrer:
                return True
    else:
        return False
```

### 防爬虫User-Agent

对请求的请求头`User-Agent`进行判断，将需要禁止的`User-Agent`放置在列表里进行比较

对于正规的爬虫其`User-Agent`命名规范，很容易做到区分并禁止

### 字段校验

公共的接口其`Uri`一般都会暴露直接给出，为避免爬虫等的无效请求，可以附带字段校验

添加`?check_name=checked`或者在**POST**数据中添加

静态的字段往往不够安全，可以使用加密方式SHA1,MD5等生成随机hash串

### 时间戳

时间戳是保证接口限时有效的一个好方法

由客户端生成时间戳，传递到后台后，后台与当前的时间进行对比。如果相差的时间在规定的范围内就视为有效请求

请求类似`?timestrmp=1111111111`

<hr>

通常情况下，需要多种方式混合使用来保证接口的安全性

