---
title: Docker新建保存容器
name: docker-new
date: 2019-05-23
id: 0
tags: [linux,docker]
categories: []
abstract: ""
---


#### Docker容器的使用
<!--more-->


#### Docker容器的使用<!--more-->

##### 新建镜像

`docker run ubuntu` docker默认从本地仓库寻找镜像，如果找不到就会自动从docker hub下载
`docker pull ubuntu` 这句话是直接从远程仓库下载Ubuntu公共镜像

##### 查看镜像

`docker images`

##### 启动

`docker run ubuntu`
`docker run -it ubuntu /bin/bash` 启动并进入shell界面

##### 修改镜像

进入shell后可以运行bash指令

`apt upgrade`
`apt install tree`
`exit` 退出镜像

##### 保存修改镜像

`docker commit '容器id' myubuntu` 将运行的容器保存名为myubuntu的新镜像
`docker ps -a` 可以查看所有的容器以及其中运行的镜像，找到运行ubuntu的容器id即可



