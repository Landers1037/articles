---
title: flask的静态文件处理
name: flask-static-path
date: 2019-07-29
id: 0
tags: [flask]
categories: []
abstract: ""
---


Flask处理静态文件

修改配置参数`static_path_for`和`static_folder`


<!--more-->


Flask处理静态文件

修改配置参数`static_path_for`和`static_folder`

<!--more-->

### flask里的static目录

在Flask中所有的静态文件统一放置在`static`目录下，所以所有的静态文件调用路径为`/static/css/`

```css
<link href="/static/css/imain.css" rel="stylesheet">
```

这样在浏览器上显示的静态文件路径就变为了`http://localhost:5000/static/css/***`

如果希望直接使用根目录的访问的方式，需要修改配置参数

```css
<link href="/css/imain.css" rel="stylesheet">
```

### 静态文件配置

修改flask的配置参数，添加`static_path_for=""`或者`static_folder=""`

```python
app = Flask(__name__,static_path_for="")
```

此时浏览器访问静态文件路径变为`http://localhost:5000/css/***`

注意：静态文件依旧放置在`static`目录下，改变的只是flask识别的静态文件目录

### 静态文件的发送

一般flask都会使用`render_template(xxx.html)`的方法渲染返回网页，html文件必须放置于`templates`目录下

如果发送的html文件是纯静态的不需要返回数据，则可以使用发送**静态文件**的方法

```python
return app.send_static_file("game/index.html")
```

默认根路径为`static`，返回的是`/static/game/index.html`

或者

```python
root = "/static/game/"
return app.send_from_directory(root,"index.html")
```

自行设定发送文件的目录位置