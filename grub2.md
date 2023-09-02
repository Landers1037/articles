---
title: grub2引导界面的美化
name: grub2
date: 2019-04-19
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


## 用于windows与ubuntu的双系统

引导界面太丑美化一下

美化包下载：[链接](https://www.gnome-look.org)
<!--more-->

下载到系统后解压

创建主题文件夹

```bash
sudo mkdir /boot/grub/themes #新建一个主题文件夹
sudo cp -R 主题文件夹名称 /boot/grub/themes
```

修改grub引导配置文件

```bash
cd /etc/grub.d
sudo nano 00_header
```

在注释下面的第一行写入

```yaml
GRUB_THEME="/boot/grub/themes/主题包名/theme.txt"
GRUB_GFXMODE="1920x1080" 或者为"auto"
```

更新grub

```bash
sudo update-grub
```

## 注意

一个themes文件夹下不能有多个主题包，否则只会生效第一次配置的那个

## 重启