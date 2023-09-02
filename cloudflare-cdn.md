---
title: 利用cloudflare的cdn加速
name: cloudflare-cdn
date: 2019-05-23
id: 0
tags: [linux]
categories: []
abstract: ""
---


#### 申请域名

在域名商处申请域名，绑定自己的服务器ip地址
如果是对GitHub Pages进行加速，使用CNAME记录，添加自己的Github.io地址
进入dns管理，等待后续操作<!--more-->

#### 注册cloufalre

进入cloudflare官网，点击添加域名，添加刚才注册的域名，选择计划`free plan`

设置dns
	如果是GitHub Pages 添加A记录，指向Github page服务的ip地址（具体ip在github page帮助页面有）
	添加CNAME记录@，指向你的GitHub.io地址

​	如果是服务器，添加A记录@，指向你的服务器ip地址。添加www记录指向你的ip地址
​	如果你有自己的ssl证书，在cytro界面选择`full`，没有且想使用ssl加密选择`flexible`，不使用选`off`

#### 设置dns

在cloudflare的dns界面，会提供给两个cloudflare的dns地址，进入自己的域名商dns设置界面，删除原来的dns服务器地址，添加cloudflare的地址，完成

#### 遇到的问题

1. 为什么全部配置完成却不能之际通过域名访问服务器？ 一般等待120s

2. 为什么访问域名提示error521，host没有运行？ 检查服务器是否可以通过ip访问，可以则检查域名是否有ssl证书，没有就不要在cloudflare的ssl处开启`full`

3. 使用cloudflare后访问更慢了？ 加速只针对国外及部分运营商的部分地区，cloudflare主要还提供ddos攻击防护

   

   

   