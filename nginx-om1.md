---
title: nginx运维笔记1
name: nginx-om1
date: 2020-02-27
id: 0
tags: [nginx]
categories: []
abstract: ""
---


Nginx服务器的运维笔记-配置


<!--more-->


Nginx服务器的运维笔记-配置

<!--more-->

**Nginx**是最常用的反向代理服务器

### 反向代理

什么是反向代理？

通俗的理解，你(client)去餐馆点菜，这时候服务员(nginx)来了问你点什么菜，你告诉它菜名(http请求)。他把菜名告诉后厨的厨师(server)，会做这个菜的厨师做好后把菜(response)给服务员，服务员再交给你(client)。

在整个过程中你只和服务员有过交流，其他的处理都是服务员和厨师进行的。即客户端发送请求给nginx后nginx把请求发送给后端的服务器处理然后返回，你全程只知道自己和**nginx**交互了，但是你不知道后端的server到底是哪一台。

## 配置

### 配置文件解释

```nginx
#user somebody;

#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志
error_log  D:/Tools/nginx-1.10.1/logs/error.log;
error_log  D:/Tools/nginx-1.10.1/logs/notice.log  notice;
error_log  D:/Tools/nginx-1.10.1/logs/info.log  info;

#PID文件，记录当前启动的nginx的进程ID
pid        D:/Tools/nginx-1.10.1/logs/nginx.pid;

#工作模式及连接数上限
events {
    worker_connections 1024;    #单个后台worker process进程的最大并发链接数
}
...
```

`user` 即运行nginx的用户（使用root运行时应该指定为root）

`worker_processes` 进程数 和cpu核心数相同即可

`worker_connections` 最大的并发数，默认即可

### http配置部分

```nginx
http {
    #设定mime类型(邮件支持类型),类型由mime.types文件定义
    include       /etc/nginx/conf/mime.types;
    default_type  application/octet-stream;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  120;
    tcp_nodelay        on;

	#gzip压缩开关
	#gzip  on;

    #HTTP服务器
    server {
        #监听80端口，80端口是知名端口号，用于HTTP协议
        listen       80;

        #定义使用www.xxx.com访问
        server_name  www.xxx.com;

		#首页
		index index.html

		#指向webapp的目录
		root /home/test;

		#编码格式
		charset utf-8;

		#代理配置参数
        proxy_connect_timeout 180;
        proxy_send_timeout 180;
        proxy_read_timeout 180;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarder-For $remote_addr;

        #反向代理的路径
        location / {
            proxy_pass http://127.0.0.1:8080;
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status           on;
            access_log            on;
            auth_basic            "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }

        #禁止访问 .log 文件
        location ~ /\.log {
            deny all;
        }

		#错误处理页面（可选择性配置）
		#error_page   404              /404.html;
		#error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
    }
}
```

每一个服务用一个`server`设置

默认nginx运行于80端口，当我们需要配置多个代理时，使用proxy添加代理地址

`proxy_pass http://127.0.0.1:8080` 

此时实际的web应用运行在8080端口上，当我们访问80端口时nginx会通过域名来判断把请求发送到哪个后端服务

#### 静态文件

```nginx
        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root /home/test/static;
            #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }
```

### https配置

nginx默认使用http，要开启https需要添加https配置和ssl证书

```nginx
server {
      #监听443端口。443为知名端口号，主要用于HTTPS协议
      listen       443 ssl;

      #定义使用www.xx.com访问
      server_name  www.xxx.com;

      #ssl证书文件位置(常见证书文件格式为：crt/pem)
      ssl_certificate      /home/cert.pem;
      #ssl证书key位置
      ssl_certificate_key  /home/cert.key;

      #ssl配置参数（选择性配置）
      ssl_session_cache    shared:SSL:1m;
      ssl_session_timeout  5m;
      #数字签名，此处使用MD5
      ssl_ciphers  HIGH:!aNULL:!MD5;
      ssl_prefer_server_ciphers  on;

      location / {
          root   /root;
          index  index.html index.htm;
      }
  }
```

## 调试

加入我们配置的域名是 www.test.com

此时启动**nginx**

```bash
systemctl start nginx
```

nginx的命令

```bash
nginx -s stop       直接关闭nginx，不保存相关信息
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen     重新打开日志文件。
nginx -c filename   为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t            不运行，仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
nginx -v            显示 nginx 的版本。
nginx -V            显示 nginx 的版本，编译器版本和配置参数。
```

启动后，修改本机的host文件

```
127.0.0.1 www.test.com
```

访问www.test.com即可看见你添加的web服务