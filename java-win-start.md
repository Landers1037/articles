---
title: windows的java环境配置
name: java-win-start
date: 2018-03-10
id: 0
tags: [java,技术]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


### 一：右键计算机，属性，高级设置，环境变量

### 二：新建环境变量

1：JAVA\_HOME  变量值地址为jdk安装目录路径
2：CLASSPATH 变量值为 .;%JAVA\_HOME%\\lib;%JAVA\_HOME%\\lib\\dt.jar;%JAVA\_HOME%\\lib\\tools.jar 
3：Path(系统已经存在Path变量，请在Path变量最后进行添加，不要删除其他内容) ;%JAVA\_HOME%\\bin;%JAVA\_HOME%\\jre\\bin

 安装完成后在cmd里输入java，javac查看是否配置成功 ![](https://www.liaorenjie.top/wp-content/uploads/2018/03/QQ截图20180310214953-1024x535.png)