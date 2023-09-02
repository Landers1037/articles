---
title: Linux下的mysql安装配置
name: linux-mysql-i
date: 2019-05-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


# ubuntu安装mysql5.7

```bash
sudo apt install mysql-server
```

<!--more-->

### 配置

```bash
sudo mysql_secure_installation

Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: n
Please set the password for root here.

New password: 

Re-enter new password: 
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : n

 ... skipping.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 

```

### 检查mysql服务

```bash
systemctl status mysql.service
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: en
   Active: active (running) since Thu 2019-05-09 11:14:22 CST; 10min ago
 Main PID: 12781 (mysqld)
    Tasks: 29 (limit: 4915)
   CGroup: /system.slice/mysql.service
           └─12781 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pi

5月 09 11:14:22 landers systemd[1]: Starting MySQL Community Server...
5月 09 11:14:22 landers systemd[1]: Started MySQL Community Server.

```

## 修改配置文件

```
cd /etc/mysql/mysql.conf.d
sudo nano mysqld.cnf

添加一句话，跳过权限认证，否则每次使用mysql的时候都必须使用sudo权限
skip-grant-tables
```

## 安装gui管理界面workbench

官网下载deb包使用dpkg命令安装，因为缺少依赖所以必须使用-f命令

```bash
sudo dpkg -i xxx.deb
sudo apt -f install
再次执行dpkg安装即可
```

# Centos安装mysql

centos的yum库中更新mysql为Maridb了

```bash
yum -y install mariadb mariadb-server
systemctl start mariadb #开启服务
systemctl enable mariadb #开机启动
```

## 配置

```bash
mysql_secure_installation
```

步骤与mysql相同，如需远程登录，就设置yes

## 测试

```mysql
mysql -uroot -p password
进入后使用
show databases; 查看数据库
```

## 修改配置文件

不修改文本编码的话，中文就会报错

```yaml
#默认为my.cnf，但是修改 /etc/my.cnf.d/mysql-clients.cnf
【mysql】添加 default-character-set=utf8
【mysqld】添加 
character-set-server=utf8 
collation-server=utf8_unicode_ci 
重启服务
systemctl restart mariadb
```

## 和windows一样mysql的默认端口为3306