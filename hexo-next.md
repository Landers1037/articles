---
title: hexo的NEXT主题配置
name: hexo-next
date: 2019-05-21
id: 0
tags: [hexo]
categories: []
abstract: ""
---


## hexo安装next主题

 [官网](http://theme-next.iissnan.com/)


<!--more-->


## hexo安装next主题

 [官网](http://theme-next.iissnan.com/)

<!--more-->

## 主题配置

好朋友：百度一下

## 部分详细配置

### 语言

默认非中文，在博客目录的`_config`中`language`后加上`zh-CN`

### 侧边栏项目

```bash
hexo new page xxx
```

使用该命令创建一个新的分类目录，目录位置`/source/xxx`
修改next主题目录下的`_config.yml`，添加刚生成的xxx目录，||后面的名称是图标文件的名称，详见next使用的图标源

```yaml
menu:
  home: / || home
  tags: /tags/ || tags
  #categories: /categories/ || archive
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

### 社交图标

```yaml
social:
  GitHub: https://github.com/xxx || github
  Gitee: https://gitee.com/xxx || envira
```

### 阅读人数统计

```yaml
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
  //开启busuanzi不需要注册其他的统计网站账户，比较方便
```

### 主题背景设置

```yaml
//主题目录source/css/custom下
// Custom styles.
body{
	background:url(/images/nap.jpg);
	background-attachment:fixed;
	background-size:cover;}
```

### 添加头像

```yaml
avatar:
  # In theme directory (source/images): /images/avatar.gif
  # In site directory (source/uploads): /uploads/avatar.gif
  # You can also use other linking images.
  url: /images/Landers.jpg
```

