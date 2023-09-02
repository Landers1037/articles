---
title: 个人搭建aria2下载平台
name: aria-setup
date: 2018-03-04
id: 0
tags: [aria2,下载]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


![](/images/aria-setup.webp)

aria2属于一种新的下载方式，之前linux系统上使用较多，所以基本操作都是使用命令行，本软件自带配置文件和图形界面。目前国内最大的假药广告平台的百度网盘已经屏蔽了大部分的可提取外链下载的脚本，可以另外抓取百度网盘的链接但是下载多了会直接封号。某雷目前也差不多不充会员基本旧的资源无法获取速度，冲了会员也只是可以高速下载储存在迅雷云端的数据，旧到渣的资源也很难下载。 于是我们有了新一代的下载工具，aria2。搭配远程服务器，配合aria2可以利用本地加速和网络远程加速下载的多线程下载方式，支持b t 、http、ftp，magnet协议 本文不提供原理，不提供远程搭建服务器，由于aria2不是个软件而是通过建立本地环境直接用命令行下载链接的，这里提供aria2的gui界面封包软件。

 ok以下附下载地址和github源码

 windows 64位
 百度网盘 [https://pan.baidu.com/s/1sliAhjR](https://pan.baidu.com/s/1sliAhjR) 
github源码 [https://persepolisdm.github.io/](https://persepolisdm.github.io/) 
更新：号称最好看的图像界面ariaNg已经下载，搭建环境和服务器另找时间测试 

2018 3.3更新:百度api重新开放，使用油猴插件可以直接提取大文件直链下载，或者使用第三方yundownload内置的提取直链api进行下载，可能会封号一次下载不要贪多