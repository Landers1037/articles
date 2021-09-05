---
title: linux虚拟机安装实践分享2
name: vmwaretools-use
date: 2018-03-06
id: 0
tags: [虚拟机linux]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->
<code>Sorry</code>该文章暂无概述💊
<!--more-->


安装VMware tools 等待虚拟机启动
![](/images/vmwaretools-use-1.webp)
![](/images/vmwaretools-use-2.webp) 选择虚拟机上方的菜单安装vmware tools 
![](/images/vmwaretools-use-3.webp) 如果是第一次安装会等待一小会，下载一个linux的zip文件 下载完成后文件会在文件管理器的下载目录里面
![](/images/vmwaretools-use-4.webp)
因为我已经删掉了所以在回收站里面，选择那个后缀名为tar.gz的文件右键提取，就提取在所在目录就行，记住提取出的文件的名字，或者重命名个简单的 
![](/images/vmwaretools-use-5.webp) 然后回到桌面，进入终端控制，鼠标右键打开终端 输入sudo su然后回车 按照提示输入你的账户root密码，就是你之前安装系统的时候的密码
![](/images/vmwaretools-use-6.webp) 
输入后会进入root模式 利用cd指令指向你放vmwaretools的目录，比如cd download 提醒，安装的是中文版linux的系统，所以目录应该是中文 改成cd 下载即可 想要输入中文点击右上角的键盘切换，切换成拼音![](/images/vmwaretools-use-7.webp) 
cd 下载目录后，继续cd 到你之前重命名的那个文件名字，后缀带上
![](/images/vmwaretools-use-8.webp)
然后回车，系统会自动开始安装，当看见successfully的时候表示已经安装完成 这个时候就可以改变分辨率，直接往虚拟机里拖文件了