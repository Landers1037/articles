---
title: meta标签和link总结
name: http-meta-link
date: 2020-08-16
id: 0
tags: [html]
categories: []
abstract: ""
---


**HTML**的`meta`元标记和`link`链接的总结

<!--more-->

## meta

元标记是html页面重要的信息，写在head头部中，可以帮助浏览器更好处理html页面

### 信息标记

使用的编码格式

```html
<meta charset='utf-8'>
```

页面描述信息

```html
<meta name="description" content="不超过150个字符"/>
```

页面的关键词

```html
<meta name="keywords" content="一般用,隔开"/>
```

作者信息

```html
<meta name="author" content="name, author@gmail.com"/>
```

rss链接

```html
<link rel="alternate" type="application/rss+xml" title="RSS" href="/rss.xml"/>
```

标题栏图标

```html
<link rel="shortcut icon" type="image/ico" href="/favicon.ico"/>
```



### 兼容性

使用Edge浏览器而不是ie，或者使用chrome

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
```

针对不支持viewport的设备

```html
<meta name="HandheldFriendly" content="true">
```

针对ie10以下浏览器

```html
<meta name="MobileOptimized" content="320">
```



### 页面控制

爬虫禁止，指定可爬取的页面

```html
<meta name="robots" content="index,follow"/>
#不允许爬取时
<meta name="robots" content="noindex,nofollow"/>
```

缓存控制

```html
<meta http-equiv="cache-control" content="no-cache" />
<meta http-equiv="expires" content="0" />
#使用no-store会禁用更加彻底
```

自动刷新页面

```html
<meta http-equiv="refresh" content="20" />
#20s刷新
```

禁用google翻译

```html
<meta name="google" content="notranslate" />
```

禁止将数字识别为电话号码

```html
<meta name="format-detection" content="telephone=no">
```



### 移动端优化

视窗大小设置

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" />
```

定义`maximum-scale`来设置网页可以被放大最大的倍数，`user-saclable=no`则为禁止用户手动放大页面

ios设备以web application模式运行(在chrome中也有效果)

```html
<meta name="apple-mobile-web-app-capable" content="yes">
```

ios浏览器的默认页面标题

```html
<meta name="apple-mobile-web-app-title" content="标题">
```

设置safari工具栏的颜色

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black"/>
```

默认桌面图标

```html
<link rel="apple-touch-icon" href="touch-icon-iphone.png" />
<link rel="apple-touch-icon" sizes="76x76" href="touch-icon-ipad.png" />
<link rel="apple-touch-icon" sizes="120x120" href="touch-icon-iphone-retina.png" />
<link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad-retina.png" />
```

默认大小为52x52

启动页面加载图片

```html
<link rel="apple-touch-startup-image" href="start.png"/>
<link rel="apple-touch-startup-image" sizes="768x1004" href="/splash-screen-768x1004.png"/>
#根据size匹配不同设备
```

QQ的应用模式

```html
<meta name="x5-page-mode" content="app">
```

UC的应用模式

```html
<meta name="browsermode" content="application">
```

QQ强制全屏

```html
<meta name="x5-fullscreen" content="true">
```

UC强制全屏

```html
<meta name="full-screen" content="yes">
```

QQ强制竖屏

```html
<meta name="x5-orientation" content="portrait">
```

UC强制竖屏

```html
<meta name="screen-orientation" content="portrait">
```

强制横屏

```html
<meta name="screen-orientation" content="landscape">
```

应用模式

```html
<meta name="browsermode" content="application">
```



## link

link标签用于加载资源

使用`rel`指定资源的类型，如`rel=stylesheet`

对于需要用到的重要资源，为了提升网页的响应速度可以使用预加载优先下载

比如字体文件，为了保证字体不会出现加载慢闪烁问题，可以使用预加载`preload`

```html
<link rel="preload" href="/static/res/font/Lato-Regular.woff2" as="font" crossorigin="" type="font/woff2">
```

`rel`指定为`preload`，使用`as`指定为`font`文件

字体文件的预加载需要添加`crossorigin`属性，为防止字体在`css`文件内引用时重复加载字体资源造成流量浪费

预加载css

```html
<link rel="preload" href="/static/css/style.css" as="style" type="text/css">
```

预加载js

```html
<link rel="preload" href="/static/js/main.js" as="script" type="text/javascript">
```

js资源使用preload预加载后不用通过`‘<script>’`标签引入

预加载会提高资源加载的优先级，并且不会阻塞页面的load过程