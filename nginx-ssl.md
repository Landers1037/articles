---
title: nginx上开启ssl
name: nginx-ssl
date: 2019-05-30
id: 0
tags: [linux,nginx]
categories: []
abstract: ""
---


在nginx上开启ssl，完成网站的http跳转https


<!--more-->


在nginx上开启ssl，完成网站的http跳转https

<!--more-->

### 申请ssl证书

在阿里云或者腾讯云上申请免费的ssl证书，不想备案就在lets'encrypt上申请

最后得到的是三个文件，需要的是certificate.crt和private.key文件

放到`/etc/nginx/cert`目录下，没有此目录需要自己创建

### 开启443端口

参考前面的firewall使用

### 配置ssl

在`/etc/nginx/conf.d`下新建一个`ssl.conf`文件
填入一下信息

```c
server {
    listen 443;
    server_name xxx.com; // 你的域名
    ssl on;
    root /var/www; // 网站的站点文件夹
    index index.html index.htm;// 上面配置的文件夹里面的index.html
    ssl_certificate  /etc/nginx/cert/cert.crt;// 改成你的证书的名字
    ssl_certificate_key /etc/nginx/cert/private.key;// 你的证书的名字
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        index index.html index.htm;
    }
}
```

### 开启http强制跳转https

在nginx.conf中修改

```c
server {
    listen 80;
    server_name bjubi.com;// 你的域名
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

### 验证

`nginx -t`

检查无误后，`nginx -reload`

