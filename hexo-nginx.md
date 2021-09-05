---
title: Nginx服务器部署hexo
name: hexo-nginx
date: 2019-03-19
id: 0
tags: [hexo]
categories: []
abstract: ""
---


# 为linux学习打基础

## 环境：centos7.3


<!--more-->


# 为linux学习打基础

## 环境：centos7.3

<!--more-->

## 	    阿里云ECS

## 	    git环境

## 准备

### 安装git在服务器上

```bash
yum install git-core # 适用centos redhat
apt-get install git-core # 使用ubuntu
```

权限不够使用sudo权限

### 安装nginx服务器

```bash
yum install -y nginx
```

启动服务

```bash
service nginx start
```

在浏览器输入服务器的ip 应该会出现nginx的测试界面

### 建立git仓库

```bash
adduser git # 添加用户
```

```bash
cd /home/git # 初始化hexo.git文件夹
git init --bare hexo.git
chown -R git:git hexo.git
```

创建一个密匙验证文件

```bash
vi /home/git/.ssh/authorized_keys
```

这时文件会自动打开，进入`C:\Users\Administrator\.ssh`文件夹下，用记事本打开`id_rsa.pub`公匙文件，把里面的内容拷贝到刚才新建的`authorized_keys`文件内保存

ssh密匙的生成参考 [廖雪峰](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)

### 配置Nginx服务

```bash
nginx -t #列出配置文件的路径
vi /etc/nginx/nginx.conf # 打开配置文件
```

```yaml
listen       80;
root   /home/git; # 网站的根目录
server_name  localhost;

location / {
  index  index.html index.htm;
}
```

```bash
service nginx restart # 重启服务
```

**遇到的问题**
访问网站时遇到403 forbidden
问题分析：使用sudo安装nginx时，用户是root，打开nginx的配置文件，首行发现user是nginx，修改为root即可

### 配置git文件

```bash
cd /home/git/hexo.git/hooks
# 新建增量更新文件配置
vi post-update
# 添加如下内容
#!/bin/sh
git --work-tree=/home/git --git-dir=/home/git/hexo.git checkout -f
# 赋予可执行权限
chmod +x post-update
```

## 测试

在本机hexo根目录下找到config.yml文件，修改deploy

```yaml
deploy:
  type: git
  repo: 
    github: git@github.com:Landers1037/Landers1037.github.io.git
    aliyun: git@服务器ip或域名:/home/git/hexo.git
  
  branch: master
```

```bash
hexo d 
```

再次在浏览器输入ip地址，部署成功