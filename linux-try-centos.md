---
title: linux虚拟机安装实践分享
name: linux-try-centos
date: 2018-03-05
id: 0
tags: [虚拟机linux]
categories: []
abstract: ""
---
一：安装VMware station虚拟机
=====================


<!--more-->


一：安装VMware station虚拟机
=====================

<!--more-->

二：下载linux虚拟机映像
==============

我下载的是ubuntu16.3的官方长期服务版

三：安装虚拟机
=======

打开虚拟机，选择添加虚拟机 
![](/images/linux-try-centos1.webp) 
![](/images/linux-try-centos2.webp)选择自定义
![](/images/linux-try-centos3.webp) 使用默认的就行，兼容性最好 
![](/images/linux-try-centos4.webp) 选择稍后安装，这样可以选择配置信息
![](/images/linux-try-centos5.webp) 
选择linux系统，版本如果是ubuntu就直接选择ubuntu的默认预设，如果是其他的选择对应的32或64位版本 安装的位置建议放在除c盘外的根目录下，否则出错
![](/images/linux-try-centos6.webp)
这里视自己的cpu性能而定，不要选择和cpu数目一样的否则会死机，建议都选择2 内存的选择虚拟机不需要很大的内存，按照建议来就行
![](/images/linux-try-centos7.webp)
![](/images/linux-try-centos8.webp)
这里的网络连接模式一定要选择，第一个桥接模式，如果你是用wifi连接的网络，不选择桥接可能导致无法联网！
![](/images/linux-try-centos9.webp)
![](/images/linux-try-centos10.webp)
类型按照默认的推荐的选择即可
![](/images/linux-try-centos11.webp)
如果之前装过ubuntu可以选择使用现有磁盘然后覆盖安装，第一次安装就选择创建新的磁盘，给你的磁盘取个名字
![](/images/linux-try-centos12.webp)
这里要注意的是，虚拟机空间不需要太大，因为你不需要在里面下载过多的东西，推荐的20gb足够了，如果选择20gb就将虚拟磁盘存为单个文件比较方便，这样也会提高访问的速度。 然后一直点下一步就好了 安装好的虚拟机就是这样
![](/images/linux-try-centos13.webp)
选择开启虚拟机试试吧

#### 当然初次开机的时候，屏幕会非常小，分辨率还无法调节

这个时候我们来安装虚拟机必备软件vmware tools来对虚拟机实现优化