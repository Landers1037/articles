---
title: nginx代理配置https
name: nginx-proxy-https
date: 2020-03-08
id: 0
tags: [nginx]
categories: []
abstract: ""
---


在**Nginx**上配置https的一个示例，涉及到多个proxy代理的配置

<!--more-->

因为项目设计到前后端请求，当把项目转换为`https`后，`axios`的请求地址仍是`http`所以会被浏览器拦截

### 环境

前端： Vue

代理服务器： nginx

后端： 开放端口5000

在`axios`的代理配置中修改请求的后端地址为https://xxx保证发出的均为https请求

### api部分

api部分的服务器运行于5000端口，在**axios**的配置里设置所有`/api`开头的请求都会发送至该端口

nginx.conf

```nginx
server{
    location /api {
        proxy_pass http://127.0.0.1:5000
    }
}
```

### https配置

首先需要开启nginx的https配置，在443端口监听需要的域名

```nginx
server
{
  # 80端口是http正常访问的接口
  listen 80;
  server_name xxx.xxx;
  location / {
        return    301 https://$server_name$request_uri;
        }
}

server {
        listen 443;
        server_name xxx.xxx;
        ssl on;
        ssl_certificate /etc/nginx/ssl/full_chain.pem;
        # 指定私钥文件路径
        ssl_certificate_key /etc/nginx/ssl/private.key;
        ssl_prefer_server_ciphers on;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

        location / {
        proxy_cache my_cache;
        proxy_ssl_certificate /etc/nginx/ssl/full_chain.pem;
        # 指定私钥文件路径
        proxy_ssl_certificate_key /etc/nginx/ssl/private.key;
        proxy_ssl_server_name on;
        proxy_ssl_session_reuse  on;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-proto https;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8080;
        expires 1d;
                }
}
```

当监听80端口匹配到域名时，就会重定向到443端口上。

`Vue`服务运行在本地`8080`端口，如果想要proxy代理的所有请求都是https则需要在对应的location下添加 `proxy_ssl`参数

`ssl_certificate`是你的ssl证书公钥

`ssl_certificate_key`是你的ssl证书私钥

在proxy代理中配置`proxy_ssl_certificate`，`proxy_ssl_certificate_key`这是你的代理服务器使用的证书密钥

配置完成后请求，会提示api部分的请求为http协议被浏览器禁止，另外配置api的`proxy_pass`的证书

```nginx
        location /api {
        proxy_ssl_certificate /etc/nginx/ssl/full_chain.pem;
        # 指定私钥文件路径
        proxy_ssl_certificate_key /etc/nginx/ssl/private.key;
        proxy_ssl_server_name on;
        proxy_ssl_session_reuse  on;
        proxy_pass http://127.0.0.1:5000;
        expires 10s;
        }
```

同样在80端口这里做重定向配置

```nginx
  location /api {
        rewrite ^/api/(.*)$   https://xxx.xxx/api/$1 permanent;
        }
```

### 测试

为了验证http是否会自动跳转为https，在浏览器中输入http://xxx.xx

在响应头中可以看到`301`响应码，地址栏自动转变为`https`

至此vue项目，api服务器的传输就都变为`https`协议