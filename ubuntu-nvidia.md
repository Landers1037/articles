---
title: linux的显卡驱动安装
name: ubuntu-nvidia
date: 2019-05-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


# 环境ubuntu18.04

## 安装nvidia显卡驱动<!--more-->

```bash
#首先使用ubuntu的驱动管理器查看当前系统推荐的显卡驱动
sudo ubuntu device #好像是这句话，参考百度一下
sudo ubuntu autoinstall #使用这句话自动安装系统推荐的驱动
有的时候上面的这句话会因为缺少依赖而无法安装，比如缺少nvidia-390-dkms依赖，我们先安装这个包，再运行一次自动安装
#或者使用以下命令直接安装390驱动
sudo apt-get install nvidia-drivers-390 #下载390的驱动
```

正常情况下，安装完成后，在软件和更新中查看正在使用的驱动会看见正在使用专有驱动，然后重启电脑，在系统设置信息里查看，正在使用的图形显示卡，看是不是nvidia显卡。如果不是就打开命令行终端，执行`sudo prime-select nvidia`，然后重启系统即可

## 出现的问题

```
#进入系统会出现循环登录的情况，一直让输入密码但是不能进入桌面
#出现的原因分析：显卡的OpenGL驱动部分和ubuntu的系统opengl文件冲突，导致启动失败。驱动版本不正确。
解决方法一
官网下载驱动run文件，在终端中执行
sudo ./***.run --no-opengl-files --no-x-check
解决方法二
在安装nvidia之前禁用自带的显卡驱动nouveau，参考百度，然而我并没有解决问题
解决方法三
在安装nvidia驱动时必须关闭图形化桌面，避免冲突，ubuntu18.04使用gdm而不是lightdm作为图形化服务
sudo systemctl stop gdm
关闭后使用终端安装驱动即可，如果之前安装了请先卸载先前的nvidia驱动，然而这个方法对我来说还是没用
解决方法四
ctrl+alt+f1或者f2进入tty界面，进入主目录下cd ~,然后列出全部文件ls -a
查看.xession.errors文件，它记录了xserver的启动错误，查看错误解决问题
我的错误是搜狗输入法的图形化界面无法启动，我在卸载了搜狗输入法后依旧没有用
```

## 尝试

因为安装显卡驱动的时候，图形界面xserver仍然在运行，所以显卡驱动的opengl文件覆盖了正在运行的xserver导致在重启的时候登录界面进入循环，尝试卸载掉全部的nvidia驱动

```bash
sudo apt remove -purge nvidia*
```

然后开机进入后按住ctrl+f1或者f2进入tty无图形化界面

再次执行安装NVIDIA驱动的指令

最后还是失败了。。。

## 于是放弃使用nvidia显卡作为图形卡

## 使用n卡驱动，禁用n卡达到省电目的

```bash
#使用自带的nouveau驱动是不能禁用nvidia显卡的必须使用nvidia的闭源驱动
sudo prime-select intel #切换为intel集成显卡
#按常理切换后nvidia显卡就会被禁用，但是由于nvidia的软件包使用的是nouveau来控制显卡的电源管理，这在ubuntu18.04上是不行的，所以显卡还是会耗电
#使用bbswitch代替nouveau方案
sudo apt-get install bbswitch-dkms
#修改nvidia的电源管理
cd /lib/systemd/system
sudo nano nvidia-prime-boot.service
#修改ExecStart后的/sys/kernel/debug/vgaswitcheroo/switch
/proc/acpi/bbswitch

cd /usr/lib
sudo nano prime-select
#你会发现他是python3写的
#注释掉_disable_nvidia方法下的三个调用的函数
#注释掉_enable_nvidia方法下的三个调用的函数

#在系统自启动配置下添加bbswitch.conf
cd /etc/modprobe.d
sudo nano bbswitch.conf
#添加如下内容
#echo 'options bbswitch load_state=0 unload_state=1'> /etc/modprobe.d/bbswitch.conf
#上面的那个#也要加

#然后添加开机启动程序
modprobe bbswitch
#更新内核
sudo update-initramfs -u

#查看是否禁用成功
lspci
#如果nvidia显卡后面的状态是 ff就是已关闭，是a1，a2就是没有禁用成功
```

