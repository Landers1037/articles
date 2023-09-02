---
title: mgek图床项目第四期
name: mgek-prj-imgbed-4
date: 2020-06-19
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第四期

全局配置设计

<!--more-->

配置文件作为十分重要的自定义服务的方式，应该使用易读格式化的格式

本系统使用**JSON**文件保存配置

## 配置设计

分为针对Flask服务的Flask配置和用户可自定义配置

配置包位于`app/config`下

### 配置引入

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: __init__.py.py
# @Date: 2020-05-12

# 配置文件处理函数，默认还是使用json作为配置文件

from .init_config import init_config
from .read_config import read_config
from .check_config import check_config
from .flask_config import flask_config
```

### 初始化配置

默认初始化函数，如果当前配置路径下不存在配置文件，则从初始化配置内读取

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: init_config.py
# @Date: 2020-05-12

import json
import os


def init_config():
    path = os.path.join(os.getcwd(), 'config.json')
    data = {
        "port": 5000,
        "debug": True,
        "server_name": None,  # 比较特殊的配置，默认应用绑定的域名，本地测试时应当忽略
        "multiroutine": False,
        "server": {
            "image_path": '',
            "image_size": 10*1024*1024,
            "image_page": 10,
            "image_url": 'http://localhost:5000/images/',
            "image_rule": 'base64',
            "image_zip": False,
            "image_zip_path": ''
        },
        "database": {
            "engine": 'sqlite',
            "sqlite": 'sqlite:///test.db',
            "mongo": ''
        }
    }
    with open(path, 'w', encoding='utf-8')as f:
        try:
            json.dump(data, f, ensure_ascii=False, indent=2)
        finally:
            f.close()

```

初始化函数会在系统启动时在本地生成`config.json`文件

### 读取配置

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: read_config.py
# @Date: 2020-05-12

import json
import os

# 避免每次调用的时候都有io操作，把数据存储在全局变量里
# 映射为类


class Config:
    def __init__(self, port, debug, server_name, multiroutine, server, database):
        self.port = port  # 通用配置
        self.debug = debug
        self.server_name = server_name
        self.multiroutine = multiroutine
        self.get_server(server)  # 服务器配置
        self.get_database(database)  # 数据库配置

    def get_server(self, server):
        self.image_path = set_default_config(
            server, 'image_path', '')  # 图片保存位置
        self.image_size = set_default_config(
            server,'image_size',10*1024*1024
        )
        self.image_page = set_default_config(server,'image_page',10)
        self.image_url = set_default_config(
            server, 'image_url', 'http://localhost:{}/images/'.format(self.port))  # 图片的服务器地址
        self.image_rule = set_default_config(
            server, 'image_rule', 'base64')  # 图片hash的命名方式
        self.image_zip = set_default_config(
            server, 'image_zip', False)  # 是否使用图片压缩，即生成缩略图
        self.image_zip_path = set_default_config(
            server, 'image_zip_path', '')  # 图片缩略图的存储位置
            
    def get_database(self, database):
        self.engine = set_default_config(
            database, 'engine', 'sqlite')  # 使用sqlite或mongodb
        self.sqlite = set_default_config(database, 'sqlite', 'sqlite:///test.db')  # sqlite的地址
        self.mongo = set_default_config(database, 'mongo', '')  # MongoDB地址

    def __repr__(self):
        return 'mgekimg_bed config Object'

    def __call__(self, *args, **kwargs):
        return {
            "port": self.port,
            "debug": self.debug,
            "multiroutine": self.multiroutine,
            "server": {
                "image_path": self.image_path,
                "image_size": self.image_size,
                "image_page": self.image_page,
                "image_url": self.image_url,
                "image_rule": self.image_rule,
                "image_zip": self.image_zip,
                "image_zip_path": self.image_zip_path
            },
            "database": {
                "engine": self.engine,
                "sqlite": self.sqlite,
                "momgo": self.mongo
            }
        }


def read_config(conf=None):
    if conf:
        # 按照键获取值
        with open(conf, 'r', encoding='utf-8')as f:
            config = json.load(f)
            pass
    else:
        try:
            path = os.path.join(os.getcwd(), 'config.json')
            with open(path, 'r', encoding='utf-8')as f:
                config = json.load(f)
                c = Config(
                    port=set_default_config(config,"port",5000),
                    debug=set_default_config(config, "debug",False),
                    server_name=set_default_config(config, 'server_name', None),
                    multiroutine=set_default_config(config, "multiroutine",False),
                    server=error_config_handler(config,"server"),
                    database=error_config_handler(config,"database")
                )
                return c
        except:
            return None

# 为防止配置文件的读取缺失 添加默认的配置项


def set_default_config(config, name, value):
    try:
        return config[name]
    except:
        return value

def error_config_handler(config,key):
    try:
        return config[key]
    except:
        return {}  
```

为了调用方便，这里对配置读取进行了类的映射

使用类`Config`即可通过其属性调用配置，如`Config().port`

### 配置合法化检测

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: check_config.py
# @Date: 2020-05-12

import os
from .init_config import init_config

def check_config():
    """
    用于生成默认的配置文件
    :return:
    """
    if not os.path.exists(os.path.join(os.getcwd(), 'config.json')):
        init_config()
    else:
        pass
```

在初始化时调用，用于判断本地是否存在配置文件，避免重写覆盖原先的配置文件

### Flask配置

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: flask_config.py
# @Date: 2020-05-14

# flask的配置对象格式
from .read_config import read_config
from .check_config import check_config

# 一个可以实例化解析为配置对象的类
# 这里有一个顺序问题，flask优先加载这里的配置文件，那么init配置步骤就在这里进行


class flask_config:
    check_config()
    raw_config = read_config()
    DEBUG = raw_config.debug
    SQLALCHEMY_DATABASE_URI = raw_config.sqlite
    SQLALCHEMY_TRACK_MODIFICATIONS = True
    SECRET_KEY = 'This is a Secret Key'
    SERVER_NAME = raw_config.server_name
    MAX_CONTENT_LENGTH = raw_config.image_size
    MONGO_DBNAME = 'mongo_mgekimghost'
    MONGO_URI = 'mongodb://localhost:27017/mgek_imghost'
    MONGO_HOST = 'localhost'
    MONGO_PORT = 27017
    MONGO_USERNAME = None
    MONGO_PASSWORD = None
    JWT = True
    IP_FILTER = True
    RULE = '10/60'
    MAIL_HOST = 'smtp.163.com'
    MAIL_PORT = 465
    MAIL_USER = 'user@163.com'
    MAIL_PASS = 'pass'

```

全局的Flask配置用于配置一些内置库的参数，你可以使用`current_app.config[]`方式调用

> 在此配置文件下的属性名称需要大写才能被Flask识别
>
> 你可以添加任意属性用于全局配置