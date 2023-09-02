---
title: linux上的golang
name: golang-linux-ins
date: 2020-03-07
id: 0
tags: [go]
categories: []
abstract: ""
---


在Linux上安装golang

<!--more-->

懒人必备

```bash
apt install golang-go
```

### 源码安装

```bash
wget https://dl.google.com/go/go1.14.linux-amd64.tar.gz
tar -zxvf go1.14.linux-amd64.tar.gz
mv go /usr/local
```

在系统的环境变量中配置go的编译链

```bash
nano ~/.bashrc
export GOROOT=/usr/local/go              # go源码路径
export GOPATH=$HOME/go     # 工作环境
export GOBIN=$GOPATH/bin           # 可执行文件存放
export PATH=$GOPATH:$GOBIN:$GOROOT/bin:$PATH       # 添加PATH路径

source ~/.bashrc
```

`GOROOT`是go自己的源码包所在的位置

`GOPATH`是用户自己的go程序所在的目录，一般建议`/home/用户名/go`，如果后面配置出现问题找不到包就修改这个路径

`GOBIN`是编译的go二进制程序的位置

测试是否配置成功

```bash
root@skypig:~/go/src/cloudp# go version
go version go1.12.5 linux/amd64
```

查看环境变量

```bash
go env
GOARCH="amd64"
GOBIN="/root/go/bin"
GOCACHE="/root/.cache/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/root/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/local/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build181243440=/tmp/go-build -gno-record-gcc-switches"
```

可以看到我的`GOPATH`在`/root/go`下，后面安装的包基本都在`/root/go/src`路径下

### 编译链工具

跨平台编译时go会用到`sys/unix`下的工具包，这是开发者包在源码中不包含需要自行下载

```bash
#在你的gopath下
cd src && mkdir -p golang.org/x
cd golang.org/x
git clone https://github.com/golang/sys.git
```

