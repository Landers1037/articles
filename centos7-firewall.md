---
title: centos7的防火墙设置
name: centos7-firewall
date: 2019-05-23
id: 0
tags: [linux]
categories: []
abstract: ""
---


<font size=3>centos7开始使用firewalld服务作为防火墙</font>

#### 启动与停止

```bash
systemctl start firewalld.service #开启服务
systemctl stop firewall.service #停止服务
systemctl disable firewall.service #禁用服务，重启生效
```

#### <!--more-->查看开启的端口

`firewall-cmd --list-ports`

#### 查询http和ssh是否开启

`firewall-cmd --zone=public --query-service=ssh`
`firewall-cmd --zone=public --query-service=http`

#### 规则策略

`permanent` 永久生效
`runtime` 运行时生效

#### 添加端口

`firewall-cmd --zone=public --add-service=https --permanent` 开放https规则端口

`firewall-cmd --zone=public --add-port=80/tcp --permanent` 开放80端口

`firewall-cmd --zone=public --add-port=8000-8080/tcp --permanent` 开放8000-8080端口

#### 删除端口

`firewall-cmd --zone=public --remove-service=https --permanent`

`firewall-cmd --zone=public --remove-port=80/tcp`

#### 查看端口是否开启

`firewall-cmd --query-port=80/tcp`

#### 使规则生效

`firewall-cmd --reload`
`systemctl restart firewalld.service`

