---
title: python多版本共存
name: python-coexist
date: 2019-06-27
id: 0
tags: [python]
categories: []
abstract: ""
---


**解决服务器上的多版本python共存问题**
因为大多数的linux服务器上都依赖了python2.7版本的python环境作为其系统环境，所以在安装python3的时候需要将python3版本特别注明与本地的python2.7区分开。
此次讨论的问题不在python2和python3的版本冲突，而是多个python3的版本共存

<!--more-->

### 问题

最新的ubuntu和centos在安装的时候都默认还包含了python3版本，我的服务器默认安装的是python3.6.8
在终端输入`python3`查看python的版本号
这就导致了我们写的flask和爬虫环境依赖python3.7等高版本时就无法正常启动

#### pip3安装的包问题

这个问题是最难发现的，在系统存在多个python3环境时，使用pip安装的包明明使用`pip3 list`命令可以看到但是使用`python3 import module`的时候却提示没有安装这个包
~~有些时候这个问题是由于你定义的py文件名称和包的文件名称一致~~

#### 分析问题

因为服务器上的用户权限几乎都是root，所以我们使用pip安装包的时候也是root安装
默认情况下，root的包安装在系统的python3环境目录下，但是当安装了新版的python3.7后pip3会安装到`/usr/local/python3/lib/python3.7/`下面
如果此时系统的python3编译器还使用的是原来的python3.6.8就不会找到安装的第三方包

### 解决

#### 安装python3.7

因为linux自带的包管理器下载的python3版本不是最新的包，所以直接去官网下载源码编译安装
安装步骤参考前面的文章
安装完成后一般的安装位置在

```sh
cd /usr/local/python3/bin
```

进入后会发现编译器会有多个，其中一个python3是系统默认的python3.6版本，一个python3.7是刚才安装的新版本python
终端输入`python3`再次查看python3版本号
不出意外应该还是系统默认的python3.6版本

#### 删除系统默认python链接

```sh
sudo rm -f /usr/bin/python3
```

创建新的链接

```sh
sudo ln -s /usr/bin/python3 /usr/local/python3/bin/python3.7
```

### 测试

查看python3版本

```sh
$ python3
```

此时应该为新安装的python3.7.x

查看pip3安装的包

```sh
$ python3
>>>import flask
```

没有报错则说明安装在python3.7下的flask包被成功导入