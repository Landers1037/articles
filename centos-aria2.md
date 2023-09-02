---
title: Centos上搭建aria2+YAAW
name: centos-aria2
date: 2019-05-21
id: 0
tags: [linux]
categories: []
abstract: ""
---


### 安装源

```sh
#安装epel源
yum -y install epel-release
#安装aria2
yum -y install aria2
#查看aria2版本
aria2c -v
```

### <!--more-->配置

```bash
#新建配置文件夹
mkdir /home/aria2
touch /home/aria2/aria2.conf
touch /home/aria2/aria2.session
#新建下载文件目录
mkdir -p /home/aria2/download
```

```c
#编辑配置
daemon=true #后台进程保护
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/home/aria2/download
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
#disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
file-allocation=trunc
# 断点续传
continue=true
 
## 下载连接相关 ##
 
# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=5
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=32
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0
# 单个任务上传速度限制, 默认:0
#max-upload-limit=0
# 禁用IPv6, 默认:false
disable-ipv6=true
 
## 进度保存相关 ##
 
# 从会话文件中读取下载任务
input-file=/home/aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/home/aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
#save-session-interval=60
 
## RPC相关设置 ##
 
# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=60005
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
#rpc-secret=
# 设置的RPC访问用户名, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-user=
# 设置的RPC访问密码, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-passwd=
 
## BT/PT下载相关 ##
 
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
#follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=60002
# 单个种子最大连接数, 默认:55
#bt-max-peers=55
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=false
# 打开IPv6 DHT功能, PT需要禁用
#enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
#bt-enable-lpd=false
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=false
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true
```

### 运行

```bash
#运行aria2c
aria2c --conf-path=/home/aria2/aria2.conf
#如果您需要常驻运行，请修改为
nohup aria2c --conf-path=/home/aria2/aria2.conf &
```

### 配置YAAW

```bash
wget https://github.com/binux/yaaw/archive/master.zip
unzip master.zip
```

`cd /home/aria2/yaaw-master`后会看见如下文件

```xml
├── aria2.conf
├── aria2.session
├── download
├── master.zip
└── yaaw-master
    ├── css
    │   ├── bootstrap.min.css
    │   ├── bootstrap-responsive.min.css
    │   └── main.css
    ├── img
    │   ├── favicon.ico
    │   ├── glyphicons-halflings.png
    │   └── glyphicons-halflings-white.png
    ├── index.html
    ├── js
    │   ├── aria2.js
    │   ├── bootstrap.min.js
    │   ├── jquery-1.7.2.min.js
    │   ├── jquery.base64.min.js
    │   ├── jquery.jsonrpc.js
    │   ├── jquery.Storage.js
    │   ├── mustache.js
    │   ├── peerid.js
    │   └── yaaw.js
    ├── offline.appcache
    ├── README.md
    └── TODO.md
```

`index.html`就是我们需要的网页端

### 配置nginx服务器

```bash
cd /etc/nginx/conf.d
nano aria2.conf
```

```yaml
server {
    # 监听端口 
    listen 60003;
    # 项目的初始化页面
    location / {
       root  /home/aria2/yaaw-master/;
       index index.html;
    }
}
```

### 测试

打开服务器地址 你的ip`:60003`

会看到yaaw的服务界面（如果出现internel error就是aria2服务还没有开启）
点击`setting` 配置前面设置的rpc端口`60005`

### 设置登录密码

```bash
yum install httpd-tools
htpasswd -bc /home/aria2/yaaw-master/passwd user 123456
```

```c
nano /etc/nginx/conf.d/aria2.conf
server {
    # 监听端口 
    listen 60003;
    auth_basic  "input password";
    auth_basic_user_file /home/aria2/yaaw-master/passwd;
  
    location / {
       root  /home/aria2/yaaw-master/;
       index index.html;
    }
}
```

### 重启服务

```bash
service nginx start
#或者
systemctl restart nginx.service
```

