---
title: Centos查看端口占用
name: centos5
date: 2019-03-26
id: 0
tags: [linux]
categories: []
abstract: ""
---


```bash
lsof -i:8888
#查看端口的占用程序
```

```bash
netstat -tunlp |grep 8888
#查看端口的进程情况
```

<!--more-->