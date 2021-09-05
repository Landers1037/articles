---
title: win上使用ss&ssr的全解
name: winndowssr
date: 2018-02-25
id: 0
tags: [chrome,下载,技术]
categories: []
abstract: ""
---


### **因为政策本文不再更新**

### 第一步：下载win版的shadowsocksr软件
<!--more-->


### **因为政策本文不再更新**

### 第一步：下载win版的shadowsocksr软件<!--more-->

解压缩后放在目录下打开 ![C6W7Y6.jpg](https://s1.ax1x.com/2018/05/18/C6W7Y6.jpg) 

你会看见这些 ![C6WLlD.jpg](https://s1.ax1x.com/2018/05/18/C6WLlD.jpg) 

选择4.0版的exe软件打开会发现桌面的右下任务栏出现一个小飞机 ![C6Wx0A.jpg](https://s1.ax1x.com/2018/05/18/C6Wx0A.jpg) 

这是一个代表自由的小飞机（滑稽） 然后点开进入界面 ![C6fpkt.jpg](https://s1.ax1x.com/2018/05/18/C6fpkt.md.jpg) 

这就是软件的主界面，点左边的添加，添加线路，从这里输入服务器信息和加密协议，也可以将自己的ssr链接复制，直接右键点击任务栏的小飞机图标选择从剪切板导入ssr链接。 ![C6fCff.jpg](https://s1.ax1x.com/2018/05/18/C6fCff.jpg) ![C6fip8.jpg](https://s1.ax1x.com/2018/05/18/C6fip8.jpg)

![C6fk6g.jpg](https://s1.ax1x.com/2018/05/18/C6fk6g.jpg) 

关于ssr的服务商，自己寻找免费线路或者购买线路，不建议分享他人，不建议进行宣传，本来就是边缘的东西。

**要注意的是添加ssr的时候混淆协议和参数不要填错，否则无法连接服务器**

*   在配置好线路后点击确定，小飞机启动，右键小飞机图标出现如下界面
*   点击系统代理模式
*   软件内置了pac的规则，一共有三种规则：直连，pac和全局代理。直连即不经过代理直接连接网络，pac规则内部有事先写好的绕过中国大陆主要网站（只有pac里规定的使用代理的ip才会开启代理），全局代理所有流量经过服务器，打开国内网络可能很慢并且费流量。

![C6fe7n.jpg](https://s1.ax1x.com/2018/05/18/C6fe7n.jpg) 压缩包里自带了pac文件不需要配置，选择pac模式可以保证网站的正常访问，其他地方不建议更改。 现在可以进行代理访问网络 **需要注意的一点，非常重要的一点！电脑上不要有杀毒软件比如360，电脑管家，他们的后台监控会主动上传ip地址，可能造成一段时间后ssr的服务器ip被墙。如果有不要打开，尊重线路的创建者最好不要打开这些软件后台运行。**

### 第二步：配置浏览器

部分浏览器貌似不需要配置就会使用系统代理，以chrome浏览器内核开发的需要使用额外插件。 

以chrome为例： **打开chrome进入插件页面（扩展程序）**![C6fnkq.jpg](https://s1.ax1x.com/2018/05/18/C6fnkq.jpg) 

**勾选开发者模式，将所提供的代理插件拖到这个页面安装后启用** ![C6fQpT.jpg](https://s1.ax1x.com/2018/05/18/C6fQpT.jpg) 

**左边的导入\\导出，点击进入，选择从备份中恢复** ![C6f1cF.jpg](https://s1.ax1x.com/2018/05/18/C6f1cF.md.jpg) **导入所提供的配置文件，然后退出，这是chrome右上方的插件会出现一个圈** ![C6fGnJ.jpg](https://s1.ax1x.com/2018/05/18/C6fGnJ.jpg) 

**单击出现选项，选择系统代理** ![C6fJB9.jpg](https://s1.ax1x.com/2018/05/18/C6fJB9.jpg) 

：）浏览器的代理上网就配置好了 

**说明：在ssr的压缩包内提供了chrome的代理插件，安装后新建代理模式选择系统代理即可。**

#### 最后提供下载链接（部分需科学上网）

##### win版ssr下载地址：github地址[https://github.com/shadowsocksrr/shadowsocksr-csharp/releases/tag/4.8.0](https://github.com/shadowsocksrr/shadowsocksr-csharp/releases/tag/4.8.0)

win版V2ray下载地址：[https://pan.baidu.com/s/1dLjwt1HCpoO3-RILvfKaPQ](https://pan.baidu.com/s/1dLjwt1HCpoO3-RILvfKaPQ)

##### 云端硬盘 [https://drive.google.com/open?id=10reY1wA0CcJc4B2CQEqCzVc0CU_rBzlh](https://drive.google.com/open?id=10reY1wA0CcJc4B2CQEqCzVc0CU_rBzlh)

##### chrome插件：[https://drive.google.com/open?id=106Pw2j4AKPEM5HYdBvI1DRw_nqn5yhao](https://drive.google.com/open?id=106Pw2j4AKPEM5HYdBvI1DRw_nqn5yhao)

##### chrome插件配置文件：[https://drive.google.com/open?id=1HzD068xhLxLBrv4QCxv9WsBcJZD9esfY](https://drive.google.com/open?id=1HzD068xhLxLBrv4QCxv9WsBcJZD9esfY)

插件名称：proxyswitchyomega可自行下载

文明上网，从你我做起
----------

最后更新于2018.6.18