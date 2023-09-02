---
title: docker本地文件共享
name: docker-share
date: 2019-06-28
id: 0
tags: [docker]
categories: []
abstract: ""
---


环境docker for windows

解决docker镜像内访问本地目录的问题

<!--more-->

### 开启共享目录

在docker程序的`settings>Shared Driver`里勾选要共享的目录

### docker挂载目录

```sh
docker run -v e:/docker:/data ubuntu:latest ls /data
```

Ubuntu为你镜像的名称

使用命令后会自动列出挂载目录`e:/docker`的全部文件

如果需要挂载整个e盘

```sh
docker run -it -v e:/:/data ubuntu /bin/bash
```

进入镜像后，`/data`目录会自动挂载到根目录下

