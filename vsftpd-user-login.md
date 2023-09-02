---
title: vsftpd无法登录问题解决
name: vsftpd-user-login
date: 2019-05-23
id: 0
tags: [vsftpd,linux]
categories: []
abstract: ""
---


在安装完vsftpd之后，添加用户ftp，然后无法使用filezila登录

#### 检查21端口是否开启

`firewall-cmd --list-ports`

#### 检查ftp配置文件<!--more-->

`/etc/vsftpd/vsftpd.conf`
`是否开启local_user`

#### 使用主动模式可以登录

使用filezila的主动模式可以登录但是被动模式不能登录
`firewall-cmd --zone=public --add-port=4000-5000/tcp --permanent`
修改配置文件`vsftpd.conf`添加
`pasv_enable=YES`
`pasv_max_port=5000`
`pasv_min_port=4000`

#### 允许用户

修改`/etc/vsftpd/user_list` 删除`nobody`

#### 重启ftp服务

`systemctl restart vsftpd.servie`