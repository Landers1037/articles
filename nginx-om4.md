---
title: nginx运维笔记4
name: nginx-om4
date: 2021-07-23 21:44:03
id: 0
tags: [nginx]
categories: [nginx]
abstract: ""
---

nginx运维笔记4 - nginx的主题配置

<!--more-->

# nginx的主题配置

## 开启nginx文件服务器

在需要开启的路由添加参数

```bash
location / {    
    root /home;
    autoindex on; #开启目录浏览
    autoindex_format html; #以html风格将目录展示在浏览器中
    autoindex_exact_size off; #切换为 off 后，以可读的方式显示文件大小，单位为 KB、MB 或者 GB
    autoindex_localtime on; #以服务器的文件时间作为显示的时间
    charset utf-8,gbk; #解决中文乱码
}    
```

此功能依赖[ngx_http_autoindex_module](https://link.segmentfault.com/?url=http%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_autoindex_module.html)默认是加入构建的

开启简单的nginx页面后看到的只是简陋的页面

## 使用主题模块

使用[github ngx-fancyindex](https://github.com/aperezdc/ngx-fancyindex)模块来加载主题

在release下载最新的模块后，拷贝至nginx的编译目录

```bash
root@HostKvm-6f1901:/home/tmp/nginx-1.19.1# ll
total 812
drwxr-xr-x 10 1001 1001   4096 Jul 23 21:53 ./
drwxr-xr-x  5 root root   4096 Jul 18 13:50 ../
drwxr-xr-x  6 1001 1001   4096 Jul 18 13:39 auto/
-rw-r--r--  1 1001 1001 304578 Jul  7  2020 CHANGES
-rw-r--r--  1 1001 1001 464961 Jul  7  2020 CHANGES.ru
drwxr-xr-x  2 1001 1001   4096 Jul 18 13:39 conf/
-rwxr-xr-x  1 1001 1001   2502 Jul  7  2020 configure*
drwxr-xr-x  4 1001 1001   4096 Jul 18 13:39 contrib/
drwxr-xr-x  2 1001 1001   4096 Jul 18 13:39 html/
-rw-r--r--  1 1001 1001   1397 Jul  7  2020 LICENSE
-rw-r--r--  1 root root    376 Jul 23 01:12 Makefile
drwxr-xr-x  2 1001 1001   4096 Jul 18 13:39 man/
drwxr-xr-x  3 root root   4096 Jul 23 21:53 ngx-fancyindex-0.5.1/
drwxr-xr-x  4 root root   4096 Jul 23 01:13 objs/
-rw-r--r--  1 1001 1001     49 Jul  7  2020 README
drwxr-xr-x  9 1001 1001   4096 Jul 18 13:39 src/
```

`ngx-fancyindex-0.5.1`即主题模块

在nginx的编译参数中添加参数，修改对应版本号为你下载的版本号

```bash
./configure --add-module=./nginx-fancyindex-?.?.? 
```

### 编译

```bash
make
# make install会替换原来的二进制为nginx.old
```

- 进入nginx源码目录下的`objs`目录，执行`2>&1 ./nginx -V | tr ' ' '\n'|grep fan`
- 用`objs`目录下的nginx文件替换/usr/bin下面的nginx即可

### 或者使用动态模块

下载nginx[官方的动态模块版本](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

```bash
yum install nginx-module-fancyindex
# 在nginx.conf中加入
load_module "modules/ngx_http_fancyindex_module.so";
```

### 最后确认模块被加载

```bash
nginx -V

nginx version: nginx/1.19.1
built by gcc 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04) 
built with OpenSSL 1.1.1  11 Sep 2018
TLS SNI support enabled
configure arguments: --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_slice_module --with-http_gzip_static_module --add-module=/home/ngx_module/ngx-fancyindex-0.5.1
```



## 下载主题

官方模块中提供了建议的主题

### [Themes](https://github.com/aperezdc/ngx-fancyindex#id6)

The following themes demonstrate the level of customization which can be achieved using the module:

- [Theme](https://github.com/TheInsomniac/Nginx-Fancyindex-Theme) by [@TheInsomniac](https://github.com/TheInsomniac). Uses custom header and footer.
- [Theme](https://github.com/Naereen/Nginx-Fancyindex-Theme) by [@Naereen](https://github.com/Naereen/). Uses custom header and footer, the header includes search field to filter by filename using JavaScript.
- [Theme](https://github.com/fraoustin/Nginx-Fancyindex-Theme) by [@fraoustin](https://github.com/fraoustin). Responsive theme using Material Design elements.
- [Theme](https://github.com/alehaa/nginx-fancyindex-flat-theme) by [@alehaa](https://github.com/alehaa). Simple, flat theme based on Bootstrap 4 and FontAwesome.

使用@Naereen的主题

```bash
git clone https://github.com/Naereen/Nginx-Fancyindex-Theme.git

root@ubuntu:/home/code/node/Nginx-Fancyindex-Theme# tree
.
├── _config.yml
├── fancyindex.conf
├── HEADER.md
├── LICENSE
├── Nginx-Fancyindex-Theme-dark
│   ├── addNginxFancyIndexForm.js
│   ├── footer.html
│   ├── header.html
│   ├── jquery.min.js
│   ├── showdown.min.js
│   └── styles.css
├── Nginx-Fancyindex-Theme-light
│   ├── addNginxFancyIndexForm.js
│   ├── apple-icon.png
│   ├── favicon.ico
│   ├── footer.html
│   ├── header.html
│   ├── HEADER.md
│   ├── jquery.min.js
│   ├── README.md
│   ├── showdown.min.js
│   └── styles.css
├── README.md
└── screenshots
    ├── Nginx-Fancyindex-Theme__example1.png
    ├── Nginx-Fancyindex-Theme__example2.png
    ├── Nginx-Fancyindex-Theme__example3.png
    ├── Nginx-Fancyindex-Theme__example4.png
    ├── Nginx-Fancyindex-Theme__example5.png
    └── Nginx-Fancyindex-Theme__example6.png

3 directories, 27 files
```

我们只使用到了`Nginx-Fancyindex-Theme-light`下的主题

### 配置主题

可以拷贝`fancyindex.conf`文件到nginx的配置目录中引入，或者直接复制里面的内容

```nginx
location / {
  root /home;
  fancyindex on;
  fancyindex_localtime on;
  fancyindex_exact_size off;
  # Specify the path to the header.html and foother.html files, that are server-wise,
  # ie served from root of the website. Remove the leading '/' otherwise.
  fancyindex_header "/Nginx-Fancyindex-Theme-light/header.html";
  fancyindex_footer "/Nginx-Fancyindex-Theme-light/footer.html";
  # Ignored files will not show up in the directory listing, but will still be public.
  fancyindex_ignore "examplefile.html";
  # Making sure folder where these files are do not show up in the listing.
  fancyindex_ignore "Nginx-Fancyindex-Theme-light";
  # Maximum file name length in bytes, change as you like.
  # Warning: if you use an old version of ngx-fancyindex, comment the last line if you
  # encounter a bug. See https://github.com/Naereen/Nginx-Fancyindex-Theme/issues/10
  fancyindex_name_length 255;s
}
```

可以看到`fancyindex_header`使用的路径是绝对路径`/Nginx-Fancyindex-Theme-light`

当前根目录为`/home`，所以拷贝`Nginx-Fancyindex-Theme-light`目录到`/home`即可

## 主题自定义

### 修改header和footer

直接修改主题目录的`header.html`

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="x-ua-compatible" content="IE=edge">
        <title>Nginx Directory</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="/Nginx-Fancyindex-Theme-light/styles.css">
        <script type="text/javascript" src="/Nginx-Fancyindex-Theme-light/jquery.min.js"></script>
    </head>
<body>
<!--
header.html
© 2015-18, Lilian Besson (Naereen) and contributors,
open-sourced under the MIT License, https://lbesson.mit-license.org/
hosted on GitHub, https://GitHub.com/Naereen/Nginx-Fancyindex-Theme
-->
<div id="raw_include_HEADER_md"></div>
<h1>Directory:
```

添加内容即可

### 官方的自定义方式

修改`HEADER.md`文件，内部的内容会被渲染到页面顶部

## 效果预览

![](/images/nginx-om4-1.png)