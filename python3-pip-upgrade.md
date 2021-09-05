---
title: python的pip3升级问题
name: python3-pip-upgrade
date: 2019-06-17
id: 0
tags: [python]
categories: []
abstract: ""
---


在使用更新pip的时候遇到一些问题
比如`cannot import module name sysconfig`

填坑


<!--more-->


在使用更新pip的时候遇到一些问题
比如`cannot import module name sysconfig`

填坑

<!--more-->

### 问题一

使用命令`pip`或`pip3`

```sh
   from distutils import sysconfig
ImportError: cannot import name 'sysconfig'
```

出现在Ubuntu18.04上

解决方法

更新apt源列表

```bash
deb http://cn.archive.ubuntu.com/ubuntu bionic main multiverse restricted universe
deb http://cn.archive.ubuntu.com/ubuntu bionic-updates main multiverse restricted universe
deb http://cn.archive.ubuntu.com/ubuntu bionic-security main multiverse restricted universe
deb http://cn.archive.ubuntu.com/ubuntu bionic-proposed main multiverse restricted universe
```

重新安装

```sh
sudo apt install python3-pip
sudo apt-get install python3-distutils
```



### 问题二

`pip`报错cannot import name 'main',一般为pip更新后出错

`sudo nano /usr/bin/pip`

```python
from pip import main
if __name__ == '__main__':
    sys.exit(main())
```

修改为

```python
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```

再次执行`pip`

可能不能在root下修改并执行`pip`，我在root下执行`pip list`依旧提示错误，但是退出root后恢复正常

### 问题三

centos下没有python-dev工具包问题

```sh
yum install python36-devel
```

python2.*执行

```shell
yum install python27-devel
```

