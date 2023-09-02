---
title: gitbook的使用
name: gitbook-create
date: 2019-06-27
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


如何使用gitbook

环境：Windows10 node.js

<!--more-->

### 使用

gitbook是一个命令行形式的markdown写书工具

可以结合使用gitbook editor作为图形化编辑器写出格式优美的书籍

### 安装

依赖node.js环境，Windows到官网下载并安装

在git-bash中输入

```sh
$ npm install gitbook-cli -g
```

查看版本

```sh
gitbook -V
```

### 初始化

使用gitbook editor写好一篇文章后，进入文章所在位置打开终端

默认路径为`C:\Users\用户名\GitBook\Library\Import\`

```sh
$ gitbook init
$ gitbook serve
```

然后就可以在本地端口4000查看书籍的html文件