---
title: 随笔-跨域请求
name: sb
date: 2019-08-16
id: 0
tags: [flask]
categories: []
abstract: ""
---


今天尝试用码云做个cdn缓存，把字体文件托管在码云，在css中引用，结果就遇到了CORS问题


<!--more-->


今天尝试用码云做个cdn缓存，把字体文件托管在码云，在css中引用，结果就遇到了CORS问题

<!--more-->

因为css中引用的外部字体和css所在域不同，浏览器直接禁止了我访问字体下载，Google了半天，使用Flask-cors插件为链接请求添加跨域请求头的方式试了半天，一开始我以为是因为我使用蓝图的原因，cors类没有初始化，后来发现就是在单独的app.py环境下配置依旧不行。

于是换了一种办法，因为前面使用的Nginx代理请求，所以直接配置了nginx的location参数，直接写了

```nginx
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
```

但是还是失败，难道是因为nginx+gunicorn+flask不能这样配置吗？

最后没有办法我以为是码云的文件不能这样引入，于是用chrome的debug模式忽略了跨域请求直接在本地测试。结果TM居然成功了，可以直接访问这个简易的cdn文件，问题仍未解决先了解下是不是flask的问题吧。