---
title: 随笔-Nginx的配置总结
name: sb2
date: 2019-08-18
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


针对平时常用**Nginx**配置做了下总结
填下Nginx这个大坑

<!--more-->

平时的**Nginx**用来反向代理，放置静态资源，一些特定的优化还是要做的

### 文件代理

简单地使用**Nginx**代理文件目录作为文件下载服务

```nginx
server {
    listen       80;
    server_name  xxx;
    charset utf-8;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root /home/;
        autoindex on;    #开启索引功能
        autoindex_exact_size off;  #关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
        autoindex_localtime on;   #显示本机时间而非 GMT 时间
   }
```



### 端口转发

对于需要多个域名绑定不同端口，统一使用**Nginx**进行转发
如果是**http**访问，设置默认监听端口80，然后对域名进行不同端口转发

```nginx
server {
        listen 80;
        server_name 域名1;
        proxy_set_header Host $host:$server_port;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        location / {
        proxy_cache my_cache;
        proxy_pass http://127.0.0.1:1318;
        expires 15d;
                }

        }
```

此时的两个域名，分别监听在**1317**和**1318**，可以直接通过域名访问



### 隐藏转发端口

```nginx
#对于由nginx代理的web服务
proxy_set_header Host $host:$server_port;
proxy_set_header Host $http_host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#已经可以实现仅通过域名访问，禁止ip端口访问
```

对于直接由**Nginx**代理的静态资源路径，通过ip默认可以直接访问根目录下所有资源，为其设置ip默认返回值

```nginx
server {
        listen 80;
        server_name 域名1;
	    return 444 #或者内部500错误码
        location / {
                }
        }
```



### 缓存配置

**Nginx**自带了缓存配置，可以对静态资源进行缓存，保证在大量服务访问服务器时直接从缓存中返回数据而不是请求服务器，这样可以减轻服务器的压力。

```nginx
#开启缓存
http{
    proxy_cache_path /home/nginx/cache levels=1:2 keys_zone=my_cache:5m max_size=1g inactive=60m use_temp_path=off;
}

```

`proxy_cache_path`缓存存放目录
`levels=1:2`目录等级，两层子目录，当静态资源过多时单层路径会降低访问速度
`keys_zone`定义缓存的名称和共享内存的大小，后面的server配置里用到
`max_size=1g`目录的最大容量，超出后会自动清除之前的缓存
`inactive`指定资源过期时间，这段时间内未被访问就会自动删除，**注意：它的优先级比expire高**

```nginx
server {
        listen 80;
        server_name 域名1;
        ...
        location / {
        proxy_cache my_cache; #生效缓存
        proxy_pass http://127.0.0.1:1318;
        expires 15d; #浏览器端的缓存过期时间，后面有详细说明
                }
        }
```



### cache设置

**Nginx**为静态资源添加`cache-control`请求头可以设置资源在浏览器的过期时间
**注意：之间的缓存配置里inactive参数优先级最高，超过这个时间后都会像服务器发送资源刷新请求**

#### 请求头参数说明

`no-cache`数据不会缓存，每次请求都会访问服务器，如果由max-age参数则缓存期间不会请求服务器
`no-store`不能缓存，不能本地存储
`private`只能在浏览器里缓存，只有第一次请求时访问服务器
`public`可以被任何缓冲区缓存
`max-age`以秒为单位的最大过期时间
`expires`绝对过期时间，优先级低于`cache-control`

#### 示例

```nginx
server {
        listen 80;
        server_name 域名1;
        ...
        location / {
        proxy_pass http://127.0.0.1:1318;
        expires 15d; #最大过期时间15天
                }
        }
```

```nginx
server {
        listen 80;
        server_name 域名1;
        ...
        location / {
        proxy_cache my_cache; #生效缓存
        proxy_pass http://127.0.0.1:1318;
        add_header Cache-Control no-cache; #不缓存
        add_header Cache-Control max-age=43200; #过期时间43200s     
                }
        }
```



### 资源gzip压缩传输

可以使用gzip参数来压缩静态资源传输，减小浏览器的响应时间

```nginx
gzip_static on; #默认全局的gzip，优先级高于gzip
gzip on; #开启gzip
gzip_min_length 1k; #需要压缩的文件最小大小
gzip_buffers 4 16k; #以16k为固定长度，压缩分段是其4倍的大小
gzip_http_version 1.1; #默认
gzip_comp_level 4; #压缩等级1-9，越高越好，服务器开销越大
gzip_disable "MSIE [1-6]\."; #ie6浏览器请求禁用gzip
gzip_types text/plain text/css application/xml text/javascript application/javascript application/json application/font-woff; #自定义gzip要压缩的文件类型
gzip_vary on; #自动识别代理客户端是否支持gzip
...
server{}
```



### 重定向

#### https重定向

```nginx
server {
	listen       80 default_server;
	listen       [::]:80 default_server;
	server_name  _;
	rewrite ^ https://$http_host$request_uri? permanent;
```

#### url重定向

```nginx
server {
	listen       80 default_server;
	listen       [::]:80 default_server;
	server_name  _;
	rewrite ^ https://abc.com  permanent;
```

