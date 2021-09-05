---
title: linux修改ip地址，使用ssh连接
name: linux-ip-set
date: 2019-05-17
id: 0
tags: [linux]
categories: []
abstract: ""
---


## ssh连接sd卡启动的linux系统
<!--more-->


## ssh连接sd卡启动的linux系统<!--more-->

```bash
sudo vi /etc/network/interfaces
#修改为静态ip
iface eth0 static
address 192.168.0.1
netmask 255.255.255.0

#修改后
sudo /etc/init.d/networking
```

## 激活网卡

```bash
sudo ifconfig eth0 up
#sudo ifconfig eth0 192.168.0.1
```

## 配置本地ip

打开windows的网络适配器，进入有线网属性
选择ipv4地址，进入编辑使用静态ip地址，输入`192.168.0.2`

## 连接ssh

使用ssh协议的连接客户端，例如putty或者xshell
使用网线连接电脑和zynq板，配置ip地址`192.168.0.1`连接
成功会提示登录输入用户名和密码