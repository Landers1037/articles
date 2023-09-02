---
title: 完全卸载双系统ubuntu的efi引导
name: delete-efi
date: 2019-04-19
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


# 使用power shell

```powershell
diskpart         #打开diskpart
list disk        #列出系统中拥有的磁盘，我笔记本上有两块磁盘，记得当时ubuntu启动项文件安装到了SSD所在的磁盘0中
select disk 0    #选择EFI引导分区所在的磁盘，请根据实际情况选择
list partition   #列出所选磁盘拥有的分区
select partition 1    #选择EFI引导分区，类型为系统的分区，就是EFI引导分区
assign letter=p:      #为所选分区分配盘符，请分配空闲盘符
exit                 # 退出
taskkill /im explorer.exe /f   #关闭explorer
explorer.exe     #再以管理员身份打开explorer
```


<!--more-->
此时，在文件管理器可以看见盘符为p的efi分区

使用管理员身份进入后，删除ubuntu文件夹

重启即可

[参考](https://www.jianshu.com/p/893c31c4fb19)

