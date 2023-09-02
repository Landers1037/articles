---
title: grub引导界面的字体更换
name: grub-font
date: 2019-05-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


## 字体生成

使用如下的命令可以创建用于开机启动界面的字体文件

```bash
sudo grub-mkfont -o name.pf2 -s=25 /usr/share/fonts/truetype/ubuntu/Ubuntu-R.ttf
```

<!--more-->

-o  指定输出的字体文件名称
-s 指定输出的字体的大小
最后为字体文件的路径

在fonts目录下选择自己喜欢的字体，修改后，把生成的pf2二进制字体文件放入主题目录下，在theme.txt中修改即可，命名与theme主题目录下的pf2文件名称相同即可

## grub界面顺序文字调整

默认第一项为ubuntu，ubuntu高级选项，windows boot manager

需要调整顺序，只需要修改grub配置文件

```bash
cd /boot/grub
sudo nano grub.cfg
```

查找启动项名称的语句，在“ ”间修改自己想要的名称即可，不需要使用`update-grub`更新引导文件