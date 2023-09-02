---
title: mgek图床项目第六期
name: mgek-prj-imgbed-6
date: 2020-06-19
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第六期

数据库模型设计，多数据库映射

<!--more-->

## 数据库工具引入

本项目使用到了sqlite和mongoDB数据库，分别使用**sqlachemy**和**pymongo**来连接这两个数据库

默认使用sqlite数据库，为了保证不使用的数据库不初始化，在`init`文件中判断当前数据库

`app/__init__.py`

```python
    db = SQLAlchemy()
	mongo = PyMongo()
...
    if global_config.engine == 'sqlite':
        db.init_app(application)

    elif global_config.engine == 'mongo':    
        mongo.init_app(application)
    else:
        db.init_app(application)
```

通过全局配置`global_config = read_config()`读取配置中定义的数据库

### 数据库模型

数据库模型仅支持使用sqlachemy的数据库

`app/models.py`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: models.py
# @Date: 2020-05-14

# 数据库的配置文件 slqite适用

from app import db

class Image(db.Model):
    __tablename__ = 'image'
    name = db.Column(db.String(100),primary_key=True) #图片唯一id
    mail = db.Column(db.String(50),nullable=False) #图片所属的账户
    path = db.Column(db.String(100),nullable=False) #图片地址
    url = db.Column(db.String(100),nullable=False) #图片在服务器上的地址
    time = db.Column(db.Integer) #图片更新时间

    def __init__(self,name,mail,path,url,time):
        self.name = name
        self.mail = mail
        self.path = path
        self.url = url
        self.time = time
    
    def info(self):
        return {
            "name": self.name,
            "path": self.path,
            "url": self.url,
            "time": self.time
            #为保证图片信息安全不返回所属账户
        }

class Token(db.Model):
    __tablename__ = 'token'
    mail = db.Column(db.String(50),primary_key=True)
    token = db.Column(db.String(100))
    check = db.Column(db.String(100),nullable=True)

    def __init__(self,mail,token,check):
        self.mail = mail
        self.token = token
        self.check = check

    def info():
        return {
            "token": self.token
        }
```

在通过sqlachemy映射操作获取到模型定义的类时，可以使用定义的函数`info()`来返回信息

### 多数据库

这只是一个简单的多数据库统一函数，为简单实现添加数据，删除数据的统一

`app/database.py`

```python
# -*- coding: utf-8 -*-
# @Author: Landers
# @Github: Landers1037
# @File: database.py
# @Date: 2020-05-15

#数据库的封装调用
#当前就支持图片信息的添加和获取的封装
#其他操作使用逻辑判断

from app import db
from app.models import *
from app import mongo

class database:
    sql_map = {
        "image": Image,
        "token": Token
    }
    def init(self):
        db.create_all()

    def set(self,engine,table,data):
        if engine == 'sqlite':
            #判断操作的表
            self.init()
            if table == 'image':
                try:
                    d = self.sql_map[table](name=data["name"], mail=data["mail"], path=data["path"], url=data["url"], time=data["time"])
                    db.session.add(d)
                    db.session.commit()
                    return True
                except:
                    db.session.rollback()
                    return False

            elif table == 'token':
                try:
                    t = self.sql_map[table](mail=data["mail"], token=data["token"],check='')
                    db.session.add(t)
                    db.session.commit()
                    return True
                except Exception as e:
                    print(e.args)
                    db.session.rollback()
                    return False

        else:
            #mongo
            if table == 'image':
                try:
                    mongo.db.images.insert_one({"name": data["name"],"mail": data["mail"], "path": data["path"],"url": data["url"],"time": data["time"]})
                    return True
                except Exception as e:
                    print(e.args)
                    return False

            elif table == 'token':
                try:
                    mongo.db.token.insert_one({"mail": data["mail"],"token": data["token"],"check": ''})
                    return True
                except:
                    return False    

    def get(self,engine,table,data):
        if engine == 'sqlite':
            if table == 'image':
                try:
                    d = self.sql_map[table].query.filter_by(name=data).first()
                    return d.info()
                except Exception as e:
                    print(e.args)
                    return {}

            elif table == 'token':
                #没有获取token的函数，这个查询的作用是判断当前传入token是否存在，存在则返回对应的mail
                try:
                    t = self.sql_map[table].query.filter_by(token=data).first()
                    if t:
                        return t.mail
                    else:
                        return None    
                except:
                    return None 
        else:
            #mongo
            if table == 'image':
                try:
                    i =  mongo.db.images.find_one({"name": data})
                    return {"name": i["name"], "path": i["path"], "url": i["url"], "time": i["time"]}

                except Exception as e:
                    print(e.args)
                    return {}

            elif table == 'token':
                try:
                    t =  mongo.db.token.find_one({"token": data})
                    if t:
                        return t["mail"]
                    else:
                        return None    

                except:
                    return None     

    def get_token(self,engine,data):
        #单独的通过mail获取token的函数
        if engine == 'sqlite':
            try:
                t = Token.query.filter_by(mail=data).first()
                return t.token
            except:
                return None    
        else:
            try:
                t = mongo.db.token.find_one({"mail": data})
                return t["token"]
            except:
                return None

    def get_image_list(self,engine,data):
        if engine == 'sqlite':
            try:
                img_list = Image.query.filter_by(mail=data).all()
                return img_list
            except:
                return False
        else:
            try:
                img_list = mongo.db.images.find({"mail": data})
                return img_list
            except:
                return False    


    def update_token(self,engine,data):
        if engine == 'sqlite':
            try:
                t = Token.query.filter_by(mail=data["address"]).first()
                t.check = data["check"]
            except:
                pass
        else:
            try:
                t = mongo.db.token.update_one({"address": data["address"]}, {'$set': {"check": data["check"]}})
            except:
                pass    

    def delete(self,engine,table,data):
        if engine == 'sqlite':
            try:
                d = self.sql_map[table].query.filter_by(name=data).first()
                db.session.delete(d)
                db.session.commit()
            except:
                db.session.rollback()                 

        else:
            try:
                data = mongo.db.images.delete_one({"name": data})
            except:
                pass  

    def delete_many(self, engine, table, data):
        if engine == 'sqlite':
            try:
                for d in data:
                    d = self.sql_map[table].query.filter_by(name=d).first()
                    db.session.delete(d)
                db.session.commit()
            except:
                db.session.rollback()

        else:
            try:
                for d in data:
                    data = mongo.db.images.delete_one({"name": d})
            except:
                pass

    def remove_token(self,engine,data):
        if engine == 'sqlite':
            try:
                mail = data["mail"]
                if data["token"]:
                    #通过token删除
                    t = Token.query.filter_by(mail=mail).first()
                    if t.token == data["token"]:
                        try:
                            db.session.delete(t)
                            db.session.commit()
                            return True
                        except Exception as e:
                            print(e.args)
                            db.session.rollback()
                            return False    
                    else:
                        return False
                elif data["check"]:
                    t = Token.qurey.filter_by(mail=mail).first()
                    if t.check == data["check"]:
                        try:
                            db.session.delete(t)
                            db.session.commit()
                            return True
                        except:
                            db.session.rollback()
                            return False
                    else:
                        return False
            except:
                return False                                  
        else:
            mail = data["mail"]
            if data["token"]:
                t = mongo.db.token.find_one({"mail": mail})
                if t["token"] == data["token"]:
                    try:
                        mongo.db.token.delete_one({"mail": mail})
                        return True
                    except:
                        return False  
                else:
                    return False
            elif data["check"]:
                t = mongo.db.token.find_one({"mail": mail})
                if t["check"] == data["check"]:
                    try:
                        mongo.db.token.delete_one({"mail": mail})
                        return True
                    except:
                        return False  
                else:
                    return False  
```

定义类**database**，内部定义数据库的常规操作函数，根据使用的数据库的不同编写不同的操作语句，保证输入参数和返回值的统一性

例如：`database().set(table.data)`函数实现数据的添加操作，`table`变量传入需要更改的表，`data`为字典包含传入要插入的数据