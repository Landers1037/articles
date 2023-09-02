---
title: Centos安装python3
name: centos-python
date: 2019-03-26
id: 0
tags: [linux]
categories: []
abstract: ""
---


**centos本身自带python2.7用于系统的环境设置**

安装另外的python3版本

```bash
#安装依赖
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

<!--more-->

```
wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tar.xz
```

```bash
#创建本地文件夹
mkdir /usr/local/python3
```

```bash
mv Python-3.7.2.tar.xz /usr/local/python3
tar -xvJf  Python-3.7.2.tar.xz
cd Python-3.7.2
./configure --prefix=/usr/local/python3
make  
make install
```

```bash
#创建软链接
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

```bash
#配置yum文件 让python版本共存
vi /usr/bin/yum
把#! /usr/bin/python修改为#! /usr/bin/python2
```

```bash
vi /usr/libexec/urlgrabber-ext-down
把#! /usr/bin/python修改为#! /usr/bin/python2
```

遇到no module name"_ctypes"

```sh
yum install libffi-devel -y
```

然后继续执行`make install`