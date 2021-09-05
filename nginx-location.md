---
title: nginx的location使用
name: nginx-location
date: 2019-06-14
id: 0
tags: [linux,nginx]
categories: []
abstract: ""
---


Nginx服务器的location配置详解

```nginx
location / {
        }
	error_page 404 /404/index.html;
            location = /404/index.html {
        }
```


<!--more-->


Nginx服务器的location配置详解

```nginx
location / {
        }
	error_page 404 /404/index.html;
            location = /404/index.html {
        }
```

<!--more-->

### Location后面的通配符

语法规则： location [=|~|~*|^~] /uri/ { … }

`=` 开头表示精确匹配

`^~` 开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为`/static/20%/aa`，可以被规则`^~ /static/ /aa`匹配到

`~` 开头表示区分大小写的正则匹配

`~*`  开头表示不区分大小写的正则匹配

`!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则

`/` 通用匹配，任何请求都会匹配到

首先匹配 `=`，其次匹配`^~`, 其次是按文件中顺序的正则匹配，最后是交给 `/` 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求

```nginx
location / {
    规则1
        }
#匹配http://localhost/+任何网址
location =/ {
    规则2
        }
#匹配http://localhost/
location = /home {  
   规则3
} 
#匹配http://localhost/home
location ^~ /static/ {  
   规则4
}  
#匹配http://localhost/static/*.
location ~ \.(gif|jpg|png|js|css)$ {  
   规则5
} 
#匹配http://localhost/*.png|gif|jpg|js|css
location ~* \.png$ {  
   规则6 
}
#匹配http://localhost/*.png
location !~ \.xhtml$ {  
   规则7 
}  
location !~* \.xhtml$ {  
   规则8 
}
#访问http://localhost/a.xhtml 不会匹配规则7和规则8，http://localhost/a.XHTML不会匹配规则8。属于排除规则
```

### root与alias使用

使用root寻找路径，最终路径为root+location

使用alias寻找路径，最终路径为alias路径（alias用来替换location）

```nginx
location ^~ /home/ {
     root /www/root/html/;
}
```

请求为`/home/a.html`，路径为：`/www/root/html/home/a.html`

```nginx
location ^~ /home/ {
 alias /www/root/html/other/;
}
```

请求为`/home/a.html`，路径为：`/www/root/html/other/a.html`

**注意alias路径最后一定要加`/`**

### 禁止访问某路径

使用`deny`参数

```nginx
location ~* /.* {
deny all;
}
```

禁止访问根目录

带`/`表示禁止访问目录，不带`/`表示禁止访问文件，按需自定义后面的规则