---
title: vsftpd错误总结2
name: linux-vsftpd-ans2
date: 2020-01-21
id: 0
tags: [linux,vsftpd]
categories: []
abstract: ""
---


使用vsftpd中遇到的错误总结篇2


<!--more-->


使用vsftpd中遇到的错误总结篇2

<!--more-->

### 无法连接至服务器

在启动vsftpd后，默认启用的是被动模式，需要开启对应的端口以供外部ftp客户端连接

例如在服务器的安全组开启`4000/8000`端口

修改`vsftpd.conf`

```python
pasv_enable=YES
pasv_min_port=4000
pasv_max_port=8000
```

在FTP客户端软件上设置优先使用被动模式

### 启用主动模式

当被动模式出问题可以尝试启动主动模式

```python
connect_from_port_20=YES
pasv_enable=NO
```

开启FTP转换规则

```bash
nano /etc/sysconfig/iptables-config
#添加
IPTABLES_MODULES="ip_conntrack_ftp"
IPTABLES_MODULES="ip_nat_ftp"
```

或使用该命令临时解决

```bash
modprobe ip_nat_ftp
```

重启防火墙和vsftpd

```bash
systemctl restart vsftpd
systemctl restart firewalld
```



### 访问权限出错

访问被服务器拒绝

vsftpd使用`userlist`和`ftpuser`文件控制访问权限，当userlist启用时(默认启用)，里面定义的所有用户名均不能访问ftp，ftpuser里的用户默认都不具有访问权限

在`/etc/vsftpd/`下修改这两个文件即可

### 服务器返回不可用外部地址

因为使用的是被动模式，没有配置被动模式下的ip地址

```bash
 nano /etc/vsftpd/vsftpd.conf
 
 pasv_address=你的服务器ip地址
```

在FTP客户端软件里修改传输规则，设置<u>被动模式</u>下的默认规则。如果是`使用外部服务器地址`代替则改为`退回到主动模式`