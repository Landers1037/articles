---
title: linux搭建的vsftpd服务器解决匿名上传和下载
name: linux-vsftpd-ans
date: 2019-05-19
id: 0
tags: [linux,vsftpd]
categories: []
abstract: ""
---


## 开启匿名上传下载

```yaml
anon_upload_enable=YES
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
anon_mkdir_write_enable=YES
anon_other_write_enable=YES #这句话不加匿名用户不能重命名和修改子目录
```

<!--more-->

## 解决匿名用户上传后不能直接下载

```yaml
chown_uploads=NO
#匿名用户上传文件的权限问题，修改后文件权限所属从root改为ftp
#根本解决问题，修改umask
默认umask为022，最终权限为600，修改umask为000，最终权限为666
```
