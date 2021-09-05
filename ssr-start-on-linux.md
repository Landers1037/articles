---
title: ssr服务器部署
name: ssr-start-on-linux
date: 2019-05-24
id: 0
tags: [linux]
categories: []
abstract: ""
---


linux服务器上一键部署ss&ssr


<!--more-->


linux服务器上一键部署ss&ssr

<!--more-->

### 添加wget库

```shell
yum -y install wget
```

### 添加一键部署

```sh
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```

