---
title: pymysql安装使用
name: pymysql-learn
date: 2019-02-01
id: 0
tags: [mysql,python,pymysql]
categories: []
abstract: ""
---


# Mysql的安装

使用官方提供的mysql installer进行安装，如果只是想使用数据库那么就只安装mysql server



<!--more-->


# Mysql的安装

使用官方提供的mysql installer进行安装，如果只是想使用数据库那么就只安装mysql server


<!--more-->
常见的安装mysql错误，错误码1405
在使用cmd的时候使用net start mysql会提示mysql服务无法启动，分析原因：

1. 之前安装过mysql数据库，之前的注册表没有完全删除，导致多个mysql服务同时在运行造成冲突
2. 正在运行的软件占用了3306端口，比如ssr使用的本地端口
3. my.ini配置出错，建议再次检查一遍
4. 另外一件事，几乎所有的服务器例如apache，wampserver在搭建本地web的时候默认使用的都是localhost的80端口，这个端口很容易被QQ强占，所以建议不要调试的时候打开QQ

解决方法：
使用everything搜索mysql删除相关的所有文件和注册表，重新安装

# PyMysql的使用

## 安装pymysql

使用pycharm的插件管理或者直接使用pip进行安装，我使用的python版本3.7.1

## 安装mysql

在官网下载mysql安装包进行安装，记住安装的文件夹，我的安装目录是：C:\Program Files\MySQL\MySQL Server 5.7，记住mysql和mysql installer不一样后者是数据库的安装程序

安装完成后：
进入cmd，进入到安装的目录 cd C:\Program Files\MySQL\MySQL Server 5.7\bin在这里创建一个my.ini文件使用notepad++进行编辑，将下面的代码拷贝进去

```mysql
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:\Program Files\MySQL\MySQL Server 5.7
# 设置mysql数据库的数据的存放目录
datadir=C:\Program Files\MySQL\MySQL Server 5.7/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

数据库的存放目录，在安装路径下并不存在，我们需要新建一个data文件夹

进入cmd安装数据库
执行初始化mysqld --initialize-insecure 
执行mysqld install(如果提示数据库已经存在则使用sc delete mysql或者mysql-remove)
启动服务net start mysql （或者进入电脑的管理，服务下自行启动mysql服务）
当遇到错误193的时候，进入bin文件夹下删除mysqld的0kb文件即可
添加密码mysqladmin -u root password******
进入数据库mysql -u root -p输入刚才的密码
（如果提示错误，就是在安装mysql的时候已经配置过密码，输入那时的密码即可）
打开成功就会出现以下提示

![1.png](https://s2.ax1x.com/2019/02/01/k3TlWQ.png)