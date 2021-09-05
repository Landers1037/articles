---
title: Nginx配置
name: nginx-conf
date: 2019-03-27
id: 0
tags: [linux]
categories: []
abstract: ""
---


# 在linux上运行nginx

## 在nginx下配置静态网站

进入`cd /etc/nginx`

编辑配置文件`vi nginx.conf`

<!--more-->


# 在linux上运行nginx

## 在nginx下配置静态网站

进入`cd /etc/nginx`

编辑配置文件`vi nginx.conf`
<!--more-->

```yaml
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /home/git;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

在本地`home/git`下面放入静态网页资源，conf文件中指定到该文件夹下

```bash
nginx -s reload
```

## 

## 在nginx下配置多个网站

注意：nginx的默认端口必须是80端口

使用nginx代理不同的端口的请求

```bash
cd /etc/nginx
mkdir conf
新建一个放置配置文件的文件夹
vi con1.conf
vi con2.conf
```

```yaml
#修改配置内容
#网站1
server {
        listen       80;
        server_name  _;
        root         /home/git;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
#网站2
server {
        listen       80;
        server_name  域名;
        root         /home/www;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

可以看见同时监听80端口，只有server name和root不同，当检测到输入的域名不同时自动切换到对应的web根目录

在nginx.conf中添加`include /etc/nginx/conf/*.conf`

方法二

在不同的端口部署

```yaml
server {
        listen       8081;
        server_name  _;
        root         /home/git;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

最终根据ip+端口访问网站

## nginx的log文件夹错误

见下一次总结