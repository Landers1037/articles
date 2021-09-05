---
title: Centos的软链接指令
name: centos4
date: 2019-03-26
id: 0
tags: [linux]
categories: []
abstract: ""
---


软链接类似于一个快捷方式

将任意文件夹的命令链接到系统的根目录下

常用指令

```bash
ln -s 文件夹
```


<!--more-->


软链接类似于一个快捷方式

将任意文件夹的命令链接到系统的根目录下

常用指令

```bash
ln -s 文件夹
```

<!--more-->

centos上常见问题

安装python3版本后pip和python指令和系统自带指令冲突

```bash
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

安装nginx后 使用nginx指令提示不存在

```bash
ln -s /usr/local/bin/nginx /usr/bin/nginx
```

安装uwsgi后指令不存在

```bash
ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi
```

