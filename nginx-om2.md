---
title: nginx运维笔记2
name: nginx-om2
date: 2020-02-27
id: 0
tags: [nginx]
categories: []
abstract: ""
---


Nginx服务器的运维笔记-负载均衡

<!--more-->

## 负载均衡

### 负载均衡的实现策略

轮询

```nginx
upstream load_balance {
  # 默认所有服务器权重为 1
  server 127.0.0.1:8080
  server 127.0.0.1:8081
  server 127.0.0.1:8082
}
```



加权轮询

```nginx
upstream load_balance {
  server 127.0.0.1:8080   weight=3
  server 127.0.0.1:8081              # default weight=1
  server 127.0.0.1:8082              # default weight=1
}
```

最少连接

```nginx
upstream load_balance {
  least_conn;
  server 127.0.0.1:8080
  server 127.0.0.1:8081
  server 127.0.0.1:8082
}
```

加权最少连接

```nginx
upstream load_balance {
  least_conn;  
  server 127.0.0.1:8080   weight=3
  server 127.0.0.1:8081              # default weight=1
  server 127.0.0.1:8082              # default weight=1
}
```

ip hash

```nginx
upstream load_balance {
  ip_hash;
  server 127.0.0.1:8080  
  server 127.0.0.1:8081              
  server 127.0.0.1:8082              
}
```

普通hash模式

```nginx
upstream load_balance {
  hash $request_uri;
  server 127.0.0.1:8080  
  server 127.0.0.1:8081              
  server 127.0.0.1:8082              
}
```

#### 示例

使用**加权轮询**的示例

```nginx
upstream load_balance {
  server 127.0.0.1:8080   weight=3
  server 127.0.0.1:8081   weight=3
  server 127.0.0.1:8082   weight=4
}

server{
    listen       80;   
    server_name  www.test.com;
    location /{
        proxy_pass  http://load_balance_server;
    }
}
```

可选的代理配置

```nginx
#proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr; #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
proxy_set_header X-Forwarded-For $remote_addr;
proxy_connect_timeout 90;          #nginx跟后端服务器连接超时时间(代理连接超时)
proxy_send_timeout 90;             #后端服务器数据回传时间(代理发送超时)
proxy_read_timeout 90;             #连接成功后，后端服务器响应时间(代理接收超时)
proxy_buffer_size 4k;              #设置代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
proxy_busy_buffers_size 64k;       #高负荷下缓冲大小（proxy_buffers*2）
proxy_temp_file_write_size 64k;    #设定缓存文件夹大小，大于这个值，将从upstream服务器传

client_max_body_size 10m;          #允许客户端请求的最大单文件字节数
client_body_buffer_size 128k;      #缓冲区代理缓冲用户端请求的最大字节数
```

配置完成后，本机的`8080`，`8081`，`8082`三个端口必须运行三个相同的web服务以达到负载均衡的目的

### 分应用配置

假如有一个大型的web项目，对其进行模块化的拆分，分为`/user`，`/admin`, `/annoymous`

即访问

www.test.com/user  用户模块

www.test.com/admin  管理员模块

www.test.com/annoymous  匿名用户模块

配置负载均衡

```nginx
upstream user_server{
		server www.test.com:8080;
	}

upstream admin_server{
		server www.test.com:8081;
	}

upstream annoy_server{
		server www.test.com:8082;
	}
server {
    location /user {
			proxy_pass http://user_server;
		}

	location /admin {
			proxy_pass http://admin_server;
		}

	location /annoymous {
			proxy_pass http://annoy_server;
		}
}
```

然后我们分别在3个对应的端口开启服务，当用户访问不同的路由时就会转到对应的代理后端地址

## 跨域

CORS

当服务之间的请求协议`http/https`，`domin`域名，`port`端口不同时就会涉及跨域问题，不同域的数据不能共享

使用添加响应头的方式

```nginx
server {
    location /{
        add_header 'Access-Control-Allow-Origin' "*";
		add_header 'Access-Control-Allow-Credentials' 'true';
		add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
		add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    }
}
```

`Access-Control-Allow-Origin`里面设置允许跨域访问的域名/IP，如果使用`*`时出于安全考虑，不能发送带认证头和`cookie`的请求

`Access-Control-Allow-Headers` 除了使用默认规范的头部还可以添加自定义的请求头键，在客户端发送时只需要附带上键和值即可

