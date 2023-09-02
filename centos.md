---
title: centOS从入门到放弃
name: centos
date: 2019-02-12
id: 0
tags: [linux]
categories: []
abstract: ""
---


再也不碰linux

# 安装CentOS虚拟机

## 对Linux系统进行配置

使用简单的命令语句
centos不允许使用root用户登录所以终端的指令有两种用户：普通用户和root
使用su进入root账户，输入root密码即可进行权限操作

<!--more-->

### 简单的权限操作

```bash
chown或者chmod usrname my  #usrname是你的用户名，my是你要修改的文件夹
chown或者chmod -R usrname my #将权限应用到子文件目录
```

### 简单的cd操作

```bash
#在想要指向的文件夹下使用终端即可进入文件夹
cd #即可回到根目录
cd.. #回到父目录
```

### 安装python3.7环境

------

在linux里python3.7以上的版本需要使用另外的依赖库否则会出现安装缺少module的问题
首先安装依赖库

```bash
yum install libffi-devel -y
```

然后安装组件

```bash
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

下载python3.7

```bash
wget https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tar.xz
```

建立一个空文件夹安装python

```bash
mkdir /usr/local/python3
```

安装

```bash
tar -xvJf  Python-3.7.1.tar.xz
cd Python-3.7.1
./configure --prefix=/usr/local/python3
make && make install
```

使用root创建软连接

```bash
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

命令行输入`python3`测试

### 添加桌面图标

使用下面命令创建一个desktop文件

```bash
gedit /home/用户名/Desktop/xxx.desktop #中文系统路径是桌面不是desktop
```

使用编辑器打开修改内容

```bash
[Desktop Entry]
Version=1.0
Encoding=UTF-8
Name=pycharm
Type=Application
Terminal=false
Name[en_US]=pycharm
Exec=pycharm脚本文件的路径.sh文件
Comment=pycharm
GenericName[en_US]=
Icon=图标的路径
# 注意所有的语句后面不能有空格
```

