---
title: mgek图床项目第二期
name: mgek-prj-imgbed-2
date: 2020-06-18
id: 0
tags: [python]
categories: []
abstract: ""
---


**Mgek项目**-ImgBed第二期

路由设计

<!--more-->

# 路由模块

1. 图片相关模块
2. 登录认证模块
3. 系统配置模块

## 图片相关模块

设计蓝图名为**img**的蓝图，位于项目路径`app/api/img/init.py`

```python
# @Author: Landers1037
# @Github: github.com/landers1037
# @File: __init__.py.py
# @Date: 2020-05-12


from flask import Blueprint
from flask_cors import CORS


img = Blueprint('img_api', __name__)
CORS(img)


from . import image_upload
from . import image_info
from . import image_list
from . import image_delete
from . import image_format
```

在`img`目录下新建路由文件，添加和操作图片相关的接口

### 图片上传

图片上传功能设计，支持批量上传，Flask原生支持批量文件的表单获取。

图片格式需要规范，`jpg/png/webp/gif`为减少后端的逻辑判断，此操作应在前端页面实现

图片名称需要安全规范且唯一，可以使用`werkzeug`内置的`secure_name`模块生成安全的名称，在这里使用**md5**和**base64**两种方式生成唯一名称

图片保存，因为要保证文件名称唯一，所以新文件的名称直接使用规范化后的名称，为保证数据库内的文件字段和本地文件同步，需要在数据库写入完成后做文件存储操作

`img/image_upload.py`

```python
@img.route('/api/image_upload',methods=['POST'])
def image_upload():
    #保持一个上下文的命名变量
    #默认的文件列表是file，这应该与前端保持同步
    #没有使用安全命名的方式，因为所有文件名会经过hash计算后重命名
    #默认需要登录认证
    token = request.args.get("token")
    files = request.files.getlist('file')
    path = global_config.image_path if global_config.image_path != '' else os.path.join(os.getcwd(),"images")

    if token and database().get(global_config.engine,'token',token):
        #账户存在
        if not os.path.exists(path):
            os.mkdir(path)
            #保证目录的创建
            #判断是否有上传文件
        if 'file' not in request.files:
            return format_response('error', '空的上传文件')
        else:
            mail = database().get(global_config.engine, 'token', token)
            for f in files:
                if f.filename:
                    name = rename.rename(f.filename)
                    f.save(os.path.join(path, name))

                    #数据库操作
                    res = database().set(global_config.engine, 'image',
                                        {"name": name, "mail": mail, "path": os.path.join(path, name),
                                        "url": "{}{}".format(global_config.image_url, name),
                                        "time": generate_time()[0]
                                    })
                    if res:
                        pass
                    else:
                        return format_response('error', '文件上传失败')

            return format_response('ok', '文件上传成功')

    else:
        return format_response('error', '无文件上传权限')
```

Flask默认从表单获取文件列表时键为`file`,所以`if 'file' not in request.files`先判断上传字段是否符合规范

因为涉及多数据库的切换操作，所以对数据库操作进行了简单的封装

`database()`类是一个封装完成的数据库映射类，是操作sqlite和mongoDB达到统一

### 图片列表

从本地存储的数据库内部读取图片列表

`img/image_list.py`

```python
@img.route('/api/image_list')
def image_list():
    #支持按需获取图片列表 用于前端的懒加载
    token = request.args.get('token')
    if request.args.get('page'):
        #分页情况
        #默认从第一页开始
        if global_config.engine == 'sqlite':
            try:
                # pages = count_page(all,global_config.image_page)
                p = Image.query.paginate(1,10)
                #暂未实现
                return format_response('ok',10)
            except:
                return format_response('error','获取图片列表错误')    
        else:
            pass    

    else:
        #默认返回全部图片列表
        #包含图片总数，计算得到的图片分页数
        g.data = []
        mail = database().get(global_config.engine, 'token', token)
        if mail:
            #仅获取当前用户下的图片列表
            if global_config.engine == 'sqlite':
                img_list = database().get_image_list(global_config.engine,mail)
                for i in img_list:
                    g.data.append(i.info())

            elif global_config.engine == 'mongo':
                img_list = database().get_image_list(global_config.engine, mail)
                for i in img_list:
                    g.data.append({"name": i["name"], "path": i["path"], "url": i["url"]})

            else:
                pass         
        else:
            pass
        return format_response('ok',g.data)  
```

懒加载分页暂未实现，所以每次获取的都是全部的图片列表，当列表过大时会导致响应巨大

默认返回的图片列表通过 `database().get_image_list`方法实现，传入`mail`参数

该参数为指定的用户(因为本系统是通过邮箱地址区分账户)，获取指定账户下的图片列表，返回图片信息包括图片的名称。在服务器上的绝对路径，图片外部链接地址

### 图片信息

和图片列表的实现基本类似，只是对单个图片的信息获取，通过图片的唯一ID即名称获取

`img/image_info.py`

```python
@img.route('/api/image_info')
def image_info():
    # 根据图片的id获取图片信息
    name = request.args.get("name")
    img = database().get(global_config.engine,'image',name)
    if img:
        return format_response('ok',img)
    else:
        return format_response('error','图片信息获取失败')
```

### 图片删除

根据图片的唯一ID删除图片

因为涉及删除操作，所以应该先进行数据库操作再进行本地文件删除

`img/image_delete.py`

```python
@img.route('/api/image_delete',methods=['POST'])
def image_delete():
    #根据图片的唯一ID删除对应的文件
    data = request.json
    if "namaelist" in data.keys():
        database().delete_many(global_config.engine,'image',data["namelist"])
        return format_response('ok', '文件删除成功')

    elif "name" in data.keys():
        database().delete(global_config.engine,'image',data["name"])
        return format_response('ok','文件删除成功')
```

### 图片格式化

一个功能选项，可以自定义输出的图片信息格式

默认会输出图片直链，图片的`html`引入格式，图片的`markdown`引入格式

```python
@img.route('/api/image_format')
def image_format():
    name = request.json["name"]
    img = database().get(global_config.engine,'image',name)
    res = {
            "raw": img["name"],
            "link": "{}{}".format(global_config.image_url,img["name"]),
            "html": "<img src={}{} alt=image>".format(global_config.image_url,img["name"]),
            "markdown": "![image]({}{})".format(global_config.image_url,img["name"])
        }
    return format_response('ok',res)
```

### 使用到的函数

格式化输出函数，数据库模型，数据库映射函数将在后面写到