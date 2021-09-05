---
title: vsftpd服务器添加用户
name: linux-vsftpd-user
date: 2019-05-21
id: 0
tags: [linux,vsftpd]
categories: []
abstract: ""
---


## Centos7

### 禁用匿名用户

```yaml
//安全起见禁用匿名用户上传权限和写权限
nano /etc/vsftpd/vsftpd.conf
anonymous_enable=YES //开启匿名
//禁用匿名上传，写权限，但是可以下载
#anon_upload_enable=YES
#anon_mkdir_write_enable=YES
#anon_other_write_enable=YES
```

### 
<!--more-->


## Centos7

### 禁用匿名用户

```yaml
//安全起见禁用匿名用户上传权限和写权限
nano /etc/vsftpd/vsftpd.conf
anonymous_enable=YES //开启匿名
//禁用匿名上传，写权限，但是可以下载
#anon_upload_enable=YES
#anon_mkdir_write_enable=YES
#anon_other_write_enable=YES
```

### <!--more-->添加用户

```bash
adduser name
passwd name
```

### 禁止用户ssh登录

```bash
cd /etc/ssh
nano sshd_config
//最后加上
DenyUsers name
//重启ssh服务
systemctl restart sshd
```

### 用户配置

```
在/etc/vsftpd/vsftpd.conf下修改
chroot_local_user=YES
#chroot_list_enable=YES
allow_writeable_chroot=YES
chroot_list_file=/etc/vsftpd/chroot_list
user_config_dir=/etc/vsftpd/userconfig
```

`user_config_dir`是用户配置信息目录
`cd /etc/vsftpd`  `mkdir userconfig`
进入userconfig下创建配置文件
`nano name`

```bash
local_root=/var/ftp/name
```

`systemctl restart vsftpd.service`

### 注意

`chroot_local_user=YES`一旦添加，用户根目录就会被锁定为/home/name不能改变
如果想要新建的用户根目录为pub公共目录下，必须注释上述项

更改根目录所属

`cd /var/ftp`    `chown -R name:name name`

### 测试

使用ssh连接用户，出现550禁止登录
使用ftp登录pub目录，可以上传下载
使用匿名用户登录pub，不能上传，可以下载