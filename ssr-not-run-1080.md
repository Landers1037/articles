---
title: ssr开启失败，1080端口被占用
name: ssr-not-run-1080
date: 2019-05-23
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


#### 检查端口占用

cmd运行 `netstat -ano`，找到占用程序关闭
<!--more-->


#### 检查端口占用

cmd运行 `netstat -ano`，找到占用程序关闭<!--more-->

#### 没发现端口占用

windows10的Hyber虚拟机服务默认监听1080端口，开启了WSL或者Hyber服务则1080端口被占用

关闭Hyber服务，或者修改ssr的本地监听端口为其他端口（注意不能和正在运行的其他服务端口冲突）