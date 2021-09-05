---
title: docker打包flask程序
name: docker-makeflask
date: 2019-06-28
id: 0
tags: [docker]
categories: []
abstract: ""
---


使用docker打包flask程序实现快速部署


<!--more-->


使用docker打包flask程序实现快速部署

<!--more-->

### Flask文件配置

运行地址和端口需要修改

```python
if __name__ == '__main__':
    app.jinja_env.auto_reload = True
    app.run(host='0.0.0.0',port=80)
```



### 配置文件

建立一个文件夹`dockertest`

**把flask项目全部放置到此目录下**

#### 添加Dockerfile文件和依赖文件

```sh
touch Dockerfile
touch requirements.txt
```

#### Dockerfile

```c
FROM python:3.7
MAINTAINER name xxx@gmail.com
WORKDIR /app
ADD . /app
RUN pip install -r requirements.txt
EXPOSE 80
RUN chmod +x /app/start.sh
ENTRYPOINT /app/start.sh
```

本镜像依赖了python3.7的官方docker镜像

在根目录下新建了`app`目录，开启了本地80端口

将开启flask的脚本写在`start.sh`里方便定制

如果直接使用命令调用使用如下形式

```c
CMD ["python", "app.py"]
```



#### python requirements

```
gunicorn
flask
```

根据自己项目依赖的包自行添加

#### start.sh

```bash
#! /bin/bash
python /app/app.py
```

因为只是测试所以直接使用python执行flask程序

真正运行时建议使用gunicorn

```bash
#! /bin/bash
gunicorn -w 4 -b 0.0.0.0:80 app:app
```



### 构建docker镜像

```sh
cd dockertest
docker build -t flasktest .
```

创建了一个名为`flasktest`的镜像

### 测试

```sh
docker -d -p 5000:80 flasktest
```

5000为本地端口 80为docker内部的flask程序运行的端口

访问[127.0.0.1:5000](http://127.0.0.1:5000)