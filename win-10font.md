---
title: win10替换字体
name: win-10font
date: 2019-02-04
id: 0
tags: [windows10]
categories: []
abstract: ""
---


# 前提：

## win10，记得自己的电脑账户密码<!--more-->

下载苹果方体
[百度网盘](https://pan.baidu.com/s/1Lwd3FKwxjgNjR2b60gHJFA)
第一步：备份原来的微软雅黑字体

`C:\Windows\Fonts`

```xml
msyh.ttc
msyhbd.tcc
msyhl.ttc
simsun.ttc
simsunb.ttf
```

第二步：解压出苹果字体，进入计算机的高级启动模式，替换fonts下的字体

因为我的电脑高级启动组件失效，所以使用如下方法：
搜索：系统配置

[![kJGnfJ.png](https://s2.ax1x.com/2019/02/04/kJGnfJ.png)](https://imgchr.com/i/kJGnfJ)

重启后进入安全模式，到fonts文件夹内替换字体即可，替换后再次进入系统配置，取消勾选安全引导选项