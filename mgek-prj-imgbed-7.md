---
title: mgek图床项目第七期
name: mgek-prj-imgbed-7
date: 2020-06-19
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第七期

运行测试


<!--more-->


**Mgek项目**-ImgBed第七期

运行测试

<!--more-->

服务运行时应当部署在WSGI服务器内部保证运行的高效

本地运行测试可以使用gevent提供的内置WSGI Server实现并发测试

### 运行文件

`/img_server.py`

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: img_server.py
# @Date: 2020-05-12

#基于rest的图床api测试版
#基于纯Flask实现

from app import create_app
from gevent.pywsgi import WSGIServer

application = create_app(mode='dev')

if __name__ == '__main__':
    #基于运行模式提供基于gevent的异步支持
    from app import global_config
    run_mode = global_config.multiroutine
    if run_mode:
        #using gevent
        http_server = WSGIServer(('',global_config.port),application)
        http_server.serve_forever()

    else:
        #usually using in dev    
        application.run(port=global_config.port)
```

### 初始化应用

`app/__init__.py`

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: __init__.py.py
# @Date: 2020-05-12

from flask import Flask
from app.config import *
from flask_sqlalchemy import SQLAlchemy
from flask_pymongo import PyMongo
from werkzeug.middleware.proxy_fix import ProxyFix

#初始时会默认初始化数据库连接，根据engine的配置选择配置的数据库
db = SQLAlchemy()
mongo = PyMongo()
global_config = None


def create_app(mode=None):
    application = Flask(__name__, static_url_path='/images', static_folder='../images')
    application.wsgi_app = ProxyFix(app.wsgi_app, x_prefix=1)
    check_config()
    global global_config
    global_config = read_config()

    if mode == 'dev' or global_config.debug:
        application.debug = True
    application.config.from_object(flask_config())

    #对数据库连接添加错误判断
    if global_config.engine == 'sqlite':
        db.init_app(application)

    elif global_config.engine == 'mongo':    
        mongo.init_app(application)
    else:
        db.init_app(application)

        
    from .api.img import img
    from .api.auth import auth
    from .api.sys import sys
    application.register_blueprint(img)
    application.register_blueprint(auth)
    application.register_blueprint(sys)


    return application
```

首先定义全局配置变量`global_config = None`，在应用初始化时进行读取`global_config = read_config()`

全局配置变量可以通过`from app import global_config`的方式在运行时进行引用

### 本地测试

在终端执行`python img_server.py`开启本地测试

如需使用异步框架，在应用`__init__.py`里修改`mode='dev'`使用**gevent**运行