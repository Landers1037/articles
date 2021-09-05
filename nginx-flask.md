---
title: nginx搭配flask
name: nginx-flask
date: 2019-06-21
id: 0
tags: [python]
categories: []
abstract: ""
---


在Nginx服务器上搭建Flask站点服务

### 需求

```python
pip install flask
pip install gunicorn
```


<!--more-->


在Nginx服务器上搭建Flask站点服务

### 需求

```python
pip install flask
pip install gunicorn
```

<!--more-->

### flask配置

```python
if __name__ == '__main__':
    from werkzeug.contrib.fixers import ProxyFix
    app.wsgi_app = ProxyFix(app.wsgi_app)
    app.run()
```

使用gunicorn代理必须加两句话

### nginx配置

```nginx
server {
    listen 8080;                #你想服务器的端口
    server_name 你的服务器地址;  
 
    location / {
        proxy_pass http://127.0.0.1:5000; #这个是Gunicorn与Ningx通信的端口。和Gunicorn的配置相同
	access_log /root/flaskweb/access.log;
	error_log  /root/flaskweb/error.log;
    }
  }

```

### 启动flask

```python
gunicorn -w 4 -b 127.0.0.1:5000 app:app
```

4表示4个进程 小网站不需要那么多

### 启动nginx

```bash
nginx -s reload
```

### 访问

访问8080端口