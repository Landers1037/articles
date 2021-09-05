---
title: minikube学习笔记1
name: minikube-tutorial1
date: 2021-08-09 11:22:01
tags: [kubernetes]
categories: [kubernetes]
abstract: 
---
minikube学习笔记 - 简单集群的搭建

<!--more-->

## minikube仪表盘

```bash
minikube dashboard
```

![](/images/minikube-tutorial1-1.png)

![](/images/minikube-tutorial1-2.png)

## 部署一个容器

```bash
kubectl create deployment nginx --image=nginx
```

获取pod信息

```bash
kubectl get pod
```

![](/images/minikube-tutorial1-3.png)

### 暴露端口并启动容器

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```

![](/images/minikube-tutorial1-4.png)

```bash
minikube service nginx
```

![](/images/minikube-tutorial1-5.png)

![](/images/minikube-tutorial1-6.png)

## kubernetes面板

![](/images/minikube-tutorial1-7.png)

## 查看所有事件

```bash
kubectl get events
```

![](/images/minikube-tutorial1-8.png)

## 查看k8s配置

```bash
kubectl config view
```

## 获取service服务信息

```bash
➜ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        9d
nginx        NodePort    10.97.84.95   <none>        80:31356/TCP   16m
```

可以看到刚才创建的nginx服务内部端口为`80` 对应的映射端口为`31356`

### 同时查看服务和pod信息

```bash
➜ kubectl get pod,svc -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
pod/coredns-558bd4d5db-8w4qg           1/1     Running   2          9d
pod/etcd-minikube                      1/1     Running   2          9d
pod/kube-apiserver-minikube            1/1     Running   2          9d
pod/kube-controller-manager-minikube   1/1     Running   2          9d
pod/kube-proxy-gnch7                   1/1     Running   2          9d
pod/kube-scheduler-minikube            1/1     Running   2          9d
pod/storage-provisioner                1/1     Running   5          9d

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   9d
```



## 使用minikube插件

```bash
minikube addons list

|-----------------------------|----------|--------------|-----------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |      MAINTAINER       |
|-----------------------------|----------|--------------|-----------------------|
| ambassador                  | minikube | disabled     | unknown (third-party) |
| auto-pause                  | minikube | disabled     | google                |
| csi-hostpath-driver         | minikube | disabled     | kubernetes            |
| dashboard                   | minikube | enabled ✅   | kubernetes            |
| default-storageclass        | minikube | enabled ✅   | kubernetes            |
| efk                         | minikube | disabled     | unknown (third-party) |
| freshpod                    | minikube | disabled     | google                |
| gcp-auth                    | minikube | disabled     | google                |
| gvisor                      | minikube | disabled     | google                |
| helm-tiller                 | minikube | disabled     | unknown (third-party) |
| ingress                     | minikube | disabled     | unknown (third-party) |
| ingress-dns                 | minikube | disabled     | unknown (third-party) |
| istio                       | minikube | disabled     | unknown (third-party) |
| istio-provisioner           | minikube | disabled     | unknown (third-party) |
| kubevirt                    | minikube | disabled     | unknown (third-party) |
| logviewer                   | minikube | disabled     | google                |
| metallb                     | minikube | disabled     | unknown (third-party) |
| metrics-server              | minikube | disabled     | kubernetes            |
| nvidia-driver-installer     | minikube | disabled     | google                |
| nvidia-gpu-device-plugin    | minikube | disabled     | unknown (third-party) |
| olm                         | minikube | disabled     | unknown (third-party) |
| pod-security-policy         | minikube | disabled     | unknown (third-party) |
| registry                    | minikube | disabled     | google                |
| registry-aliases            | minikube | disabled     | unknown (third-party) |
| registry-creds              | minikube | disabled     | unknown (third-party) |
| storage-provisioner         | minikube | enabled ✅   | kubernetes            |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party) |
| volumesnapshots             | minikube | disabled     | kubernetes            |
|-----------------------------|----------|--------------|-----------------------|
```

列出所有支持的插件

### 激活插件

```bash
minikube addons enable metrics-server
```

### 失效插件

```bash
minikube addons disable metrics-server
```

## 清除资源

通过`delete`可以删除创建的服务

```bash
kubectl delete service nginx
kubectl delete deployment nginx
```

![](/images/minikube-tutorial1-9.png)

## 停止minikube

```bash
minikube stop
```

## 删除minikube虚拟机

```bash
minikube delete
```



## 参考资料

[hello minikube](https://kubernetes.io/docs/tutorials/hello-minikube)

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)