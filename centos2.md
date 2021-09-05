---
title: linux文件操作指令
name: centos2
date: 2019-03-19
id: 0
tags: [linux]
categories: []
abstract: ""
---


# CentOS测试

## 文件夹

```bash
#生成文件夹test
mkdir test
#生成特定目录下文件夹
mkdir /home/test
#删除
rm test
rm -r test #删除test以及下面的子目录
```


<!--more-->


# CentOS测试

## 文件夹

```bash
#生成文件夹test
mkdir test
#生成特定目录下文件夹
mkdir /home/test
#删除
rm test
rm -r test #删除test以及下面的子目录
```

<!--more-->

## 文件

```bash
#生成key文件
vi key
#删除文件
rm key
```

```bash
rm -rf ***
#不提示用户直接删除，慎用
```

## 移动

```bash
mv tmp/1.test /home/test
#移动tmp下的1.test文件到home/test下
```

## 复制

```bash
cp 1.test /home/test
cp 1.test 2.test #把1中的内容复制到2中
cp -b 1.test 2.test #创建1.test的副本名为2.test
cp -R test /home/test2 #复制test以及子目录到test2中
```

## 遍历

```bash
ls #遍历列出当前文件夹下的所有文件
```

## vim编辑器

```bash
#使用vi编辑文件内容时
i #进入插入编辑模式，移动光标插入内容
a #在光标后面插入内容

#ESC键退出编辑模式
: #输入指令
q #退出
wq #保存退出
q! #强制退出
wq！ #强制保存退出
```

