---
title: minikube学习
name: minikube-start
date: 2021-08-02 23:05:14
id: 0
tags: [kubernetes,docker]
categories: [kubernetes]
abstract: ""
---

**Kubernetes**实战 使用**minikube**创建集群

<!--more-->

## 安装docker

在mac版本的docker下自带了kubernetes相关的组件

用brew进行安装

```bash
brew cask install docker
```

通过docker for mac安装

[Docker Desktop for Mac by Docker | Docker Hub](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

## 安装kubectl

通过brew安装

```bash
brew install kubectl
```

![](/images/minikube-start1.png)

## 安装minikube

```bash
brew install minikube
```

![](/images/minikube-start2.png)

### 查看minikube版本

![](/images/minikube-start3.png)

## 创建一个集群

默认会使用`hyperkit`驱动，当我们安装`parallels`时可以使用`paralles`驱动

```bash
minikube start --driver=parallels
```

![](/images/minikube-start4.png)

启动成功后，使用`status`查看状态

```bash
minikube status
```

![](/images/minikube-start5.png)

### parallels

此时查看parallels desktop虚拟机可以看到多了一个`minikube`

![](/images/minikube-start6.png)

## 注意

在虚拟机中使用minikube时驱动应该设置为none

```bash
minikube start --driver=none
```

## 基于阿里云镜像的安装

可以尝试，最新的kube1.22版本镜像很多缺失没有配置成功

### 配置国内镜像

默认的`k8s.gcr.io`会重定向到Google的容器服务这在国内时不能正常访问的

```bash
minikube start --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --image-mirror-country="cn"
```

根据minikube的帮助信息

```bash
minikube start --help

--image-mirror-country='': Country code of the image mirror to be used. Leave empty to use the
global one. For Chinese mainland users, set it to cn.
      --image-repository='': Alternative image repository to pull docker images from. This can be
used when you have limited access to gcr.io. Set it to "auto" to let minikube decide one for you.
For Chinese mainland users, you may use local gcr.io mirrors such as
registry.cn-hangzhou.aliyuncs.com/google_containers
```

当在中国使用时使用`--image-mirror-country=cn`来使用国内的镜像

此时的镜像仓库可以设置为阿里云镜像`registry.cn-hangzhou.aliyuncs.com/google_containers`



但是这在我的电脑上报错

```bash
[init] Using Kubernetes version: v1.21.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'

stderr:
        [WARNING Firewalld]: firewalld is active, please ensure ports [8443 10250] are open or your cluster may not function correctly
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING Swap]: running with swap on is not supported. Please disable swap
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR ImagePull]: failed to pull image registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.0: output: Error response from daemon: manifest for registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.0 not found: manifest unknown: manifest unknown
, error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```



## 参考

[minikube release](https://github.com/kubernetes/minikube/releases)

