---
title: windows10省电配置
name: windows-settings-for-power
date: 2019-05-14
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


# Windows1809

### 关闭onedrive

`win+r`输入`gpedit.msc`,找到计算机配置，windows组件，onedrive，`禁用onedrive进行文件存储`<!--more-->

### 关闭windows安全中心通知

`gpedit.msc`，计算机配置，windows组件，windows安全中心，`禁用所有通知`

### 关闭sysmain（旧版为superfetch服务）

计算机右键管理，服务，找到Sysmain，属性设置`禁用`

### 关闭后台

进入`设置`，`隐私`，`后台应用`，关闭需要的

### 关闭ipv6服务

进入`服务`，找到`ip helper`，禁用

### 关闭同步主机服务（作用是存储到onedrive）

win+r运行`regedit`，以下路径`计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`

找到`OneSyncSvc`，双击打开，找到start项，修改值为4（默认值为2）
找到`OneSyncSvc_***`，同样修改为4
找到`UserDataSvc`，修改start为4，默认为3
找到`UserDataSvc_***`,重复步骤
重启

### 关闭shell hardware detection

`服务`，`禁用`

### 关闭TCP/IP NetBIOS Helper

`服务，禁用`，作用网络文件共享

### 关闭windows defender

`gpedit`打开组策略，windows组件，`windows defender`禁用

### 关闭Windows Search服务

### 关闭Xbox服务

### 关闭家长控制服务

### 关闭Remote Desktop Services ***服务

### 关闭Connected User Experiences and Telemetry服务

### 关闭诊断服务Diagnostic Policy Service

### 关闭兼容助手Program Compatibility Assistant Service

### 组策略关闭rss服务

### 组策略禁止windows错误报告和日志记录

### 组策略禁用windows游戏录制

### 组策略禁用加入家庭组

### 关闭windows software 保护

`计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\sppsvc`