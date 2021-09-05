---
title: docker入门教程1
name: docker-start1
date: 2021-07-20 21:31:42
tags: [docker]
categories: [docker]
abstract: 
---
docker入门 - 在Goland中使用docker

![](/images/docker-start1-4.png)

<!--more-->

# 在Goland中配置使用docker

![](/images/docker-start1-1.png)

## 可以使用本机安装的docker或者来自远程tcp的docker

### **配置开启tcp连接**

```bash
vi /lib/systemd/system/docker.service
```

```bash
注释掉旧的
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

添加新的2375端口
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
# 对于跨域访问 添加响应头
--api-cors-header='*'
```

![](/images/docker-start1-2.png)

### 重启docker

```bash
# 重新加载守护进程
systemctl daemon-reload

# 重启docker
systemctl restart docker
```



### 开启端口

```bash
ss -tnl | grep 2375

firewall-cmd --zone=public --add-port=2375/tcp --permanent
firewall-cmd --reload
```

![](/images/docker-start1-3.png)

## 最终效果

在Goland中通过docker插件连接

![](/images/docker-start1-4.png)