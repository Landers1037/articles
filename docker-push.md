---
title: docker的push操作
name: docker-push
date: 2019-05-25
id: 0
tags: [docker]
categories: []
abstract: ""
---


docker的push有许多版本
当前只建立在push到远程的docker hub仓库

<!--more-->

首先docker desktop要登录到docker hub上

加入本地刚刚生成了一个新的镜像名为myubuntu，标签为latest
我们使用`docker tag`为其修改远程仓库的标签

`docker tag ubuntu:latest  landers1037/ubuntu:test1.0`

操作是 把本地的`ubuntu:latest`镜像tag修改为远程仓库的`ubuntu:test1.0`
其中`ubuntu`是远程仓库名，没有会自动创建，`test1.0`是tag

`docker images`

查看本地镜像是否应用更改

`docker push landers1037/ubuntu:test1.0`

开始推送到docker hub