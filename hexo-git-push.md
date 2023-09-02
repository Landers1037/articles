---
title: Linux上为hexo添加git远程部署
name: hexo-git-push
date: 2019-05-21
id: 0
tags: [hexo,linux]
categories: []
abstract: ""
---


### 安装git

```bash
yum install -y git
```

### 创建git用户<!--more-->

```bash
adduser git
passwd git #记住自己的密码
```

### 创建公钥

```bash
git config --global user.name "name"
git config --global user.email "xxx@gmail.com"
//添加用户名 邮箱
ssh-keygen -t rsa -C "your mail"
//生成ssh公私钥
cat ~/.ssh/id_rsa.pub //查看公钥
```

### 上传公钥

```bash
mkdir /home/git/.ssh
cd /home/git/.ssh
nano authorized_keys
//把本地的公钥复制进来
```

### 初始化git仓库

```bash
cd /home/git
mkdir hexo.git
cd hexo.git
git init --bare //初始化
```

### 创建钩子

```bash
cd /home/git/hexo.git/hooks
nano post-receive
//加入内容
#!/bin/sh
git --work-tree=/home/git --git-dir=/home/git/hexo.git checkout -f
//其中目录为你的服务器部署hexo的web目录
chmod +x post-receive
```

### 注意

服务器上的公钥所有者应为git用户
权限 chmod 600 ~/.ssh/authorized_keys
		chmod 700 ~/.ssh