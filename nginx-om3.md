---
title: nginx运维笔记3
name: nginx-om3
date: 2021-05-30 17:29:28
id: 0
tags: [nginx]
categories: [nginx]
abstract: ""
---

nginx运维笔记3 - nginx的简单认证和请求体限制

<!--more-->

## nginx的简单认证

nginx默认自带了`ngx_http_auth_basic_module`模块可以用于简单的http身份认证

在使用时只需要添加命令`auth_basic`即可

**语法:**   auth_basic string | off;
**语法:** auth_basic_user_file file;

**默认值:**   auth_basic off;
**配置段:**    http, server, location, limit_except

用户的密钥文件形式类似`username:password`

其中password应该是使用加密算法混淆后的密文

使用**htpasswd**或者**openssl**即可生成专属的密钥文件

### 生成密钥文件

openssl方式

```bash
printf "user1:$(openssl passwd -crypt 123456)\n" >>conf/htpasswd
# cat conf/htpasswd 
user1:xFdIQekfBatrw
```

htpasswd方式

```bash
yum -y install httpd-tools
htpasswd -c passwd user1
# 输入密码
```

### 配置nginx认证

在需要认证的server或者location下添加

```nginx
server {
    listen       80;
    server_name  _;
    
    auth_basic "一段用于认证的提示话语";
    auth_basic_user_file /etc/nginx/passwd;
    
    location / {
        root   /data/www;
        index  index.html;
    }
}
```

再次访问页面就会弹出认证框

## nginx限制请求体

在http上传文件时可能遇到这样的提示

```bash
error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413 Request Entity Too Large
fatal: The remote end hung up unexpectedly
# nginx
client intended to send too large body
```

因为nginx默认限制了请求体大小，大文件的上传超过了nginx的限制导致nginx拒绝该请求

在对应的location下添加对请求体的限制

```nginx
location / {
    client_max_body_size 1024m;
}
```

限制请求体最大为1024mb
