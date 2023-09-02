---
title: Vsftpd的搭建
name: centos3
date: 2019-03-20
id: 0
tags: [linux]
categories: []
abstract: ""
---


# Centos7+vsftpd

## 安装

```bash
sudo yum install -y vsftpd #安装
```

<!--more-->

## 配置

```bash
cd /etc/vsftpd
systemctl enable vsftpd.service #开机启动
systemctl start vsftpd.service #启动服务
```

最重要的配置文件是vsftpd.conf

## 设置FTP

打开conf文件

```bash
 anonymous_enable=YES #允许匿名用户访问
# Uncomment this to allow local users to log in. 
# When SELinux is enforcing check for SE bool ftp_home_dir 
local_enable=YES #允许本地用户
# 
# Uncomment this to enable any form of FTP write command. 
write_enable=YES #全局设置允许写入创建文件（开启）
# 
# Default umask for local users is 077. You may wish to change this to 022, 
# if your users expect that (022 is used by most other ftpd's) 
local_umask=022 

anon_upload_enable=YES #允许匿名用户上传 
anon_mkdir_write_enable=YES #允许匿名用户创建文件夹
anon_other_write_enable=YES #允许匿名用户重命名文件夹 
# 
# Activate directory messages - messages given to remote users when they 
# go into a certain directory. 
dirmessage_enable=YES 
# 
# Activate logging of uploads/downloads. 
xferlog_enable=YES 
# 
connect_from_port_20=YES 

#chown_uploads=YES 
#chown_username=whoever #允许修改文件所有者
 
xferlog_std_format=YES 
# 
listen=YES #监听ipv4地址
# Make sure, that one of the listen options is commented !! 
listen_ipv6=NO #监听ipv6地址
 
pam_service_name=vsftpd 
userlist_enable=YES 
tcp_wrappers=YES 
```

修改配置文件后，修改公开目录的权限

```bash
chmod o+w /var/ftp/pub/ #更改/var/ftp/pub目录的权限
systemctl restart vsftpd.service #重启ftp服务
```

修改后如果还是不能上传文件，修改为允许全部权限

```bash
chmod -R 777 /var/ftp/pub/ 
```

遇到的问题

```bash
550 no permission
没有权限上传文件
开启匿名用户的上传权限，修改pub目录权限为777
```

```bash
553 无法创建文件
anon_mkdir_write_enable=YES #允许匿名用户创建文件夹
anon_other_write_enable=YES #允许匿名用户重命名文件夹
```

## 添加本地用户

```bash
useradd ftptest #创建ftptest用户
passwd ftptest #修改ftptest用户密码
#会让用户输入两次密码确认 
#创建的目录在/home/ftptest下
```

## 测试

使用xftp登录用户ftp出现如下问题

```bash
#无法显示远程文件夹
检查conf的设置
local_enable=YES #允许本地用户
将ftptest文件夹的权限设置为777
```

```bash
#用户验证失败
找到/etc/vsftpd下的user_list文件
里面包含禁止访问的用户名单项，如果里面有刚才创建的ftptest用户则删除
最新的vsftpd版本默认包含nobody用户，禁止一切外部连接，删除即可
```

```bash
#连接远程服务器失败
进入阿里云的控制台，找到安全，防火墙，添加规则
自定义 协议tcp 端口1
自定义 协议tcp 端口1024/65535 #开启后服务器安全性降低，测试端口后关闭
FTP   协议tcp 端口21
```

阿里云服务器默认关闭linux的网络防火墙，所以不需要配置linux防火墙的规则


每次上传文件的时候，如果新建文件夹，都需要配置权限